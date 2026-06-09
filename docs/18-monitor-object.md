---
layout: default
title: Monitor Object
---

# Monitor Object (Monitor Objektum / Fast Food Restaurant)

## Mi ez a pattern

A Monitor Object szinkronizálja a konkurens hívásokat egy olyan lehetőség biztosításával, amellyel egyszerre csak egy metódus hajtódik végre az objektumon belül. Ezen felül kooperatív ütemezést biztosít a metódushívásoknak monitor feltételeken keresztül.

Ez a Concurrency kategória második patternje. Más nevén: Thread Safe Passive Object.

---

## A probléma

Konkurens alkalmazásokban vannak olyan objektumok, amelyek metódusait több kliens is meghívja párhuzamosan, és ezek a metódusok módosítják az objektum belső állapotát. A korrekt futtatáshoz négy kényszert kell betartani:

- az objektum interfész metódusai definiálják a szinkronizációs határokat, és egyszerre csak egy metódus lehet aktív ugyanazon az objektumon belül
- a klienseknek nem kellene explicit zárolási mechanizmusokat kezelniük, az objektum maga felelős a szinkronizációért átlátszó módon
- ha egy metódusnak blokkolnia kell a végrehajtás során, önként le kell mondania a szálvezérlésről, hogy más szálak is hozzáférhessenek az objektumhoz
- amikor egy metódus lemond a szálvezérlésről, stabil állapotban kell hagynia az objektumot

---

## A megoldás

A megosztott objektumokat monitor objektumként definiáljuk. A monitor objektumot a kliensek csak szinkronizált metóduson keresztül érhetik el. Egyszerre csak egy szinkronizált interfész metódus futhat. A szinkronizációt a monitor objektum egy monitor lock-kal oldja meg. A szinkronizált metódusok monitor feltételek kiértékelésével döntik el, mikor szakítsák meg, illetve mikor folytassák futásukat.

---

## Szereplők

A patternnek négy szereplője van:

**Monitor Object**: exportál egy vagy több metódust, amelyeken keresztül a kliensek elérhetik. A belső állapot védelme érdekében minden hozzáférés kizárólag ezeken a metóduson keresztül történik. A monitor objektumnak nincs saját szála, minden metódus a hívó kliens szálában fut.

**Synchronized Method (Szinkronizált metódus)**: a monitor objektum által exportált szálbiztos függvény. Egyszerre csak egy szinkronizált metódus futhat az objektumon belül, függetlenül attól, hány szál hívja párhuzamosan. Az interfész szinkronizált metódusai csak zárolnak és delegálnak az implementációs metódusokra, a Thread-Safe Interface pattern szerint.

**Monitor Lock (Monitor zár)**: minden monitor objektum saját monitor lock-ot tartalmaz. A szinkronizált metódusok ezzel a zárat objektumonkénti szinten sorosítják a metódushívásokat. A monitor lock-ot a metódus belépésekor meg kell szerezni és kilépéskor fel kell oldani.

**Monitor Condition (Monitor feltétel)**: a szinkronizált metódusok kooperatív ütemezésre használják. Egy metódus felfüggesztheti magát egy monitor feltételen (wait), és más metódusok értesíthetik (notify, notifyAll). A monitor lock és a monitor condition atomikusan működik együtt: a wait feloldja a lock-ot és felfüggeszti a szálat egyszerre.

---

## Struktúra

<svg viewBox="0 0 680 320" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Client threads -->
  <rect x="20" y="100" width="100" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="70" y="118" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Client T1</text>
  <text x="70" y="134" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">put(msg)</text>

  <rect x="20" y="180" width="100" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="70" y="198" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Client T2</text>
  <text x="70" y="214" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">get()</text>

  <!-- Monitor Object boundary -->
  <rect x="160" y="30" width="480" height="270" rx="12" fill="none" stroke="var(--color-border-secondary)" stroke-width="1.5" stroke-dasharray="8 4"/>
  <text x="180" y="52" font-size="10" fill="var(--color-text-secondary)">Monitor Object</text>

  <!-- Monitor Lock -->
  <rect x="180" y="65" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="245" y="83" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Monitor Lock</text>
  <text x="245" y="99" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">Thread_Mutex</text>

  <!-- Monitor Condition -->
  <rect x="340" y="65" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="410" y="85" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Monitor Condition</text>
  <text x="410" y="101" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">not_empty</text>
  <text x="410" y="116" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">not_full</text>

  <!-- Synchronized Methods -->
  <rect x="180" y="165" width="130" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="245" y="185" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Synchronized</text>
  <text x="245" y="201" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Methods</text>
  <text x="245" y="217" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">put() / get()</text>

  <!-- Internal State -->
  <rect x="500" y="165" width="120" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="560" y="188" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Belső állapot</text>
  <text x="560" y="204" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">message_count</text>
  <text x="560" y="218" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">max_messages</text>

  <!-- Arrows -->
  <line x1="120" y1="122" x2="180" y2="190" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="120" y1="202" x2="180" y2="202" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>

  <line x1="245" y1="165" x2="245" y2="109" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="255" y="142" font-size="9" fill="var(--color-text-secondary)">acquire/release</text>

  <line x1="310" y1="195" x2="340" y2="110" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="320" y="155" font-size="9" fill="var(--color-text-secondary)">wait/notify</text>

  <line x1="310" y1="195" x2="500" y2="195" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="405" y="189" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">módosítja</text>
