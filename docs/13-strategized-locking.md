---
layout: default
title: Strategized Locking
---

# Strategized Locking (Paraméterezett Zárolás)

## Mi ez a pattern

A Strategized Locking paraméterezéssel különböző típusú szinkronizációs mechanizmusokat implementál egy rendszerben. Egy komponens szinkronizációs aspektusait úgy teszi cserélhetővé, hogy a Lock típust kívülről lehet megadni, a komponens kódját megváltoztatása nélkül.

Ez a Synchronization kategória második patternje, és szorosan kapcsolódik a Scoped Locking-hoz: a Strategized Locking általánosítja a Guard osztályt, hogy bármilyen Lock típussal működjön.

---

## A probléma

Egy többszálú alkalmazásban a komponenseknek védelmi zárakkal kell körülvenniük a kritikus szakaszokat. Különböző helyzetekben különböző zárak optimálisak:

- egyszálú alkalmazásban felesleges minden overhead, ott elég egy üres zár
- csak olvasó szálak esetén hatékonyabb a readers/writer lock, mint a mutex
- valós idejű rendszerekben speciális, prioritást kezelő mutex kell

Ha a lock típusát belekódolják a komponensbe, minden konfigurációhoz külön implementációt kell írni és karbantartani. Ez kódmásoláshoz és verzió-eltérésekhez vezet.

---

## A megoldás

Paraméterezzük a komponens szinkronizációs aspektusait úgy, hogy csatlakoztathatóvá tesszük. Minden Lock típus egy adott szinkronizációs stratégiát ír le. A komponens ezeket a csatlakoztatható típusokat példányosítja belül, és a metódusainak szinkronizálására használja.

Két megközelítés létezik:

- polimorfizmus: absztrakt `Lock` alaposztály, amelyből le lehet származtatni a konkrét zárakat. Futásidőben dől el a lock típusa.
- template paraméter: fordítási időben dől el a lock típusa, nulla futásidejű overhead-del.

---

## Szereplők

A patternnek három szereplője van:

**Lock (absztrakt zár)**: egységes interfész az összes szinkronizációs stratégiához. Minimálisan `acquire()` és `release()` metódusokat tartalmaz.

**Concrete Lock (konkrét zár)**: a Lock interfész egy adott implementációja. Például `Thread_Mutex` (valódi mutex), `RW_Lock` (olvasó/író zár), `Null_Mutex` (üres zár egyszálú esethez).

**Component (komponens)**: a Lock-ot template paraméterként vagy polimorf pointer formájában tartalmazza, és a Scoped Locking idiommal szinkronizálja a kritikus szakaszait.

---

## Struktúra

Polimorf megközelítés:

```cpp
class Lock {
public:
    virtual void acquire() = 0;
    virtual void release() = 0;
};

class Thread_Mutex_Lock : public Lock {
public:
    virtual void acquire() { lock_.acquire(); }
    virtual void release() { lock_.release(); }
private:
    Thread_Mutex lock_;
};

class Null_Mutex : public Lock {
public:
    void acquire() {} // üres, egyszálú esethez
    void release() {}
};
```

Template megközelítés:

```cpp
template <class LOCK>
class Guard {
public:
    Guard(LOCK &lock) : lock_(&lock), owner_(false) {
        lock_->acquire();
        owner_ = true;
    }
    ~Guard() {
        if (owner_) lock_->release();
    }
private:
    LOCK *lock_;
    bool owner_;
};
```

---

## Működés

A komponens nem tudja és nem is kell tudnia, milyen konkrét Lock-ot kap. A Scoped Locking Guard osztályt a Lock típussal paraméterezi, és minden kritikus szakasz elején létrehoz egy Guard objektumot:

```cpp
template <class LOCK>
class File_Cache {
public:
    const void *lookup(const string &path) const {
        Guard<LOCK> guard(lock_); // acquire automatikusan
        // kritikus szakasz
    }
private:
    mutable LOCK lock_;
};
```

Ugyanaz a kód különböző konfigurációkban:

```cpp
// Egyszálú, zéró overhead
typedef File_Cache<Null_Mutex> Content_Cache;

// Többszálú, mutex-szel
typedef File_Cache<Thread_Mutex> Content_Cache;

// Többszálú, readers/writer lock-kal
typedef File_Cache<RW_Lock> Content_Cache;
```

A komponens kódja egyik esetben sem változik.

---

## Implementáció lépései

**1. A komponens definiálása szinkronizáció nélkül**

Először írjuk meg a komponenst a szinkronizációs szempontok figyelembevétele nélkül. Ez a tiszta üzleti logika.

**2. A szinkronizációs mechanizmusok strategizálása**

Két altevékenység:

- definiáljuk az absztrakt Lock interfészt egységes `acquire()` és `release()` szignatúrával. Polimorfizmusnál virtuális metódusokkal, parameterized types esetén a Wrapper Facade pattern gondoskodik az egységes interfészről.
- definiáljuk a template Guard osztályt, amely a Lock-on dolgozik. Ez a Strategy pattern: a Guard a context, a konkrét Lock a stratégia.

**3. A komponens frissítése**

A Lock-ot konstruktor paraméterként (polimorf) vagy template paraméterként adjuk a komponensnek. A kritikus szakaszokat Guard-dal védjük a Scoped Locking idiom szerint.

**4. Deadlock és felesleges overhead elkerülése**

Ha a komponensen belül metódusok hívnak más metódusokat, önholtpont keletkezhet nem rekurzív mutex esetén. Ezt a Thread-Safe Interface pattern oldja meg.

