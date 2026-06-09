---
layout: default
title: Thread-Safe Interface
---

# Thread-Safe Interface (Szálbiztos Interfész / Security Checkpoints)

## Mi ez a pattern

A Thread-Safe Interface minimalizálja a zárolási overhead-et és biztosítja, hogy a komponensen belüli metódushívások ne okozzanak önholtpontot (self-deadlock) azáltal, hogy megpróbálnak egy már tartott zárat újra megszerezni.

Ez a Synchronization kategória harmadik patternje. Szorosan kapcsolódik a Scoped Locking-hoz és a Strategized Locking-hoz: azok nyújtják a zárolási mechanizmust, a Thread-Safe Interface pedig megszervezi, hogy melyik metódus zárol és melyik nem.

---

## A probléma

Egy többszálú komponens publikus és privát metódusokat tartalmaz. A versenyhelyzetek megelőzéséhez a publikus metódusok zárolnak. Ha azonban egy metódus meghív egy másik metódust ugyanazon a komponensen belül, probléma keletkezik:

```cpp
template <class LOCK>
class File_Cache {
public:
    const void *lookup(const string &path) const {
        Guard<LOCK> guard(lock_); // zár megszerzve
        const void *fp = check_cache(path);
        if (fp == 0) {
            insert(path); // meghívja az insert-et, ami szintén zárolni akar
            fp = check_cache(path);
        }
        return fp;
    }

    void insert(const string &path) {
        Guard<LOCK> guard(lock_); // önholtpont: a zár már foglalt!
        // ...
    }
private:
    mutable LOCK lock_;
};
```

Ha nem rekurzív mutexet használunk, a `lookup()` meghívja az `insert()`-et, amely megpróbálja megszerezni a már `lookup()` által tartott zárat. A szál örökre várakozik önmagára: önholtpont.

Ha rekurzív mutexet használunk a megoldáshoz, az elkerüli az önholtpontot, de felesleges overhead-et okoz minden belső metódushívásnál.

---

## A megoldás

Minden komponens metódusát két kategóriába osztjuk:

**Interface metódusok (check)**: csak zárolnak és delegálnak. Megszerzik a zárat, meghívják a megfelelő implementációs metódust, majd feloldják a zárat. Soha nem hívnak más interfész metódust ugyanazon a komponensen.

**Implementációs metódusok (trust)**: csak a tényleges munkát végzik. Nem zárolnak és nem oldanak fel zárat. Megbíznak abban, hogy az interfész metódus már megszerezte a szükséges zárat.

---

## Szereplők

A patternnek két szereplője van:

**Interface Method (Interfész metódus)**: publikus metódus, amely a Scoped Locking idiommal zárol, majd az implementációs metódusra delegál. Ez az "ellenőrzőpont".

**Implementation Method (Implementációs metódus)**: privát vagy protected metódus, amely elvégzi a tényleges munkát. Nem zárol, nem old fel zárat, és nem hív interfész metódust.

---

## Struktúra

```
Külső hívás
    ↓
InterfaceMethod()          ← zárol + delegál
    ↓
    Guard guard(lock_);    ← zár megszerzése
    ↓
    implementation_i()     ← tényleges munka, nem zárol
    ↓
    ~Guard()               ← zár feloldása
    ↓
Visszatérés a hívóhoz
```

---

## Működés

A működés három lépésből áll:

1. A külső kliens meghívja az interfész metódust
2. Az interfész metódus megszerzi a zárat (Scoped Locking-gal), majd azonnal az implementációs metódusra delegál
3. Az implementációs metódus elvégzi a munkát és más implementációs metódusokat hívhat, de soha nem hív interfész metódust

```cpp
template <class LOCK>
class File_Cache {
public:
    // Interfész metódus: csak zárol és delegál
    const void *lookup(const string &path) const {
        Guard<LOCK> guard(lock_);
        return lookup_i(path); // delegál az implementációra
    }

    void insert(const string &path) {
        Guard<LOCK> guard(lock_);
        insert_i(path); // delegál az implementációra
    }

private:
    mutable LOCK lock_;

    // Implementációs metódusok: nem zárolnak
    const void *lookup_i(const string &path) const {
        const void *fp = check_cache(path);
        if (fp == 0) {
            insert_i(path); // belső hívás, nem zárol: nincs önholtpont
            fp = check_cache(path);
        }
        return fp;
    }

    void insert_i(const string &path) {
        // tényleges munka, zár nélkül
    }
};
```

---

## Implementáció lépései

**1. Az interfész és implementációs metódusok meghatározása**

Minden publikus metódushoz tartozik egy privát implementációs metódus. Az elnevezési konvenció: az implementációs metódus neve az interfész metódus neve `_i` utótaggal (pl. `lookup` és `lookup_i`).

**2. Az interfész metódusok programozása**

Az interfész metódusok teste mindössze két dologból áll: Guard objektum létrehozása és az implementációs metódus meghívása. Semmi mást nem tartalmaznak.

**3. Az implementációs metódusok programozása**

Az implementációs metódusok tartalmazzák az összes tényleges logikát. Hívhatnak más implementációs metódusokat, de soha nem hívhatnak interfész metódust, és soha nem zárolnak.

<div class="callout trap" markdown="1">
**Csapda:** a Thread-Safe Interface nem oldja meg teljes mértékben a deadlock problémát. Ha az A komponens interfész metódusa meghívja a B komponens implementációs metódusát, amely visszahív az A komponens interfész metódusára, deadlock keletkezik, mert A zárat az első hívás már tartja.
</div>

