---
layout: default
title: POSA2 a kutatásban
---

# POSA2 minták a saját kutatásban

Ez az oldal összefoglalja, hogy a POSA2 könyv egyes patternjeivel hogyan találkozunk a
szemantikus környezetmodellezés és a 2D point cloud regisztráció területein, és hogyan
jelennek meg ezek a valóságban Unity és MRTK alapú rendszerekben.

A két kutatási terület:

- Szemantikus környezetmodellezés kontextuális feladatvégzéshez, két platformon validálva:
  VR headset és Unitree G1 humanoid robot
- 2D point cloud regisztráció megbízhatóságának becslése belső ICP metrikákból, HoloLens 2
  és LiDAR szenzorpárral

---

## Wrapper Facade

**Miért releváns:** a HoloLens 2, a Unitree G1 robot SDK és a LiDAR szenzor mind más-más
platformspecifikus C API-kon keresztül érhető el. Ha ezeket közvetlenül hívjuk az alkalmazásból,
minden platformváltáskor az egész feldolgozó kódot át kell írni.

**Hogyan alkalmazható:** minden szenzorhoz Wrapper Facade osztályt írunk, amely elrejti a
platformspecifikus API-t. A szemantikus modellező réteg csak ezekkel az osztályokkal kommunikál,
nem a nyers szenzor API-kkal.

**Unity és MRTK kapcsolat:** az MRTK3 az OpenXR interfészt használja az alkalmazás és az XR
runtime között. Az OpenXR egy egységes, generikus API-n keresztül hívható, miközben a
hardverspecifikus implementációt az XR runtime kezeli. Ez pontosan a Wrapper Facade: az
OpenXR elrejti a HoloLens 2, Meta Quest és más eszközök eltérő OS API-jait.

A psiUnity rendszer is ezt a megközelítést követi: a HoloLens 2 Research Mode nyers
szenzorfolyamait (mélységkamerák, IMU) Wrapper Facade osztályok rejtik el a C# feldolgozó
kód elől, egységes interfészt nyújtva minden szenzortípushoz.

---

## Component Configurator

**Miért releváns:** a két platform (VR headset és robot) más szenzormodulokat igényel. VR
headseten mélységkamera és kézkövetés aktív, roboton LiDAR és proprioceptív szenzor. A
szemantikus modellező rétegnek mindkét esetben ugyanannak kell maradnia, csak a betöltött
szenzorkomponensek változnak.

**Hogyan alkalmazható:** minden szenzormodul külön DLL-be csomagolt komponens, amelyet a
Component Configurator tölt be futás közben az adott platform konfigurációja alapján. Az
alkalmazást nem kell újrafordítani platformváltáskor.

**MRTK3 kapcsolat:** az MRTK3 subsystem architektúrája pontosan ezt valósítja meg. A
subsystem-ek moduláris adatszolgáltatók, amelyek futás közben töltődnek be és cserélhetők ki.
A `MRTKLifecycleManager` kezeli a subsystem-ek életciklusát `Start`, `Stop` és `Destroy`
metódusokkal, ami megfelel a Component Configurator `init()`, `fini()` és `suspend()`
interfészének.

Egy konkrét példa: a kézkövetés subsystem HoloLens 2-n az MRTK3 saját implementációját
töltette be, Meta Questen az OpenXR HandTracking extensiont. A felső réteg kódja nem tudja
és nem is kell tudnia, melyik fut alatta.

---

## Interceptor

**Miért releváns:** a point cloud regisztrációs pipeline-ban több feldolgozási lépés van, amelyek
átláthatóan szúrhatók be a pipeline-ba anélkül, hogy a magját módosítani kellene. A confidence
score számítás, a szimmetria-ellenőrzés és a kalibrálás mind ilyen interceptor-szerű lépés.

**Hogyan alkalmazható:** a regisztrációs pipeline beavatkozási pontokat definiál az ICP futása
előtt és után. Minden interceptor (szimmetria-ellenőrzés, metrika-számítás, kalibrálás) regisztrál
erre a pontra, és automatikusan lefut, amikor az ICP eléri azt. A pipeline magját nem kell
módosítani új feldolgozási lépés hozzáadásakor.

**MRTK kapcsolat:** az MRTK input rendszer pontosan az Interceptor mintáját valósítja meg.
Minden regisztrált globális input handler kap értesítést az eseményről. Ha egy handler feldolgozottnak
jelöli az eseményt, a folyamat leáll. A kézkövetési és szemkövetési adatok feldolgozása ilyen
interceptor láncként van felépítve az MRTK3-ban.

A pipeline ablation study a cikkben lényegében azt mérte, hogy az egyes interceptorok
mennyit adnak hozzá a rendszer teljesítményéhez. A szimmetria-ellenőrző interceptor eltávolítása
a sikerességi rátát 100%-ról 78.6%-ra csökkentette.

---

## Reactor

**Miért releváns:** a VR headset és a robot egyszerre több szenzorforrásból kap adatot:
kamera, LiDAR, IMU, mélységérzékelő. Ezek mindegyike eseményként érkezik, és a rendszernek
reagálnia kell rájuk anélkül, hogy bármelyik blokkolná a másikat.