**5. A Lock-stratégiák definiálása**

Általános Lock-stratégiák: rekurzív és nem rekurzív mutex, readers/writer lock, szemafor, fájl-zár, és a különösen hasznos `Null_Mutex` egyszálú esethez.

<div class="callout tip" markdown="1">
**Tipp:** a Null_Mutex meglepően hasznos. Minden metódusa üres inline függvény, amelyet az optimalizáló fordító teljesen kiszed a kódból. Így ugyanaz a komponens egyszálú módban zéró overhead-del fut, többszálú módban pedig valódi szinkronizációval.
</div>

---

## Ismert felhasználások

**ACE framework**: a Strategized Locking pattern-t az egész ACE keretrendszerben alkalmazzák. Az ACE container komponensek, mint az `ACE_Hash_Map_Manager`, szinkronizációs aspektusai template paraméterrel strategizálhatók.

**Booch Components**: az első C++ osztálykönyvtárak egyike, amely template paraméterekkel strategizálta a zárolást.

**Dynix/PTX operációs rendszer**: a kernel kódjában kiterjedten alkalmazza a Strategized Locking pattern-t.

**ATL Wizards (Microsoft)**: a Visual Studio ATL Wizard a parameterized type implementációt alkalmazza default template paraméterekkel. Single-threaded apartment esetén Null_Mutex-et, multi-threaded apartment esetén rekurzív mutexet használ.

---

## Valós példa

Egy játékszerver fájl-cache komponenst tart fenn a pályafájlokhoz. Fejlesztés közben egyszálú tesztkörnyezetben fut, élesben azonban több szál olvassa párhuzamosan a cache-t.

Strategized Locking nélkül két külön implementáció kellene: egy szinkronizáció nélküli egyszálú és egy mutex-es többszálú verzió. Ha hibát javítanak a cache-ben, mindkét helyen javítani kell.

Strategized Locking-gal egyetlen implementáció elegendő:

```cpp
template <class LOCK = RW_Lock>
class MapCache {
public:
    const MapData *lookup(const string &map_id) const {
        Guard<LOCK> guard(lock_);
        return cache_.find(map_id);
    }
    void insert(const string &map_id, const MapData *data) {
        Guard<LOCK> guard(lock_);
        cache_[map_id] = data;
    }
private:
    mutable LOCK lock_;
    map<string, const MapData *> cache_;
};

// Tesztkörnyezetben
typedef MapCache<Null_Mutex> TestCache;

// Éles szerveren
typedef MapCache<RW_Lock> ProductionCache;
```

A cache kódja egyszer van megírva. A readers/writer lock azért előnyös, mert sok szál olvashat egyszerre, de íráskor csak egy szál fér hozzá.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Rugalmasság és testreszabhatóság | Nem minden fordító támogatja a template-et |
| Kevés karbantartás | Over-engineering veszélye |
| Újrafelhasználhatóság | |

**Előnyök részletesen:**

**Rugalmasság és testreszabhatóság**: a komponens szinkronizációs stratégiája konfigurálható anélkül, hogy a kódot módosítani kellene. Ha új szinkronizációs stratégia kell, csak egy új Lock osztályt kell írni.

**Kevés karbantartás**: egyetlen implementáció van, nem több párhuzamos verzió. A hibajavítás és bővítés egy helyen történik, ami minimalizálja a verzióeltéréseket.

**Újrafelhasználhatóság**: a komponens kevésbé függ konkrét szinkronizációs mechanizmusoktól, ezért más projektekben is felhasználható különböző Lock-stratégiákkal.

**Hátrányok részletesen:**

**Nem minden fordító támogatja a template-et**: template megközelítésnél a lock stratégia láthatóvá válik az alkalmazáskódban, és nem minden fordító kezeli hatékonyan a template-eket. Ilyenkor a polimorf megközelítés a jobb választás.

**Over-engineering veszélye**: ha egy komponenshez csak egyetlen szinkronizációs stratégia szükséges, a Strategized Locking felesleges komplexitást ad hozzá. Csak akkor érdemes alkalmazni, ha a gyakorlat azt mutatja, hogy a komponens viselkedése valóban ortogonális a zárolási stratégiájára.

---

## Kapcsolata más patternekkel

A **Scoped Locking** pattern általában együtt használják a Strategized Locking-gal: a Guard osztályt template paraméterrel általánosítják, így bármilyen Lock típussal működik.

A **Thread-Safe Interface** pattern megoldja a Strategized Locking egyik kiegészítő problémáját: ha egy komponens metódusai más metódusokat hívnak, az önholtpont elkerüléséhez szétválasztja az interfész és implementációs metódusokat.

A **Wrapper Facade** pattern gondoskodik arról, hogy a különböző platform-specifikus mutex API-k egységes `acquire()` és `release()` interfészt kövessenek, amelyet a Strategized Locking megkövetel.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Strategized Locking egy komponens szinkronizációs mechanizmusát cserélhetővé teszi. Három szereplője van: absztrakt Lock interfész, Concrete Lock implementációk és a Component. Két megközelítés létezik: polimorfizmus (futásidőben dől el a lock típusa) és template paraméter (fordítási időben, zéró overhead). Különösen hasznos a Null_Mutex egyszálú esethez. Előnye a rugalmasság, az alacsony karbantartási igény és az újrafelhasználhatóság. Hátránya a template-et nem jól kezelő fordítókkal való inkompatibilitás és az over-engineering veszélye.
</div>