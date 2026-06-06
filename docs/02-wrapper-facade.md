---
layout: default
title: Wrapper Facade
---

# Wrapper Facade (Burkoló/Homlokzat)

## Mi ez a pattern

![Facade design pattern](https://refactoring.guru/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b)

A Wrapper Facade <mark>nem objektumorientált API-k függvényeit és adatait hordozható és karbantartható OOP osztályba burkolja</mark>. Az alkalmazás ezután nem az OS API-val kommunikál közvetlenül, hanem ezzel az osztállyal, tehát egy <mark>magasabb szintű interfészt</mark> hozunk létre.

Ez az <mark>első és legalapvetőbb pattern</mark> a könyvben. Szinte minden más pattern (Reactor, Proactor, Active Object, Strategized Locking) erre épül, mert mindegyiknek kell valamilyen OS-szintű hozzáférés.

---

## A probléma

Az operációs rendszerek alacsony szintű C függvényeken keresztül adnak hozzáférést alapvető erőforrásokhoz, például szálakhoz, hálózati kapcsolatokhoz és zárakhoz. Ezeknek az API-knak három fő problémája van:

- **Alacsony szintű API használata terebélyes kódra vezet**: sok függvényt kell ismerni, rossz sorrendben is meg lehet hívni őket, és könnyen elfelejti az ember a cleanup hívásokat
- **Nem hordozható kódot eredményez**: ugyanaz a fogalom platformonként teljesen más függvénynevekkel van megvalósítva
- **Ritkán alkotnak egybefüggő csoportot**: nem nyilvánvaló, hogy mely függvények tartoznak össze

Vegyük a **mutex** fogalmát. A mutex (kölcsönös kizárás) <mark>egy olyan zár, amellyel meg lehet akadályozni, hogy két szál egyszerre módosítson közös adatot</mark>. Például ha két szál egyszerre próbálja növelni ugyanazt a számlálót, az adatsérüléshez vezet. A mutex megoldja ezt: az első szál lezárja, elvégzi a munkáját, majd feloldja. A második addig vár.

Ez így működik minden operációs rendszeren, de a konkrét függvények teljesen mások platformonként:

Solaris (egy Unix-alapú OS) mutexe:
```cpp
mutex_init(&mutex_, USYNC_THREAD, 0);
mutex_lock(&mutex_);
mutex_unlock(&mutex_);
mutex_destroy(&mutex_);
```

Windows (Win32) mutexe:
```cpp
InitializeCriticalSection(&mutex_);
EnterCriticalSection(&mutex_);
LeaveCriticalSection(&mutex_);
DeleteCriticalSection(&mutex_);
```

<mark>Ugyanaz a fogalom, teljesen különböző kód.</mark> Ha az alkalmazás ezeket közvetlenül hívja, akkor platformváltáskor az egész alkalmazást át kell írni.

A Socket API is hasonlóan problémás. A socket egy hálózati kapcsolat végpontja, ezen keresztül küld és fogad az alkalmazás adatot a hálózaton. Több tucat C függvény tartozik hozzá egységes névkonvenció nélkül. Nem nyilvánvaló például, hogy a `socket()`, `bind()`, `listen()`, `connect()` és `accept()` egymáshoz tartoznak, és pontosan ebben a sorrendben kell őket hívni.

---

## A megoldás

![Telefóniás analógia](https://refactoring.guru/images/patterns/diagrams/facade/live-example-en.png?id=461900f9fbacdd0ce981dcd24e121078)

<mark>Hozzunk létre burkoló osztályt, amely metódusaiba foglaljuk a függvényeket és adatokat.</mark> Az alkalmazás csak ezzel az osztállyal kommunikál, az OS-specifikus részletek az osztályon belül maradnak.

Gondolj rá úgy mint egy telefonos ügyfélszolgálatra: a hívó nem tudja és nem is kell tudnia, hogy a háttérben milyen rendszerek futnak. Az operátor elintézi a közvetítést.

---

## Szereplők

![Structure of the Facade design pattern](https://refactoring.guru/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7)

![Structure of the Facade design pattern indexed](https://refactoring.guru/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8)

A patternnek <mark>két szereplője van</mark>:

**Függvények (Functions)**: a meglévő, nem OOP API függvényei. Önállóan nyújtanak szolgáltatást, globális változókon vagy paramétereken keresztül kezelik az adatot.

**Wrapper Facade (Burkoló osztály)**: egy vagy több OOP osztály, amely ezeket a függvényeket és a hozzájuk tartozó adatot becsomagolja, és egy összefüggő interfészt exportál. Az osztály metódusai a hívásokat a megfelelő alacsony szintű függvényekre irányítják tovább.

---

## Működés

![Facade example](https://refactoring.guru/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced)

A működés két lépésből áll:

1. Az alkalmazás meghív egy metódust a Wrapper Facade osztályon
2. A Wrapper Facade metódus továbbítja a kérést és paramétereit az általa becsomagolt alacsony szintű API függvényeknek, átadva az esetlegesen szükséges belső adatokat

A logging szerver példájában a `Thread_Mutex` Wrapper Facade osztály becsomagolja a Solaris mutex API-t. Az alkalmazás csak az `acquire()` és `release()` metódusokat látja, a tényleges OS függvényeket nem:

```cpp
Thread_Mutex m;
m.acquire();
// ide kerül a védett kód, amit egyszerre csak egy szál futtathat
m.release();
```

Ugyanez az osztály Windowson belül teljesen más C függvényeket hív, de az interfész változatlan marad. Az alkalmazáskódot nem kell átírni.

---

## Implementáció lépései

Az implementáció négy fő lépésből áll:

**1. API függvények összetartozó csoportjainak megállapítása**

Az első feladat azonosítani, hogy a meglévő C függvények közül melyek tartoznak logikailag össze. Például a mutex függvények egy csoportot alkotnak, a socket függvények egy másikat. Ha a kód átláthatatlan és nincs logikai struktúrája, először refaktorálni kell, mielőtt Wrapper Facade-et írunk rá.

**2. Az összetartozó csoportokhoz csomagoló osztályok készítése**

Minden egybefüggő függvénycsoporthoz egy OOP osztályt írunk. Az osztály privát részén tároljuk az adatokat (pl. a nyers mutex handle-t), a publikus metódusok pedig a C függvényekre delegálnak. Ha az eredeti API nagyon sok funkciót tartalmaz, több Wrapper Facade osztályt is érdemes létrehozni a concerns szétválasztásához.

**3. Gyakori hívássorozatokhoz külön metódus készítése**

Ha az API-ban van néhány függvény, amelyeket mindig egymás után kell hívni (pl. `socket()`, `bind()`, `listen()`), ezeket érdemes egyetlen metódusba összevonni. Ez garantálja a helyes sorrendet és csökkenti a hibalehetőséget.

**4. Létrehozás és megsemmisítés automatizálása**

A konstruktor végzi az inicializálást (pl. `mutex_init`), a destruktor a felszabadítást (pl. `mutex_destroy`). A programozó nem felejtheti el a cleanup hívást, mert az automatikusan megtörténik, amikor az objektum kilép a hatókörből.

<div class="callout tip" markdown="1">
**Tipp:** az escape hatch mechanizmus szükség esetén kontrollált hozzáférést biztosít a belső implementációhoz. Ez egy `get_handle()` jellegű metódus, amellyel el lehet kérni a nyers OS handle-t. Ritkán szabad használni, mert visszahozza azokat a problémákat, amelyeket a Wrapper Facade megoldott. Ha sokszor előkerül, az azt jelzi, hogy az adott funkciót be kell venni a publikus interfészbe.
</div>

---

## Ismert felhasználások

**ACE framework**: `Thread_Mutex`, `SOCK_Stream`, `SOCK_Acceptor`, `Thread_Manager` osztályok. POSIX és Win32 thread és socket API-k Wrapper Facade-jei. A POSA2 könyv többi patternje mind ezekre épít.

**MFC (Microsoft Foundation Classes)**: a Win32 GUI API-t csomagolja OOP osztályokba.

**OWL (Borland)**: szintén Win32 API-t burkoló osztálykönyvtár.

**Java AWT és Swing**: a C natív hívásokat rejtik el OOP interfész mögé. A fejlesztő `JButton`-t és `JFrame`-et lát, nem OS-szintű ablakkezelő hívásokat.

---

## Valós példa

Képzeld el, hogy írsz egy 2D játékot. A játéknak hangot kell lejátszania, amikor a karakter ugrik. Windowson és Linuxon ez teljesen különböző C függvény. Ha ezeket közvetlenül hívod a játék kódjában, platformváltáskor minden egyes hanglejátszó sort meg kell keresned és átírnod.

A Wrapper Facade megoldása: írsz egy `Sound` osztályt, és a játék csak ezt látja:

```cpp
Sound jump("jump.wav");
jump.play();
```

<mark>A játék többi része nem tudja és nem is érdekli, hogy éppen Windowson vagy Linuxon fut.</mark> Ha portolni kell, csak a `Sound` osztály belsejét írod át.

Ugyanez előfordul:

**Fájlkezelésben**: Windowson az útvonalak fordított perjelet használnak (`\`), Linuxon normál perjelet (`/`). Egy `File` Wrapper Facade osztály elrejti ezt, és te csak annyit írsz, hogy `file.open("saves/slot1")`.

**Ablakkezelésben**: ha valaha használtál SDL-t vagy SFML-t, akkor Wrapper Facade-eket használtál. Az `SDL_Window` becsomagolja a Win32 `CreateWindow` és a Linux X11 `XCreateWindow` hívásokat. Te csak `SDL_CreateWindow(...)` hívást írsz, és fut mindenhol.

**Időmérésben**: Windowson `QueryPerformanceCounter`, Linuxon `clock_gettime`. Egy `Timer` Wrapper Facade mögé elrejted ezt, és mindenhol `timer.elapsed()` hívással mérsz időt.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Tömör, robusztus OOP interfész | A funkcionalitás esetleges csorbulása |
| Hordozhatóság és karbantarthatóság | Teljesítménycsökkenés |
| Modularitás és konfigurálhatóság | Nyelvek és fordítók korlátai |

**Előnyök részletesen:**

**Tömör, robusztus OOP interfész**: az encapsuláció megszünteti a típusnélküli adatstruktúrák helytelen használatából eredő hibákat. Egy socket handle eddig egyszerű `int` volt, amit véletlenül el lehetett rontani. A Wrapper Facade osztályban ez egy erősen típusos privát adat, amit a fordító véd.

**Hordozhatóság és karbantarthatóság**: a platform-specifikus kód egyetlen helyre kerül. Ha új OS-re kell portolni, csak a Wrapper Facade implementációját kell átírni.

**Modularitás és konfigurálhatóság**: az OOP osztályok öröklődéssel és template-ekkel beépíthetők más komponensekbe. C függvénycsoportokkal ezt sokkal nehezebb elérni.

**Hátrányok részletesen:**

**A funkcionalitás esetleges csorbulása**: egy magas szintű absztrakció elfedhet egyes alacsony szintű képességeket. Megoldás az escape hatch mechanizmus.

**Teljesítménycsökkenés**: az extra metódushívás plusz indirectiont jelent. Inlining-gal megszüntethető, mert a fordító behelyettesíti a metódus törzsét közvetlenül a hívás helyére.

**Nyelvek és fordítók korlátai**: C++ és Java esetén az integráció viszonylag egyszerű. Ada, Smalltalk és más nyelveken nincs általánosan elfogadott szabvány a C függvények beillesztésére.

---

## Kapcsolata más patternekkel

<div class="callout trap" markdown="1">
**Csapda:** A Wrapper Facade könnyen összekeverhető a GoF Facade, Adapter, Bridge és Decorator patternekkel.

**Facade (GoF)** vs **Wrapper Facade**: a GoF Facade OOP osztályok közötti komplex kapcsolatokat rejt el egy egyszerűbb interfész mögé. A Wrapper Facade ennél specifikusabb: C függvényeket emel OOP szintre.

**Adapter**: két meglévő OOP interfészt illeszt össze. A Wrapper Facade nem OOP API-t emel OOP szintre.

**Bridge**: az absztrakciót és implementációt futásidőben választja szét polimorfizmussal. A Wrapper Facade ritkán variálódik dinamikusan.

**Decorator**: futásidőben bővíti az objektumot új viselkedéssel. A Wrapper Facade statikusan csomagolja be a C függvényeket.
</div>

A **Layers pattern** (POSA1) segít <mark>több Wrapper Facade-et egy külön rétegbe szervezni az OS és az alkalmazás közé</mark>, így az alkalmazás egyáltalán nem kerül közvetlen kapcsolatba az OS API-val.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Wrapper Facade nem OOP API-k függvényeit és adatait hordozható és karbantartható OOP osztályba burkolja. Az alkalmazás ezután nem az OS API-val, hanem ezzel az osztállyal kommunikál. Két szereplője van: a Functions (a meglévő C függvények) és a Wrapper Facade osztály maga. Implementáció során az összetartozó függvényeket csoportosítjuk, burkoló osztályokat készítünk, a gyakori hívássorozatokat egyetlen metódusba vonjuk össze, és automatizáljuk a létrehozást és megsemmisítést. Előnye a hordozhatóság, a robusztus interfész és a modularitás. Hátránya a funkcionalitás esetleges csorbulása és a teljesítménycsökkenés, utóbbit inlining-gal lehet megszüntetni. Ez a pattern az alapja szinte minden más patternnek a könyvben.
</div>