</svg>

---

## Működés

A működés négy fázisból áll:

**1. Szinkronizált metódus meghívása és sorosítása**: amikor a T1 kliens szál meghív egy szinkronizált metódust, a metódusnak először meg kell szereznie a monitor lock-ot. Ha T2 szál már tartja a lock-ot, T1 blokkolódik, amíg T2 fel nem oldja. Miután T1 megszerzi a lock-ot, a metódus fut.

**2. Szinkronizált metódus felfüggesztése**: ha a metódus nem tud haladni (pl. üres sorból olvasna), felfüggeszti magát egy monitor feltételen. Ez atomikusan feloldja a monitor lock-ot és felfüggeszti a szálat, lehetővé téve más szálak számára a monitor objektum elérését.

**3. Monitor feltétel értesítése**: egy másik szinkronizált metódus notifyAll()-lal értesíti a monitor feltételt. Ez felébreszti az összes várakozó szálat, amelyek versenyeznek a monitor lock-ért.

**4. Szinkronizált metódus folytatása**: az értesített metódus szál folytatja a futást ott, ahol felfüggesztette magát. A monitor lock-ot atomikusan visszaszerzi, mielőtt folytatja a végrehajtást.

---

## Implementáció lépései

**1. A monitor objektum interfész metódusainak definiálása**

Az interfész metódusok a Thread-Safe Interface pattern szerint csak zárolnak és implementációs metódusokra delegálnak. A szinkronizált metódusok névegyezménye: `put()` az interfész, `put_i()` az implementáció.

```cpp
class Message_Queue {
public:
    // Szinkronizált interfész metódusok
    void put(const Message &msg) {
        Guard<Thread_Mutex> guard(monitor_lock_);
        while (full_i())
            not_full_.wait(); // atomikusan feloldja a lock-ot és vár
        put_i(msg);
        not_empty_.notify();
    }

    Message get() {
        Guard<Thread_Mutex> guard(monitor_lock_);
        while (empty_i())
            not_empty_.wait(); // vár, amíg nem lesz elem
        Message msg = get_i();
        not_full_.notify();
        return msg;
    }
private:
    void put_i(const Message &msg); // implementáció, nem zárol
    Message get_i();
    bool empty_i() const;
    bool full_i() const;
};
```

**2. A monitor objektum belső állapotának és szinkronizációs mechanizmusainak definiálása**

A monitor lock-ot mutex-szel, a monitor feltételeket condition variable-ekkel implementálják. Mindkét monitor feltétel ugyanazt a monitor lock-ot osztja meg.

```cpp
private:
    mutable Thread_Mutex monitor_lock_;    // a monitor lock
    Thread_Condition not_empty_;           // signal: már nem üres
    Thread_Condition not_full_;            // signal: már nem tele
    size_t message_count_;
    size_t max_messages_;
```

<div class="callout trap" markdown="1">
**Csapda: nested monitor lockout**. Ha egy monitor objektum metódusa meghív egy másik monitor objektum szinkronizált metódusát, amely wait()-el vár, a külső monitor lock-ja fogva marad. Más szálak nem érhetik el a külső monitort, és holtpont keletkezik. Megoldás: a monitor feltételek osszák meg a lock-ot, ne legyen beágyazott monitor hívás.
</div>

---

## Ismert felhasználások

**Dijkstra és Hoare stílusú monitorok**: az eredeti monitor fogalom Dijkstrától és Hoare-tól származik. A Monitor Object pattern ezek objektumorientált megfelelője.

**Java objektumok**: minden Java objektum implicit monitor lock-ot tartalmaz. A `synchronized` kulcsszó szinkronizált metódust jelöl, a `wait()`, `notify()` és `notifyAll()` metódusok a monitor feltételek kezelésére szolgálnak. Ez a Java legalacsonyabb szintű szinkronizációs primitívje.

**Java java.util.concurrent**: a `LinkedBlockingQueue`, `ArrayBlockingQueue` és más blocking collection-ök a Monitor Object pattern megvalósításai, ahol a `put()` és `take()` szinkronizált metódusok monitor feltételeken blokkolnak.

