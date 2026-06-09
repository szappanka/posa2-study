---
layout: default
title: Thread-Specific Storage
---

# Thread-Specific Storage (Szálspecifikus Tárolás / Local Telephone Directory Service)

## Mi ez a pattern

A Thread-Specific Storage mintával a többszálú alkalmazás logikai globális hozzáférési ponttal éri el az objektumait és adatait. Az objektumok és adatok fizikailag lokálisak a szál számára, így elkerülhetjük a szinkronizációs overhead-et.

Ez a Concurrency kategória ötödik és egyben utolsó patternje. Más nevén: Thread-Local Storage.

---

## A probléma

A többszálú alkalmazások egyik nehézsége a konkurenciakezelés. A szinkronizációs overhead elronthatja a többszálúságból eredő teljesítményelőnyöket.

Bizonyos adatokat minden szálnak saját példányban kellene kezelnie, de globálisan elérhetőnek kellene lenniük. A klasszikus példa az `errno`: minden szálnak saját `errno` értéke kell, hogy a más szálban lefutó függvények ne írják felül egymás hibakódjait.

Ha erre egy globális változót használunk, szinkronizálni kell. Ha lokális változót, akkor nem érhető el globálisan. Ha minden függvénynek paraméterként adjuk át, az összes meglévő API-t meg kell változtatni.

---

## A megoldás

Definiáljunk globális hozzáférési pontot minden szálspecifikus objektumhoz és adathoz, de tartsuk karban a tényleges objektumot egy olyan tárolóban, ami lokális a szál szempontjából. Az alkalmazások kezeljék a szálspecifikus objektumokat kizárólag a globális hozzáférési pontokon keresztül.

Gondolj rá úgy, mint a helyi telefonkönyv-szolgáltatásra. Az USA-ban a 411-es szám logikailag globális: mindenki ugyanazt a számot hívja. Mégis mindenki a saját körzete szerint kap eltérő helyi információt. A hozzáférési pont globális, az adat lokális.

---

## Szereplők

A patternnek öt szereplője van:

**Thread-Specific Object**: egy konkrét objektum, amelyet kizárólag egy adott szál érhet el. Fizikailag a szál saját tárolójában él. Például az `errno` értéke minden szálban külön egész szám.

**Thread-Specific Object Set**: az egy szálhoz tartozó összes szálspecifikus objektum gyűjteménye. Minden szálnak saját Object Set-je van. `set()` és `get()` metódusokkal érhető el, amelyeknek egy Key-t adunk át paraméterként.

**Key**: a szálspecifikus objektumot azonosító kulcs. Az összes szál ugyanazt a Key értéket használja az adott objektum eléréséhez, de a Key mögött minden szálban más tényleges objektum áll. A Key Factory osztja ki a Key értékeket.

**Key Factory**: gondoskodik arról, hogy minden kulcs globálisan egyedi legyen. POSIX-on ez a `pthread_key_create()`, Win32-n a `TlsAlloc()`.

**Thread-Specific Object Proxy**: lehetővé teszi a kliensek számára, hogy egy szálspecifikus objektumot úgy érjenek el, mintha közönséges objektum lenne. A Proxy tárolja a Key-t, és a metódushíváskor lekéri az aktuális szál Thread-Specific Object Set-jéből a megfelelő objektumot, majd delegál rá.

---

## Struktúra

A pattern résztvevői egy kétdimenziós mátrixként modellezhetők:

```
              Szál 1      Szál 2      Szál 3
Key 0 (errno) [int *]     [int *]     [int *]
Key 1 (log)   [Logger *]  [Logger *]  [Logger *]
Key 2 (conn)  [Conn *]    [Conn *]    [Conn *]
```

Minden sor egy Key-nek felel meg (logikailag globális), minden oszlop egy szálnak. A mátrix egy cellájában a szál saját, szálspecifikus objektuma áll.

