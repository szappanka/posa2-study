---
layout: default
title: Double-Checked Locking Optimization
---

# Double-Checked Locking Optimization (Kétszeres Ellenőrzéses Zárolás / Lock Hint)

## Mi ez a pattern

A Double-Checked Locking Optimization csökkenti a szinkronizációs overhead-et olyan esetekben, ahol a kritikus szakasz csak egyszer fut le a program teljes futása alatt. Egy flag segítségével a legtöbb esetben elkerüli a felesleges zárolást.

Ez a Synchronization kategória negyedik és egyben utolsó patternje. Szorosan kapcsolódik a Scoped Locking-hoz és a Strategized Locking-hoz, amelyeket a zárolási logika megvalósítására használ.

---

## A probléma

A Singleton pattern biztosítja, hogy egy osztályból csak egyetlen példány létezzen. Az `instance()` metódus a következőképpen néz ki naiv implementációban:

```cpp
static Singleton *instance() {
    if (instance_ == 0)
        instance_ = new Singleton;
    return instance_;
}
```

Ez nem szálbiztos: ha két szál egyszerre hajtja végre, mindkettő láthatja, hogy `instance_ == 0`, és mindkettő létrehoz egy új példányt.

Az egyszerű megoldás Scoped Locking-gal:

```cpp
static Singleton *instance() {
    Guard<Thread_Mutex> guard(singleton_lock_);
    if (instance_ == 0)
        instance_ = new Singleton;
    return instance_;
}
```

Ez szálbiztos, de minden egyes `instance()` hívásnál megszerzi és feloldja a zárat, még akkor is, ha a Singleton már rég inicializálva van. A Singleton-t az alkalmazás milliószor is lekérdezheti, és mindannyiszor felesleges zárolás történik.

Az első zárolás nélküli optimalizálási kísérlet:

```cpp
static Singleton *instance() {
    if (instance_ == 0) {
        Guard<Thread_Mutex> guard(singleton_lock_);
        instance_ = new Singleton;
    }
    return instance_;
}
```

Ez sem helyes: két szál egyszerre láthatja, hogy `instance_ == 0`, az egyik megszerzi a zárat és inicializál, a másik várakozik. Amikor a második szál megszerzi a zárat, újra inicializálja a Singleton-t, mert a zár megszerzése után nem ellenőrzi újra a flag-et.

---

## A megoldás

Vezessük be a flag kétszeres ellenőrzését. Az első ellenőrzés zárolás nélkül fut le, és csak akkor zárolunk, ha a flag azt jelzi, hogy a kritikus szakasz még nem futott le. A záron belül másodszor is ellenőrizzük a flag-et, hogy kizárjuk azt az esetet, amikor több szál egyszerre jutott a zárig.

```
// Első ellenőrzés: zárolás nélkül
if (first_time_in_flag is FALSE) {
    // Zárolás
    acquire the mutex
    // Második ellenőrzés: a záron belül
    if (first_time_in_flag is FALSE) {
        execute the critical section
        set first_time_in_flag to TRUE
    }
    release the mutex
}
```

---

## Szereplők

A patternnek két szereplője van:

**First-time-in flag**: egy jelző, amely megmutatja, hogy a kritikus szakasz lefutott-e már. Singleton esetén maga az `instance_` pointer tölti be ezt a szerepet: ha nem nulla, az inicializálás már megtörtént.

**Critical section**: az a kód, amelynek pontosan egyszer kell lefutnia, például egy Singleton konstruktorának meghívása.

---

## Működés

A működés három lépésből áll:

1. Az első szál lefuttatja az első ellenőrzést: `instance_ == 0`, tehát belép a feltételbe, megszerzi a zárat, elvégzi a második ellenőrzést, inicializálja a Singleton-t, feloldja a zárat

2. Ha egy második szál az első szál mögé kerül a zárnál, megvárja a zár feloldását, majd megszerzi azt. A második ellenőrzésnél már `instance_ != 0`, tehát nem inicializál újra, feloldja a zárat és visszatér

3. Minden további szál az első ellenőrzésnél (`instance_ != 0`) azonnal visszatér, anélkül hogy zárolna

```cpp
class Singleton {
public:
    static Singleton *instance() {
        // Első ellenőrzés: zárolás nélkül, gyors út
        if (instance_ == 0) {
            Guard<Thread_Mutex> guard(singleton_lock_);
            // Második ellenőrzés: versenyhelyzet kizárása
            if (instance_ == 0)
                instance_ = new Singleton;
        }
        return instance_;
    }
private:
    static Singleton *instance_;
    static Thread_Mutex singleton_lock_;
};
```

---

## Implementáció lépései

**1. A kritikus szakasz azonosítása**

Azonosítani kell azt a kódot, amelynek pontosan egyszer kell lefutnia. Általában ez inicializálási logika, például egy Singleton konstruktora vagy egy konfigurációs fájl beolvasása.

**2. A zárolási logika implementálása**

A Scoped Locking idiommal implementáljuk a zárolást. A lock-ot a program indulása előtt inicializálni kell. C++-ban statikus objektumként definiálva garantált, hogy a fordító inicializálja az első használat előtt.

**3. A first-time-in flag implementálása**

A flag-nek atomikusan kell beállíthatónak lennie. Ha az `instance_` pointert használjuk flag-ként, az összes bitje egyszerre kell hogy legyen írható és olvasható. Problémás platformokon `volatile` kulcsszóval kell deklarálni, hogy a fordító ne cachezze regiszterbe.

