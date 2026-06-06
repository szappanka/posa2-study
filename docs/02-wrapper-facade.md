# Wrapper Facade

> **Kategória:** Service Access & Configuration  
> **Egy mondatban:** Becsomagolja a csúnya C API-kat OOP osztályokba, hogy a kód hordozható, biztonságos és karbantartható legyen.

---

## A probléma

Az operációs rendszer alacsony szintű C függvényeken keresztül ad hozzáférést a szolgáltatásaihoz — szálakhoz, mutexekhez, socketekhez. Ezeknek három nagy bajuk van:

**1. Platformfüggők** — ugyanaz a művelet Windowson és Linuxon teljesen másképp néz ki:

```c
// Linux (POSIX)
pthread_mutex_lock(&m);

// Windows
WaitForSingleObject(hMutex, INFINITE);
```

**2. Hibára hajlamosak** — semmi nem kényszerít rá, hogy felszabadítsd az erőforrást. Ha elfelejtesz egy `destroy()` hívást, memory leak lesz.

**3. Nem típusbiztosak** — a socket handle egy sima `int` — átadhatsz bármit, a fordító nem szól.

---

## A megoldás

Írj egy OOP osztályt, ami elrejti ezeket a C függvényeket. Az alkalmazás csak ezzel az osztállyal kommunikál — soha nem az OS API-val közvetlenül.

```cpp
// Így NE:
pthread_mutex_t m;
pthread_mutex_init(&m, NULL);
pthread_mutex_lock(&m);
// ... kritikus szekció
pthread_mutex_unlock(&m);
pthread_mutex_destroy(&m);   // ← ha ezt elfelejtjük: memory leak

// Így IGEN — Wrapper Facade:
Thread_Mutex m;
m.acquire();
// ... kritikus szekció
m.release();
// destruktor automatikusan hívja a destroy()-t
```

---

## Struktúra — kik a résztvevők?

Csak **két résztvevő** van:

| Résztvevő | Mi ez? | Példa |
|---|---|---|
| **Functions** | A meglévő C API függvények | `pthread_mutex_lock()`, `socket()`, `bind()` |
| **Wrapper Facade** | Az OOP osztály, ami becsomagolja őket | `Thread_Mutex`, `SOCK_Stream` |

A Wrapper Facade metódusai egyszerűen továbbítják a hívásokat a C függvényeknek — de ez az alkalmazás elől el van rejtve.

---

## Előnyök

- **Hordozható** — ugyanaz a kód fut Windowson és Linuxon, mert a platformspecifikus különbségek a Wrapper Facade-ben vannak elrejtve
- **Típusbiztos** — a fordító ellenőrzi, nem adhatsz át rossz típust
- **Automatikus erőforrás-kezelés** — a konstruktor inicializál, a destruktor felszabadít, nem lehet elfelejteni
- **Cserélhető** — ha új OS-re kell portolni, csak a Wrapper Facade implementációját cseréled, az alkalmazást nem

---

## Hátrányok

- **Elveszíthet egyes funkciókat** — ha a Wrapper Facade nem tesz elérhetővé minden alacsony szintű funkciót, nem tudod használni. Megoldás: **escape hatch mechanizmus** — egy metódus, amivel lekérheted a nyers handle-t szükség esetén (pl. `get_handle()`)
- **Kis teljesítményveszteség** — az extra metódushívás overhead-et jelent. Megoldás: **inlining** — a fordító inline-olja a hívásokat, így nincs különbség

---

## Escape hatch — mikor kell?

Ha a Wrapper Facade elfed egy szükséges funkciót, az alkalmazás nem tudja használni. Ilyenkor kell egy "vészkijárat":

```cpp
class SOCK_Stream {
public:
    // Normál Wrapper Facade interface
    int send(const char* buf, size_t len);
    int recv(char* buf, size_t len);

    // Escape hatch — csak szükség esetén!
    SOCKET get_handle() const { return handle_; }
    void set_handle(SOCKET h) { handle_ = h; }
};
```

> ⚠️ Az escape hatch-et csak indokolt esetben használd — csökkenti a hordozhatóságot.

---

## Valós példák

| Példa | Mi a Wrapper Facade? | Mit csomagol be? |
|---|---|---|
| Java `ReentrantLock` | A Lock interfész implementációja | OS mutex |
| .NET `System.Threading.Mutex` | A Mutex osztály | Win32 `CreateMutex` / POSIX `pthread_mutex` |
| ACE `Thread_Mutex` | C++ osztály | POSIX `pthread_mutex` vagy Win32 mutex |
| ACE `SOCK_Stream` | C++ osztály | BSD Socket API (`send`, `recv`, `close`) |
| Java `java.io.FileInputStream` | A stream osztály | OS file descriptor |

---

## Kapcsolódó patternek

- **Component Configurator** — a Wrapper Facade-del csomagolt komponenseket dinamikusan tölti be
- **Strategized Locking** — a Wrapper Facade-del csomagolt mutexeket stratégiaként alkalmazza
- **Reactor, Proactor, Active Object** — mind Wrapper Facade-et használnak belül az OS eléréséhez
- **Adapter (GoF)** — hasonló, de az Adapter OOP osztályok között dolgozik; a Wrapper Facade C függvényeket csomagol OOP-ba

---

## Hogyan különbözik a hasonló patternektől?

| Pattern | Mire való? | Különbség |
|---|---|---|
| **Facade (GoF)** | Összetett osztályhierarchiát egyszerűsít | A Wrapper Facade C függvényeket csomagol OOP-ba |
| **Adapter (GoF)** | Inkompatibilis interfészeket köt össze | Az Adapter OOP↔OOP, a Wrapper Facade C→OOP |
| **Bridge (GoF)** | Absztrakciót szétválaszt implementációtól | A Bridge dinamikusan vált; a Wrapper Facade statikus |

---

## Vizsgán mondd ezt

> „A Wrapper Facade pattern célja, hogy alacsony szintű, nem objektumorientált API-kat — például operációs rendszer szintű C függvényeket — becsomagoljon OOP osztályokba. Az alkalmazás ezután nem az OS API-val, hanem ezzel az osztállyal kommunikál. Ez hordozhatóságot, típusbiztonságot és automatikus erőforrás-kezelést biztosít. Hátránya, hogy bizonyos alacsony szintű funkciókat elfedhet — erre megoldás az escape hatch mechanizmus, ami szükség esetén hozzáférést ad a nyers handle-höz."

---

← [Vissza: Service Access & Configuration áttekintés](./01-service-access-overview.md)  
→ [Következő: Component Configurator](./03-component-configurator.md)
