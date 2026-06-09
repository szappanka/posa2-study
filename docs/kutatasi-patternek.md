# POSA2 patternek a kutatásomban

## A kontextus egy mondatban

Különböző szenzorforrásokból (HoloLens 2, LiDAR, Quest 3, Unitree robot)
minimális szemantikus modellt építek egy fizikai térről, amelyet Unity + MRTK +
OpenXR keretrendszerben valós idejű feladatvégzésre használok fel.

---

## Wrapper Facade

Ez az alap. A HoloLens 2, Quest 3 és az Unitree robot mind teljesen eltérő
hardware API-kat használ. A HoloLens a Win32 és WinRT felületen keresztül adja
a szenzorokat, a Quest 3 OpenXR-en, az Unitree saját SDK-n keresztül.

Ha ezeket az API-kat közvetlenül hívom a jelenetmodellező kódból, az egész
rendszer platformfüggő lesz. Ha holnap megváltozik az SDK, az egész kódot
újra kell írni.

A Wrapper Facade megoldása: minden szenzorforrás köré írok egy OOP osztályt,
amely egységes interfészen adja az adatot. A jelenetmodellező kód nem tudja és
nem is kell tudnia, hogy éppen HoloLens-től vagy LiDAR-tól kapta a 2D scan-t.
Pontosan ezt csinálja a regisztrációs pipeline is: az ICP algoritmus nem foglalkozik
azzal, hogy a két felhő melyik szenzorból jött.

---

## Component Configurator

A rendszerem két különböző platformon fut: VR headseten és Unitree roboton.
A jelenetmodellező komponens ugyanaz, de a szenzorfeldolgozó komponens más
a két platformon, és futás közben cserélhető kell legyen, például ha a HoloLens
mellé bekapcsoljuk a külső LiDAR modult.

A Component Configurator megoldása: minden platform-specifikus feldolgozó
komponens egy egységes interfészt (init, fini, suspend, resume) valósít meg,
és a rendszer futás közben tölti be a megfelelőt, az alkalmazás többi részének
újraindítása nélkül. Ez kritikus valós környezetben: nem állíthatom le a robotot
csak azért, hogy átkapcsoljak egy másik szenzormodulra.

---

## Interceptor

Az MRTK pontosan interceptor-szerűen működik. Az OpenXR kérések feldolgozása
előtt és után automatikusan lefutnak a gesture recognition, eye tracking és
spatial mapping hook-ok, anélkül hogy az alkalmazás kódjába kellene belenyúlni.
A keretrendszer nem tudja és nem is kell tudnia, hogy milyen hook-ok vannak
regisztrálva, mégis automatikusan meghívja őket minden releváns eseménynél.

A konfidencia score számítása szintén interceptorként értelmezhető: minden
regisztráció befejezésekor automatikusan lefut az öt belső metrika kiértékelése,
nem kell explicit meghívni. Ha holnap új metrikát akarok hozzáadni, csak
regisztrálok egy új Concrete Interceptort, a pipeline többi részét nem érintem.

---

## Extension Interface

Ez a legszorosabb párhuzam a Unity komponens-rendszerrel.

Egy fizikai tér objektuma egyszerre több szerepet tölt be a rendszeremben:
a renderelő modulnak a vizuális reprezentációja kell, a regisztrációs pipeline-nak
a geometriája, a szemantikus modellnek a kategóriája és a feladatkezelőnek az
állapota.

Ha ezeket egyetlen nagy interfészbe rakom, minden változtatás minden klienst
érint. A Unitree robot más adatokat igényel ugyanarról az objektumról, mint a
HoloLens nézet.

Az Extension Interface megoldása: minden szerephez külön interfész, és a
getExtension() mechanizmuson keresztül mindenki csak azt a részt látja, amire
szüksége van. Ez pontosan a Unity MonoBehaviour rendszerének a logikája:
egy GameObject-hez tetszőleges komponenseket lehet csatolni, és minden rendszer
(fizika, render, AI) csak a saját komponensét kéri le. A COM QueryInterface()
mechanizmusa ugyanez.

---

## Reactor

A regisztrációs pipeline több eseményforrást figyel párhuzamosan:
bejövő scan az egyik szenzorból, ICP konvergencia jelzés, symmetry-flip
ellenőrzés eredménye, UI esemény a HoloLens-ről.

Ha ezeket sorban, blokkolva várnám, a rendszer lelassulna. A Reactor megoldása:
egy eseményciklus figyeli az összes forrást egyszerre, és csak akkor dolgoz fel,
amikor valamelyik ténylegesen kész. Ez az amit a Unity event rendszere is
implementál, és amit az MRTK input kezelése is követ.

---

## Half-Sync/Half-Async

A regisztrációs pipeline hosszan futó műveletet tartalmaz: az ICP iteratív
optimalizálás és a symmetry-flip check akár több száz milliszekundumig futhat.
Ezt nem futtathatom a fő szálon, mert a HoloLens UI lefagyna, a robot
vezérlése leállna.

A Half-Sync/Half-Async megoldása: az aszinkron réteg fogadja a szenzor adatokat
és berakja egy sorba. A szinkron réteg munkaszálai futtatják az ICP-t és
a konfidenciapontszám számítást. A két réteg közt a Monitor Object-tel
implementált sor kommunikál. Ez az architektúra pontosan az, amit a cikkben
leírt pipeline is követ: párhuzamos hipotézisek kiértékelése egymástól
függetlenül futó szálakban.

---

## Monitor Object

A Half-Sync/Half-Async Queuing Layer-e Monitor Object-tel van implementálva.
A szenzor adatokat fogadó aszinkron réteg és az ICP-t futtató szinkron réteg
ugyanahhoz a sorhoz fér hozzá párhuzamosan. A Monitor Object garantálja, hogy
egyszerre csak egy szál írhat vagy olvashat belőle, condition variable-ekkel
jelezve, ha új adat érkezett vagy ha a sor megtelt.

Ez nem opcionális: ha a két réteg közt nincs szinkronizált sor, adatsérülés vagy
race condition keletkezhet a szenzor adatok feldolgozásakor.

---

## Összefoglalás szóbelire egy mondatban minden patternről

- **Wrapper Facade**: elrejti a HoloLens, Quest 3 és Unitree platform-specifikus
  API-jait egy egységes szenzor interfész mögé.
- **Component Configurator**: a szenzorfeldolgozó komponensek futás közben
  cserélhetők, a rendszer többi részének újraindítása nélkül.
- **Interceptor**: az MRTK hook-ok és a konfidencia score számítás automatikusan
  fut le minden releváns eseménynél, a pipeline módosítása nélkül.
- **Extension Interface**: egy fizikai tér objektum egyszerre több szerepet tölt be,
  minden kliens csak a saját interfészét látja, Unity GameObject-szerűen.
- **Reactor**: az eseményvezérelt bemenetek (scan, ICP kész, UI) egyetlen
  eseménycikluson keresztül, nem blokkolva érkeznek.
- **Half-Sync/Half-Async**: a hosszan futó ICP és konfidencia számítás
  háttérszálban fut, a UI és robot vezérlés nem fagy le.
- **Monitor Object**: a szenzor adatokat fogadó és az ICP-t futtató réteg közt
  szinkronizált sor biztosítja az adatbiztonságot.