A Key létrehozása új sort jelent a mátrixban. Új szál létrehozása új oszlopot jelent. A Thread-Specific Object Proxy a (k, t) cellát éri el: a k. Key-t a t. szál számára.

<svg viewBox="0 0 680 280" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Application Thread -->
  <rect x="20" y="100" width="120" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="80" y="123" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Application</text>
  <text x="80" y="139" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Thread</text>
  <text x="80" y="153" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">kliens</text>

  <!-- Proxy -->
  <rect x="200" y="100" width="130" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="265" y="123" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">TS Object</text>
  <text x="265" y="139" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Proxy</text>
  <text x="265" y="153" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">tárolja a Key-t</text>

  <!-- Key Factory -->
  <rect x="200" y="20" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="265" y="38" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Key Factory</text>
  <text x="265" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">pthread_key_create()</text>

  <!-- Object Set -->
  <rect x="400" y="60" width="140" height="100" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="470" y="82" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">TS Object Set</text>
  <text x="470" y="98" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">szálonként egy</text>
  <text x="470" y="114" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">set(key, obj)</text>
  <text x="470" y="130" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">get(key) -> obj</text>
  <text x="470" y="148" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">pthread_getspecific()</text>

  <!-- TS Object -->
  <rect x="580" y="100" width="80" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="620" y="118" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">TS Object</text>
  <text x="620" y="134" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">szálhoz tartozó</text>

  <!-- Arrows -->
  <line x1="140" y1="130" x2="200" y2="130" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="170" y="124" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">hív</text>

  <line x1="265" y1="100" x2="265" y2="64" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="278" y="85" font-size="9" fill="var(--color-text-secondary)">kér Key-t</text>

  <line x1="330" y1="130" x2="400" y2="110" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="360" y="115" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">get(key)</text>

  <line x1="540" y1="120" x2="580" y2="120" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="560" y="114" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">visszaad</text>
</svg>

---

## Működés

A működés két szcenárióból áll:

**1. szcenárió: szálspecifikus objektum első elérése**

1. Az Application Thread meghívja a Proxy interfész metódusát
2. A Proxy lekérdezi a Thread-Specific Object Set-et a Key segítségével
3. Ha az objektum még nem létezik ebben a szálban, a Proxy létrehozza és berakja az Object Set-be a `set()` metódussal
4. A Proxy delegál a most visszakapott objektumra

**2. szcenárió: szálspecifikus objektum ismételt elérése**

1. Az Application Thread meghívja a Proxy interfész metódusát
2. A Proxy lekérdezi a Thread-Specific Object Set-et a Key segítségével
3. Az objektum már létezik, a `get()` azonnal visszaadja
4. A Proxy delegál az objektumra, szinkronizáció nélkül

---

## Implementáció lépései

**1. A Thread-Specific Object Set meghatározása**

Két stratégia létezik a Thread-Specific Object Set tárolására:

- **External**: az Object Set a szál adatszerkezetein kívül, globálisan tárolt adatstruktúrában él, ahol minden szál azonosítóját kulcsként használjuk. Egyszerűbb implementáció, de egy extra szintű indirectiont igényel.
- **Internal**: az Object Set közvetlenül a szál belső állapotában tárolódik, például a POSIX `pthread_t` struktúrában. Hatékonyabb, mert nem kell szálidentifikátor alapján keresni.

**2. A Key Factory implementálása**

A Key Factory globálisan számolja ki a kulcsokat. POSIX-on:

```cpp
typedef int pthread_key_t;
static pthread_key_t total_keys_ = 0;

int pthread_key_create(pthread_key_t *key,
                       void (*thread_exit_hook)(void *)) {
    if (total_keys_ >= _POSIX_THREAD_KEYS_MAX) {
        INTERNAL_ERRNO = ENOMEM;
        return -1;
    }
    thread_exit_hook_[total_keys_] = thread_exit_hook;
    *key = total_keys_++;
    return 0;
}
```