**Hogyan alkalmazható:** a Reactor egyetlen szálban figyeli az összes szenzorforrást. Ha a
mélységkamera új képkockát küld, a Reactor meghívja a megfelelő event handler-t. Ha az IMU
új adatot közöl, azt egy másik handler dolgozza fel. A szemantikus modellező kód csak az
event handler-t írja meg, a multiplexálást a Reactor kezeli.

**Unity kapcsolat:** a Unity fő eseményciklusa (Update loop) pontosan a Reactor mintáját követi:
egyetlen szálban fut, figyeli az összes bemeneti eseményt, és a megfelelő handler-t hívja meg.
Az `OnTriggerEnter`, `OnCollisionStay` és más Unity callback-ek mind Concrete Event Handler-ek.

A psiUnity rendszer a HoloLens 2 Research Mode szenzorfolyamait Reactor-elvű
eseménykezeléssel kezeli: minden szenzorfolyam külön eseményforrás, és minden adatmintához
magas pontosságú időbélyeg tartozik, amelyet a Reactor alapú dispatch megőriz.

---

## Proactor és Asynchronous Completion Token

**Miért releváns:** a cikkben leírt regisztrációs pipeline négy hipotézist értékel párhuzamosan:
két forrás-cél irány és két overlap mód kombinációja. Ha ezeket sorban futtatnánk, az
eredmény négyszer lassabb lenne. Aszinkron párhuzamos futtatással a négy hipotézis
egyszerre fut, és a legjobb confidence score-ú eredmény kerül kiválasztásra.

**Proactor alkalmazása:** a Proactor minden hipotézishez aszinkron ICP futtatást indít el. Az
operációs rendszer a háttérben futtatja őket. Amikor bármelyik befejeződik, completion event
kerül a sorba, amelyet a Proactor kivesz és a Completion Handler-nek ad.

**ACT alkalmazása:** minden aszinkron ICP futáshoz egy ACT tartozik, amely azonosítja, melyik
hipotézishez tartozik az eredmény. Amikor mind a négy befejezési esemény megérkezett, az ACT
alapján O(1) időben össze lehet gyűjteni az eredményeket és kiválasztani a legjobbat.

**Windows Completion Port kapcsolat:** a Win32 `ReadFile` overlapped I/O módban ugyanezt a
mechanizmust valósítja meg. A HoloLens 2 szenzorbeolvasás Win32 UWP felületen fut, ahol a
depth camera frame-ek aszinkron completion event-ként érkeznek vissza.

---

## Half-Sync/Half-Async

**Miért releváns:** a szemantikus modellező rendszerben az alacsony szintű szenzorbeolvasás
és a magas szintű szemantikus feldolgozás eltérő időzítési követelményekkel rendelkezik. A
szenzorbeolvasás valós idejű, a modellépítés lassabb és összetett számítást igényel.

**Hogyan alkalmazható:** az aszinkron réteg (szenzorbeolvasás) folyamatosan gyűjti az
adatokat és egy sorba helyezi őket. A szinkron réteg (szemantikus modellépítés) a sorból
fogyasztja az adatokat a saját tempójában. A két réteg egymástól függetlenül fut.

**Unity kapcsolat:** a Unity Job System és a Burst Compiler pontosan ezt a struktúrát valósítja
meg. Az alacsony szintű párhuzamos számítások (pl. pontfelhő feldolgozás) aszinkron Job-okban
futnak, az eredmény a fő szálra visszakerülve kerül be a scene graph-ba. Ez a Half-Sync/Half-Async
architektúra: az aszinkron réteg a Job System, a szinkron réteg a Unity fő szála.

A roboton (Unitree G1) ugyanez a szétválasztás kritikus: a mozgásvezérlő aszinkron, valós idejű
hurokban fut, a szemantikus modellező szinkron, magasabb szintű döntéshozói hurokban.

---

## Összefoglalás

| Pattern | Kutatási terület | Konkrét alkalmazás |
|---|---|---|
| Wrapper Facade | mindkettő | szenzor API-k elrejtése, platform-független feldolgozás |
| Component Configurator | szemantikus modellezés | szenzormodulok futás közbeni cseréje platformonként |
| Interceptor | point cloud regisztráció | szimmetria-ellenőrzés, confidence score, kalibrálás pipeline-ban |
| Reactor | szemantikus modellezés | több szenzorforrás egyszálú eseménykezelése |
| Proactor + ACT | point cloud regisztráció | négy párhuzamos ICP hipotézis aszinkron futtatása |
| Half-Sync/Half-Async | szemantikus modellezés | szenzorbeolvasás és modellépítés szétválasztása |

A patternek nem elszigetelten jelennek meg: a Wrapper Facade az alapja minden másnak,
a Component Configurator a Reactor-t vagy Proactor-t használja a betöltött komponensek
futtatásához, az Interceptor a Component Configurator-ral együtt töltődhet be dinamikusan,
a Half-Sync/Half-Async pedig a Reactor-t alkalmazza aszinkron rétegként.