**POSIX condition variables**: a POSIX `pthread_cond_wait()` és `pthread_cond_signal()` a monitor feltételek megvalósítása, amelyet a Monitor Object pattern C++-ban közvetlenül felhasznál.

---

## Valós példa

Egy játékszerver üzenetsorát (message queue) több szál használja párhuzamosan: supplier szálak beraknak üzeneteket, consumer szálak kivesznek. Ha a sor tele van, a supplier várjon. Ha üres, a consumer várjon.

```cpp
class GameMessageQueue {
public:
    void put(const GameEvent &event) {
        Guard<Thread_Mutex> guard(lock_);
        while (full_i())
            not_full_.wait();    // várakozik, ha tele
        put_i(event);
        not_empty_.notify();     // értesíti a consumereket
    }

    GameEvent get() {
        Guard<Thread_Mutex> guard(lock_);
        while (empty_i())
            not_empty_.wait();   // várakozik, ha üres
        GameEvent e = get_i();
        not_full_.notify();      // értesíti a suppliereket
        return e;
    }
private:
    mutable Thread_Mutex lock_;
    Thread_Condition not_empty_;
    Thread_Condition not_full_;
    // belső sor reprezentáció
};
```

A supplier szálak (hálózati eseménykezelők) beraknak játékeseményeket, a consumer szálak (játéklogika) feldolgozzák. A Monitor Object garantálja, hogy soha nem kerül sor adatsérülésre, és a szálak nem vesztegetik az erőforrásokat busy waiting-gel.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Egyszerűsített konkurenciakezelés | Nehéz kiterjesztés (inheritance anomaly) |
| Egyszerű metódushívás ütemezés | Nested monitor lockout veszélye |
| Gyakran használt és jól ismert | Szinkronizáció és logika szoros csatolása |

**Előnyök részletesen:**

**Egyszerűsített konkurenciakezelés**: a klienseknek nem kell explicit szinkronizációs kódot írniuk. A monitor objektum transzparensen kezeli a szinkronizációt.

**Egyszerű metódushívás ütemezés**: a monitor feltételek kooperatív ütemezést tesznek lehetővé. A `wait()` és `notify()` mechanizmus egyszerű és jól ismert.

**Hátrányok részletesen:**

**Nehéz kiterjesztés**: az öröklődési anomália (inheritance anomaly) problémája miatt nehéz leszármaztatni egy monitor objektumból, ha az alosztály más szinkronizációs mechanizmust igényel. Az Aspect-Oriented Programming vagy a Strategized Locking és Thread-Safe Interface pattern együttes alkalmazása csökkenti ezt a problémát.

**Nested monitor lockout**: beágyazott monitor hívások holtponthoz vezethetnek, különösen Java-ban, ahol a monitor lock és az objektum szorosan csatolt. POSIX condition variable-ekkel ezt könnyebb elkerülni.

**Szinkronizáció és logika szoros csatolása**: a monitor objektum szinkronizációs és ütemezési logikája szorosan összefonódik a metódusok funkcionalitásával. Ez megnehezíti a szinkronizációs policy cseréjét anélkül, hogy a metódus-implementációkat módosítani kellene.

---

## Kapcsolata más patternekkel

Az **Active Object** pattern összehasonlítva a Monitor Object-tel: mindkettő szinkronizál és ütemez konkurens metódushívásokat, de az Active Object saját szálban hajtja végre a metódusokat, míg a Monitor Object a kliens szálában. Az Active Object kifinomultabb ütemezést tesz lehetővé, de drágább. A Monitor Object egyszerűbb és hatékonyabb, de kevésbé rugalmas.

A **Thread-Safe Interface** pattern segít a monitor objektum implementálásában: az interfész metódusok csak zárolnak és delegálnak, az implementációs metódusok nem zárolnak.

A **Strategized Locking** pattern segíthet a monitor lock típusának cserélhetővé tételében, csökkentve a szinkronizáció és funkcionalitás szoros csatolásának hátrányát.

A **Wrapper Facade** pattern rejti el a platform-specifikus mutex és condition variable API-kat a `Thread_Mutex` és `Thread_Condition` osztályokban.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Monitor Object szinkronizálja a megosztott objektumhoz való konkurens hozzáférést: egyszerre csak egy szinkronizált metódus futhat az objektumon belül. Négy szereplője van: Monitor Object, Synchronized Method, Monitor Lock és Monitor Condition. A monitor lock mutex-szel, a monitor feltételek condition variable-ekkel valósulnak meg. A wait() atomikusan feloldja a lock-ot és felfüggeszti a szálat, a notify() felébreszti a várakozó szálakat. Az Active Object-től eltérően a Monitor Object nem rendelkezik saját szállal, minden metódus a hívó kliens szálában fut. Előnye az egyszerűség és hatékonyság. Hátránya az öröklődési anomália és a nested monitor lockout veszélye.
</div>