A `thread_exit_hook` paraméter egy callback, amelyet a Pthreads könyvtár automatikusan meghív, amikor a szál kilép, hogy felszabadítsa a szálspecifikus objektumot.

**3. A Thread-Specific Object Proxy implementálása**

A Proxy template-ként implementálható C++-ban, amely az `operator->` operátort definiálja:

```cpp
template <class TYPE>
class TS_Proxy {
public:
    TYPE *operator->() const {
        TYPE *tss_data = 0;
        // Double-Checked Locking: Key létrehozása csak egyszer
        if (!once_) {
            Guard<Thread_Mutex> guard(keylock_);
            if (!once_) {
                pthread_key_create(&key_, cleanup_hook);
                once_ = true;
            }
        }
        // Szálspecifikus objektum lekérése
        pthread_getspecific(key_, (void **) &tss_data);
        if (tss_data == 0) {
            tss_data = new TYPE;
            pthread_setspecific(key_, (void *) tss_data);
        }
        return tss_data;
    }
private:
    mutable pthread_key_t key_;
    mutable bool once_;
    mutable Thread_Mutex keylock_;
    static void cleanup_hook(void *ptr) { delete (TYPE *) ptr; }
};
```

<div class="callout tip" markdown="1">
**Tipp:** a Key létrehozásához Double-Checked Locking Optimization-t kell alkalmazni, mert a Key-t csak egyszer szabad létrehozni, de a Proxy-t esetleg több szál is eléri egyszerre az első híváskor. Az objektum `pthread_getspecific()` és `pthread_setspecific()` hívások viszont nem igényelnek szinkronizációt, mert szálspecifikus adatot kezelnek.
</div>

---

## Ismert felhasználások

**errno (POSIX és Win32)**: a legklasszikusabb példa. A Solaris `<errno.h>`-ban az `errno` egy makró, amely egy `___errno()` függvényt hív meg, amely `pthread_getspecific()`-kel éri el a szál saját `errno` értékét. Win32-n a `GetLastError()` és `SetLastError()` ugyanezt valósítja meg.

**Win32 ablakkezelés**: a Win32-ben az ablakok a szál tulajdonában vannak. Minden ablaktulajdonos szálnak saját message queue-ja van thread-specific storage-ban, ahol az OS sorba helyezi a UI eseményeket.

**Java ThreadLocal**: a `java.lang.ThreadLocal` osztály a Thread-Specific Storage pattern Java implementációja. A `ThreadLocal` objektum a Proxy szerepét tölti be. Statikus változóként szokás deklarálni, hogy globálisan látható legyen.

```java
// Logikailag globális, fizikailag szálspecifikus
static ThreadLocal<Connection> dbConnection = new ThreadLocal<>() {
    protected Connection initialValue() {
        return createConnection();
    }
};

// Minden szál a saját Connection-jét kapja
Connection conn = dbConnection.get();
```

**OpenGL**: az OpenGL állapotváltozóit thread-specific storage-ban tárolja Win32-n. Minden szál saját OpenGL állapottal rendelkezik, ami szükséges, ha több szál párhuzamosan renderel.

**ACE framework**: az `ACE_TSS` template osztály a pattern referencia-implementációja, amelyet az ACE hibakezelési rendszerében alkalmaz.

---

## Valós példa

Egy webszerver minden beérkező HTTP kéréshez egy munkaszálat rendel. Minden munkaszálnak szüksége van egy adatbázis-kapcsolatra, egy naplózóra és egy hibakódra.

Thread-Specific Storage nélkül: globális adatbázis-kapcsolat, amit minden szál zárral véd. Nagy terhelésnél a zárolás szűk keresztmetszetté válik.

Thread-Specific Storage-gal:

```cpp
// Minden szálnak saját adatbázis-kapcsolata van
TS_Proxy<DatabaseConnection> db_conn;
TS_Proxy<Logger> thread_logger;

void handle_request(Request req) {
    // Automatikusan a hívó szál saját példányát adja vissza
    db_conn->execute(req.query());   // nincs zárolás
    thread_logger->log(req.path()); // nincs zárolás
}
```