```cpp
// volatile: a fordító nem optimalizálja el a második ellenőrzést
static Singleton *volatile instance_;
```

**Template adapter variáns**

A Double-Checked Locking Optimization beépíthető egy újrafelhasználható template adapter osztályba, amely bármely osztályt Singleton-ná alakít:

```cpp
template <class TYPE>
class Singleton {
public:
    static TYPE *instance() {
        if (instance_ == 0) {
            Guard<Thread_Mutex> guard(singleton_lock_);
            if (instance_ == 0)
                instance_ = new TYPE;
        }
        return instance_;
    }
private:
    static TYPE *instance_;
    static Thread_Mutex singleton_lock_;
};
```

---

## Ismert felhasználások

**ACE framework**: az egész ACE keretrendszerben kiterjedten alkalmazzák. Az `ACE_Singleton` template adapter automatikusan alkalmazza a Double-Checked Locking Optimization-t bármely osztályra.

**POSIX once variables**: a POSIX szabvány `pthread_once()` mechanizmusa garantálja, hogy egy függvény pontosan egyszer fusson le. A Linux `LinuxThreads` implementáció a Double-Checked Locking Optimization-t alkalmazza a `pthread_once()` belsejében.

**Sequent Dynix/PTX operációs rendszer**: az operációs rendszer kernelkódjában is alkalmazzák.

---

## Valós példa

Egy játékszerver konfigurációs rendszert tart fenn. A konfigurációt egyszer kell betölteni az induláskor, utána csak olvasni kell. Több száz szál kérdezheti le a konfigurációt másodpercenként.

Egyszerű zárolással minden lekérdezés zárolna, ami felesleges overhead-et okoz:

```cpp
static Config *instance() {
    Guard<Thread_Mutex> guard(lock_); // minden hívásnál zárol
    if (instance_ == 0)
        instance_ = new Config("server.cfg");
    return instance_;
}
```

Double-Checked Locking Optimization-nal csak az első inicializálásnál zárol:

```cpp
static Config *instance() {
    if (instance_ == 0) {             // gyors ellenőrzés, nem zárol
        Guard<Thread_Mutex> guard(lock_);
        if (instance_ == 0)           // versenyhelyzet kizárása
            instance_ = new Config("server.cfg");
    }
    return instance_;                 // zárolás nélkül tér vissza
}
```

A 100 szálból 99 az első ellenőrzésnél azonnal visszatér, a zárhoz csak az inicializálás pillanatában kerül sor.

Ugyanez a minta jelenik meg adatbázis-kapcsolat pooloknál: a pool egyszer jön létre, utána mindenki csak lekérdezi.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Overhead minimalizálása | Multi-processor cache coherency problémák |
| Versenyhelyzetek megelőzése | Nem atomikus pointer-írás egyes platformokon |

**Előnyök részletesen:**

**Overhead minimalizálása**: az első ellenőrzés biztosítja, hogy az inicializálás után egyetlen hívás sem zárol. A zárolás overhead-je csak az első inicializáláskor merül fel.

**Versenyhelyzetek megelőzése**: a második ellenőrzés garantálja, hogy a kritikus szakasz pontosan egyszer fut le, még akkor is, ha több szál egyszerre próbálta elérni a zárat.

**Hátrányok részletesen:**

**Multi-processor cache coherency**: egyes processzorok (COMPAQ Alpha, Intel Itanium) agresszív memória-cache optimalizálást végeznek, ahol az írási és olvasási műveletek sorrendje felcserélődhet. Ezeken a platformokon a Double-Checked Locking Optimization nem működik helyesen memory barrier utasítások nélkül.

**Nem atomikus pointer-írás egyes platformokon**: ha a pointer írása nem atomikus (pl. 32 bites pointer 16 bites buszon), egy másik szál részlegesen megírt, érvénytelen pointert olvashat. Ilyenkor külön, word-aligned integert kell flag-ként használni.

---

## Kapcsolata más patternekkel

A **Scoped Locking** pattern a zárolási logika implementálásához szükséges, a Double-Checked Locking Optimization a Guard osztályt használja a kritikus szakasz körüli automatikus zárkezeléshez.

A **Strategized Locking** pattern lehetővé teszi, hogy a lock típusa cserélhető legyen, és egyszálú esetben Null_Mutex-szel teljesen eltávolítható legyen a zárolás overhead-je.

A **Singleton pattern** (GoF) a leggyakoribb felhasználási kontextusa a Double-Checked Locking Optimization-nak.

A **Component Configurator** pattern dinamikusan konfigurálja az alkalmazást, ezért a Double-Checked Locking Optimization-nal nehéz kombinálni: a pre-inicializáció nem alkalmazható, ha a komponensek futás közben töltődnek be.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Double-Checked Locking Optimization csökkenti a szinkronizációs overhead-et olyan esetekben, ahol a kritikus szakasz csak egyszer fut le. Két szereplője van: a first-time-in flag és a kritikus szakasz. Az első ellenőrzés zárolás nélkül fut le és a legtöbb esetben azonnal visszatér. Csak ha a flag azt jelzi, hogy az inicializálás még nem történt meg, zárol, majd a záron belül másodszor is ellenőrzi a flag-et a versenyhelyzet kizárásához. Előnye az alacsony overhead az inicializálás után. Hátránya, hogy egyes multi-processor platformokon cache coherency problémák merülhetnek fel, amelyeket volatile deklarációval vagy memory barrier utasításokkal kell kezelni.
</div>