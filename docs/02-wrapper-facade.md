---
layout: default
title: Wrapper Facade
---

# Wrapper Facade (Homlokzat/Csomagoló)

## Mi ez a pattern

![Facade design pattern](https://refactoring.guru/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b)

A Wrapper Facade <mark>alacsony szintű, nem objektumorientált API-kat csomagol be OOP osztályokba</mark>. Az alkalmazás ezután nem az OS API-val kommunikál közvetlenül, hanem ezzel az osztállyal, szóval egy <mark>magasabb szintű interface</mark>t hozunk létre.

Ez az <mark>első és legalapvetőbb pattern</mark> a könyvben. Szinte minden más pattern (Reactor, Proactor, Active Object, Strategized Locking) erre épül, mert mindegyiknek kell valamilyen OS-szintű hozzáférés.

---

## A probléma

Az operációs rendszerek alacsony szintű C függvényeken keresztül adnak hozzáférést alapvető erőforrásokhoz, például szálakhoz, hálózati kapcsolatokhoz és zárakhoz. Ezek a függvények platformonként teljesen mások.

Vegyük a **mutex** fogalmát. A mutex (kölcsönös kizárás) <mark>egy olyan zár, amellyel meg lehet akadályozni, hogy két szál egyszerre módosítson közös adatot</mark>. Például ha két szál egyszerre próbálja növelni ugyanazt a számlálót, az adatsérüléshez vezet. A mutex megoldja ezt: az első szál lezárja, csinálja a dolgát, majd feloldja. A második addig vár. Ez így működik minden operációs rendszeren, de a konkrét függvények teljesen mások platformonként.

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

A Socket API is hasonlóan problémás. A socket egy hálózati kapcsolat végpontja, nagyjából olyan, mint egy telefon csatlakozó: ezen keresztül küld és fogad az alkalmazás adatot a hálózaton. A Socket API több tucat C függvénnyel dolgozik egységes névkonvenció nélkül. Nem nyilvánvaló például, hogy a `socket()`, `bind()`, `listen()`, `connect()` és `accept()` egymáshoz tartoznak, és pontosan ebben a sorrendben kell őket hívni.

Ha ezeket az API-kat közvetlenül használod az alkalmazásban:

- platformváltáskor az egész alkalmazást át kell írni
- a fordító nem tud típusellenőrzést végezni, mert egy socket handle egyszerű `int` számként van ábrázolva, és semmilyen szabály nem akadályozza meg, hogy véletlenül egy másik `int`-et adj át helyette
- semmi nem akadályozza meg, hogy rossz sorrendben hívd a függvényeket
- könnyű elfelejteni a cleanup hívásokat, mint a `mutex_destroy`, ami erőforrás-szivárgáshoz vezet

---

## A megoldás

![Telefóniás analógia](https://refactoring.guru/images/patterns/diagrams/facade/live-example-en.png?id=461900f9fbacdd0ce981dcd24e121078)

<mark>Minden összefüggő C függvénycsoport köré írj egy OOP osztályt.</mark> Az alkalmazás csak ezzel az osztállyal kommunikál, az <mark>OS-specifikus részletek az osztályon belül maradnak</mark>.

Gondolj rá úgy mint egy telefonos ügyfélszolgálatra: a hívó nem tudja és nem is kell tudnia, hogy a háttérben milyen rendszerek futnak. Az operátor elintézi a közvetítést.

A struktúrának két résztvevője van:

**Functions**: <mark>a meglévő, nem OOP API függvényei</mark>. Önállóan nyújtanak szolgáltatást, globális változókon vagy paramétereken keresztül kezelik az adatot.

**Wrapper Facade**: <mark>egy vagy több OOP osztály, amely ezeket a függvényeket és a hozzájuk tartozó adatot becsomagolja, és egy összefüggő interfészt exportál</mark>. Az osztály metódusai a hívásokat a megfelelő alacsony szintű függvényekre irányítják tovább.

---

## Struktúra

![Structure of the Facade design pattern](https://refactoring.guru/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7)

![Structure of the Facade design pattern indexed](https://refactoring.guru/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8)

1. A **Facade** kényelmes hozzáférést biztosít az alrendszer egy részéhez. Tudja, hová kell irányítani a kéréseket.
2. **Additional Facade** is létrehozható, ha egy Facade túl sok, egymáshoz nem kapcsolódó funkciót gyűjt össze. Ezt a kliens és más Facade-ek is használhatják.
3. A **Complex Subsystem** az összetett, alacsony szintű osztályokból áll. Nem tud a Facade létezéséről, közvetlenül kommunikál egymással.
4. A **Client** a Facade-en keresztül éri el a rendszert, nem közvetlenül.

---

## Konkrét példa a könyvből

![Facade example](https://refactoring.guru/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced)

A könyv végig egy logging szerver példán mutatja be a patterneket. Ebben a szerverben szálak futnak párhuzamosan, és közös adathoz nyúlnak. Ehhez mutexet kell használni.

A `Thread_Mutex` egy Wrapper Facade osztály, amely becsomagolja a Solaris mutex API-t. A neve azt jelenti: szálak közötti kölcsönös kizárás (Thread Mutex).

```cpp
class Thread_Mutex {
public:
    // Konstruktor: létrehozza és inicializálja a mutexet
    Thread_Mutex()  { mutex_init(&mutex_, USYNC_THREAD, 0); }
    // Destruktor: automatikusan felszabadítja, nem lehet elfelejteni
    ~Thread_Mutex() { mutex_destroy(&mutex_); }
    // Zár megszerzése: a szál megvárja, amíg szabad lesz
    void acquire()  { mutex_lock(&mutex_); }
    // Zár elengedése: más szálak is hozzáférhetnek
    void release()  { mutex_unlock(&mutex_); }
private:
    mutex_t mutex_; // a tényleges Solaris mutex, el van rejtve
};
```

Ugyanez az osztály Windowson belül teljesen más kóddal működik, de az interfész, tehát az `acquire()` és `release()` metódusok neve, pontosan ugyanaz marad:

```cpp
class Thread_Mutex {
public:
    Thread_Mutex()  { InitializeCriticalSection(&mutex_); }
    ~Thread_Mutex() { DeleteCriticalSection(&mutex_); }
    void acquire()  { EnterCriticalSection(&mutex_); }
    void release()  { LeaveCriticalSection(&mutex_); }
private:
    CRITICAL_SECTION mutex_; // a tényleges Windows mutex
};
```

Az alkalmazás mindkét esetben pontosan ugyanígy használja:

```cpp
Thread_Mutex m;
m.acquire();
// ide kerül a védett kód, amit egyszerre csak egy szál futtathat
m.release();
```

A Socket API-ra ugyanez az elv vonatkozik. A `SOCK_Acceptor` Wrapper Facade osztály konstruktora garantálja, hogy a `socket()`, `bind()` és `listen()` függvények mindig a helyes sorrendben hívódnak meg. Az alkalmazásnak nem kell ezt fejben tartania:

```cpp
class SOCK_Acceptor {
public:
    // A konstruktor elvégzi a helyes sorrendű inicializálást
    SOCK_Acceptor(const INET_Addr &addr) {
        handle_ = socket(PF_INET, SOCK_STREAM, 0);
        bind(handle_, addr.addr(), addr.size());
        listen(handle_, 5);
    }
    void accept(SOCK_Stream &s) {
        s.set_handle(accept(handle_, 0, 0));
    }
private:
    SOCKET handle_;
};
```

---

## Escape hatch mechanizmus

Előfordulhat, hogy a becsomagolt interfész nem fed le minden szükséges funkciót. Erre megoldás az escape hatch: <mark>egy kontrollált kiskapu, amelyen keresztül az alkalmazás szükség esetén hozzáférhet a belső implementációhoz</mark>.

A neve onnan jön, hogy vészkijáratot biztosít az absztrakcióból, ha arra mégis szükség lenne.

```cpp
class SOCK_Stream {
public:
    // Escape hatch: szükség esetén el lehet kérni a nyers socket handle-t
    void set_handle(SOCKET h) { handle_ = h; }
    SOCKET get_handle() const { return handle_; }
};
```

<div class="callout trap" markdown="1">
**Csapda:** Az escape hatch-et ritkán szabad használni, mert visszahozza azokat a problémákat, amelyeket a Wrapper Facade megoldott. Ha az alkalmazásban sokszor előkerül ugyanaz az escape hatch, az azt jelzi, hogy az adott funkciót be kell venni a Wrapper Facade publikus interfészébe.
</div>

---

## Előnyök

**Hordozhatóság és karbantarthatóság**: <mark>a platform-specifikus kód egyetlen helyre kerül</mark>. Ha új OS-re kell portolni, csak a Wrapper Facade implementációját kell átírni, az alkalmazáskód változatlan marad.

**Robusztusabb interfész**: az encapsuláció megszünteti a <mark>típusnélküli adatstruktúrák helytelen használatából eredő hibákat</mark>. Egy socket handle például eddig egyszerű `int` volt, amit véletlenül is el lehetett rontani. A Wrapper Facade osztályban ez egy erősen típusos privát adat, amit a fordító véd.

**Automatikus erőforrás-kezelés**: <mark>a konstruktor elvégzi az inicializálást, a destruktor a felszabadítást</mark>. A programozó nem felejtheti el a `destroy()` hívást, mert az automatikusan megtörténik, amikor az objektum kilép a hatókörből.

**Moduláris és újrafelhasználható**: az OOP osztályok öröklődéssel és template-ekkel beépíthetők más komponensekbe. C függvénycsoportokkal ezt sokkal nehezebb elérni.

---

## Hátrányok

**Funkcionalitásveszteség**: egy magas szintű absztrakció <mark>elfedhet egyes alacsony szintű képességeket</mark>. Megoldás az escape hatch mechanizmus, de ezt ritkán szabad alkalmazni.

**Teljesítményveszteség**: az <mark>extra metódushívás plusz indirectiont jelent</mark>, vagyis a kód nem közvetlenül az OS függvényt hívja, hanem előbb belép a Wrapper Facade-be. Ez <mark>inlining-gal megszüntethető</mark>: a fordító behelyettesíti a metódus törzsét közvetlenül a hívás helyére, mintha az OS függvény lett volna ott.

**Nyelvi korlátok**: C++ és Java esetén az integráció viszonylag egyszerű. Ada, Smalltalk és más nyelveken nem létezik általánosan elfogadott szabvány a C függvények beillesztésére.

---

## Kapcsolata más patternekkel

<div class="callout trap" markdown="1">
**Csapda:** A Wrapper Facade könnyen összekeverhető a GoF Facade, Adapter, Bridge és Decorator patternekkel.

**Facade (GoF)** vs **Wrapper Facade**: a GoF Facade OOP osztályok közötti komplex kapcsolatokat rejt el egy egyszerűbb interfész mögé. A Wrapper Facade ennél specifikusabb: nem OOP objektumokat, hanem C függvényeket emel OOP szintre.

**Adapter**: két meglévő OOP interfészt illeszt össze, általában egyetlen objektum köré. A Wrapper Facade nem OOP API-t emel OOP szintre, és általában függvénycsoportokat kezel egyszerre.

**Bridge**: az absztrakciót és implementációt futásidőben választja szét polimorfizmussal, tehát dinamikusan cserélhető az implementáció. A Wrapper Facade ritkán variálódik dinamikusan, általában fordítási időben dől el, melyik platform-specifikus implementáció kerül bele.

**Decorator**: futásidőben bővíti az objektumot új viselkedéssel. A Wrapper Facade ezzel szemben statikusan csomagolja be a C függvényeket.
</div>

A **Layers pattern** (POSA1) segít <mark>több Wrapper Facade-et egy külön rétegbe szervezni az OS és az alkalmazás közé</mark>, így az alkalmazás egyáltalán nem kerül közvetlen kapcsolatba az OS API-val.

---

## Valós felhasználás

Képzeld el, hogy írsz egy egyszerű 2D játékot C++-ban. A játéknak hangot kell lejátszania, amikor a karakter ugrik.

Windowson a hang lejátszása így néz ki:

```cpp
PlaySound("jump.wav", NULL, SND_FILENAME);
```

Linuxon ez teljesen más:

```cpp
Mix_PlayChannel(-1, jump_sound, 0);
```

Ha ezeket közvetlenül hívod a játék kódjában, akkor amikor portolod a játékot a másik platformra, minden egyes hanglejátszó sort meg kell keresned és átírnod. Ha húsz helyen játszol le hangot, húsz helyen kell módosítanod.

A Wrapper Facade megoldása: írsz egy `Sound` osztályt.

```cpp
Sound jump("jump.wav");
jump.play();
```

A `play()` metódus belsejében van a platform-specifikus kód. <mark>A játék többi része nem látja, nem tudja, és nem is érdekli, hogy éppen Windowson vagy Linuxon fut.</mark> Ha portolni kell, csak a `Sound` osztály belsejét írod át, a játékkód változatlan marad.

Ugyanez előfordul:

**Fájlkezelésben**: Windowson az útvonalak fordított perjelet használnak (`\`), Linuxon normál perjelet (`/`). Egy `File` Wrapper Facade osztály elrejti ezt, és te csak annyit írsz, hogy `file.open("saves/slot1")`.

**Ablakkezelésben**: ha valaha használtál SDL-t vagy SFML-t (népszerű játékfejlesztő könyvtárak), akkor Wrapper Facade-eket használtál. Az `SDL_Window` például becsomagolja a Win32 `CreateWindow` hívást és a Linux X11 `XCreateWindow` hívást. Te csak annyit írsz, hogy `SDL_CreateWindow(...)`, és fut mindenhol.

**Időmérésben**: Windowson a pontos idő lekérése `QueryPerformanceCounter`, Linuxon `clock_gettime`. Egy `Timer` Wrapper Facade osztály mögé elrejted ezt, és mindenhol `timer.elapsed()` hívással mérsz időt.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Wrapper Facade alacsony szintű, nem objektumorientált API-kat, jellemzően OS szintű C függvényeket csomagol be OOP osztályokba. Az alkalmazás ezután nem az OS API-val, hanem ezzel az osztállyal kommunikál. Ez hordozhatóságot, típusbiztonságot és automatikus erőforrás-kezelést biztosít. Hátránya, hogy bizonyos alacsony szintű funkciókat elfedhet, erre megoldás az escape hatch mechanizmus. Ez a pattern az alapja szinte minden más patternnek a könyvben.
</div>