Mindhárom erőforrás szálspecifikus: minden szál csak a saját példányával dolgozik, nincs versenyhelyzet, nincs szinkronizáció.

Ugyanez a minta jelenik meg játékszerverek esetén: minden játékos-kezelő szálnak saját véletlenszám-generátora van thread-specific storage-ban, így nincs szükség a PRNG állapotát mutex-szel védeni.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Hatékonyság | Szigorú (nem minden adat szálspecifikus) |
| Újrafelhasználhatóság | Implementáció érzékenység |
| Egyszerű használat | |
| Hordozhatóság | |

**Előnyök részletesen:**

**Hatékonyság**: szálspecifikus adathoz való hozzáférés nem igényel zárolást, mert a versenyfeltételek eleve kizártak. A `pthread_getspecific()` és `pthread_setspecific()` hívások gyorsak, mert az OS közvetlenül a szál belső állapotából olvassa az adatot.

**Újrafelhasználhatóság**: a Thread-Specific Object Proxy template-ként implementálható, amely bármely típusú objektumot szálspecifikussá tehet anélkül, hogy az objektum kódját módosítani kellene.

**Egyszerű használat**: az alkalmazások a Proxy-n keresztül úgy kezelik a szálspecifikus objektumokat, mintha közönséges objektumok lennének. A szálspecifikusság transzparens marad.

**Hordozhatóság**: az OS-specifikus API-k (pthread_key_create, TlsAlloc) Wrapper Facade-del elrejthetők, így az alkalmazáskód platformfüggetlen marad.

**Hátrányok részletesen:**

**Szigorú**: a pattern csak olyan adatokra alkalmazható, amelyeket szálak nem osztanak meg egymással. Ha az adatot mégis meg kellene osztani, a Thread-Specific Storage nem megfelelő megoldás.

**Implementáció érzékenység**: a Key Factory nem portábilis minden platformon: például Solaris nem biztosít API-t a Key felszabadítására, ami erőforrás-szivárgáshoz vezethet. A Proxy destruktora is kényes, mert nehéz portábilisan felszabadítani a Key-t.

---

## Kapcsolata más patternekkel

A **Double-Checked Locking Optimization** pattern a Key létrehozásához szükséges: a kulcsot csak egyszer szabad inicializálni, amit a Double-Checked Locking garantál minimális overhead-del.

A **Wrapper Facade** pattern elfedi a platform-specifikus thread-specific storage API-kat (`pthread_key_create`, `TlsAlloc`) a hordozható implementáció érdekében.

A **Extension Interface** pattern alternatív megközelítést nyújt a Thread-Specific Object Proxy-hoz: az ATL ezt a megközelítést alkalmazza a TSS implementálásához COM-ban.

A **Proxy pattern** (GoF, POSA1) az alapja a Thread-Specific Object Proxy-nak, amely átlátszó hozzáférést biztosít a szálspecifikus objektumokhoz.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Thread-Specific Storage logikai globális hozzáférési pontot biztosít szálspecifikus objektumokhoz, amelyek fizikailag a szál saját tárolójában élnek. Öt szereplője van: Thread-Specific Object, Thread-Specific Object Set, Key, Key Factory és Thread-Specific Object Proxy. A Key Factory globálisan egyedi kulcsokat oszt ki. Minden szálnak saját Object Set-je van, amelyből a Key segítségével érhető el a szálhoz tartozó objektum. A Proxy elrejti ezt a mechanizmust, és közönséges objektumként használható. Szinkronizáció nélkül működik, mert a versenyhelyzet eleve kizárt. Leghíresebb példája az errno. Előnye a hatékonyság és az egyszerű használat. Hátránya, hogy csak szálak között nem megosztott adatokra alkalmazható.
</div>