---

## Ismert felhasználások

**java.util.Hashtable**: a `put(Object key, Object value)` publikus metódus megszerzi a zárat, majd a belső rehash mechanizmust védő `rehash()` implementációs metódust hívja zár nélkül. Ha a `rehash()` is zárolna, felesleges overhead keletkezne.

**java.util.Collections SynchronizedMap**: a Thread-Safe Wrapper Facade variáns megvalósítása. Bármely `Map` implementációt szálbiztossá tesz úgy, hogy az interfész metódusok szinkronizálnak egy belső monitoron, majd delegálnak az eredeti objektumra.

**ACE framework**: az `ACE_Message_Queue` osztályban alkalmazzák. Az interfész metódusok (`enqueue()`, `dequeue()`) zárolnak, a belső implementációs metódusok nem.

**Dynix/PTX operációs rendszer**: a kernel egyes részeiben alkalmazzák.

**Biztonsági ellenőrzőpontok**: egy épületbe belépéskor az őr ellenőriz (interfész metódus: zárol). Miután belépett az ember, a belső területeken senki nem kérdezi újra (implementációs metódusok: megbíznak a beléptetésben).

---

## Valós példa

Egy játékszerver ranglistát tart fenn. A ranglista több helyen is frissíthető: egy játékos meccs végén pontot kap, egy másik helyen az inaktív játékosokat törlik, egy harmadik helyen a toplista lekérhető.

Naiv megközelítéssel mindhárom publikus metódus zárol és meghívja egymást is:

```cpp
void add_score(const string &player, int points) {
    Guard guard(lock_);
    remove_inactive(); // ez is zárolna: önholtpont
}
```

Thread-Safe Interface-szel:

```cpp
class Leaderboard {
public:
    // Interfész metódusok: csak zárolnak
    void add_score(const string &player, int points) {
        Guard<Thread_Mutex> guard(lock_);
        add_score_i(player, points);
    }

    void remove_inactive() {
        Guard<Thread_Mutex> guard(lock_);
        remove_inactive_i();
    }

    vector<string> get_top(int n) {
        Guard<Thread_Mutex> guard(lock_);
        return get_top_i(n);
    }

private:
    Thread_Mutex lock_;

    // Implementációs metódusok: nem zárolnak, hívhatják egymást
    void add_score_i(const string &player, int points) {
        scores_[player] += points;
        if (is_inactive(player))
            remove_inactive_i(); // nincs önholtpont
    }

    void remove_inactive_i() {
        // törli az inaktívakat
    }

    vector<string> get_top_i(int n) {
        // visszaadja a top n-et
    }
};
```

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Robusztusság | Deadlock fennállhat külső hívásláncban |
| Jobb teljesítmény | Overhead |
| Modularitás és egyszerűség | Plusz indirection |

**Előnyök részletesen:**

**Robusztusság**: biztosítja, hogy belső metódushívások ne okozzanak önholtpontot.

**Jobb teljesítmény**: a zárak nem szerzik meg és nem oldják fel feleslegesen a belső metódushívásoknál. Rekurzív mutex helyett egyszerű mutex is elég, ami gyorsabb.

**Modularitás és egyszerűség**: a zárolási logika és a tényleges funkcionalitás szét van választva. Mindkét rész egyszerűbbé válik: az interfész metódusok csak zárolnak, az implementációs metódusok csak dolgoznak.

**Hátrányok részletesen:**

**Deadlock fennállhat**: ha egy A komponens implementációs metódusa visszahív B komponensen keresztül A interfész metódusára, deadlock keletkezik. A Thread-Safe Interface ezt az esetet nem oldja meg.

**Overhead**: minden interfész metódushoz legalább egy implementációs metódus tartozik, ami növeli a komponens méretét és egy extra indirection szintet ad. Inlining-gal csökkenthető.

**Visszaélés lehetősége**: C++-ban egy objektum megkerülheti a publikus interfészt, ha ugyanannak az osztálynak egy másik objektumán közvetlenül hívja a privát metódust. Programozói fegyelemre van szükség.

---

## Kapcsolata más patternekkel

A **Scoped Locking** pattern az interfész metódusokban a zár automatikus kezelését biztosítja.

A **Strategized Locking** pattern lehetővé teszi, hogy a Lock típusa cserélhető legyen anélkül, hogy az interfész vagy implementációs metódusokat módosítani kellene.

A **Decorator pattern** (GoF) hasonló szándékú: extra felelősséget csatol egy objektumhoz. A különbség: a Decorator dinamikusan bővít, a Thread-Safe Interface statikusan osztja fel a metódus-felelősségeket.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Thread-Safe Interface minimalizálja a zárolási overhead-et és megakadályozza az önholtpontot belső metódushívásoknál. Két szereplője van: Interface Method (zárol és delegál) és Implementation Method (dolgozik, nem zárol). Az interfész metódusok csak a Scoped Locking Guard-ot hozzák létre és az implementációs metódusra delegálnak. Az implementációs metódusok soha nem zárolnak és soha nem hívnak interfész metódust. A Strategized Locking-gal kombinálva a lock típusa is cserélhető. Előnye a robusztusság és a jobb teljesítmény. Hátránya, hogy külső hívásláncban továbbra is keletkezhet deadlock.
</div>