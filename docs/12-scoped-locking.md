---
layout: default
title: Scoped Locking
---

# Scoped Locking (Hatókör-alapú Zárolás / Guard)

## Mi ez a pattern

A Scoped Locking C++ idiom biztosítja, hogy a zár bekapcsol, amikor a vezérlés belép a védendő blokkba, és feloldódik, amikor a vezérlés elhagyja a blokkot, függetlenül attól, hogy a visszatérés milyen módon történt meg.

Ez a Synchronization kategória első patternje. Más nevén: Guard, Synchronized Block.

---

## A probléma

Több szálat tartalmazó alkalmazásban a közös erőforrásokat védelmi zárakkal kell körülvenni. Ha a programozó manuálisan foglalja le és oldja fel a zárat, könnyen előfordulhat, hogy a feloldás elmarad.

C++-ban a vezérlés elhagyhatja a hatókört `return`, `break`, `continue`, `goto` utasítás, vagy elkapott kivétel miatt. Mindegyik esetben gondoskodni kell arról, hogy a zár feloldódjon.

```cpp
bool increment(const string &path) {
    lock_.acquire();

    Table_Entry *entry = lookup_or_create(path);
    if (entry == 0)
        return false; // a lock_.release() elfelejtve

    entry->increment_hit_count();
    return true; // a lock_.release() elfelejtve
}
```

Ha bármelyik visszatérési ágon elfelejtjük a `release()` hívást, a szerver lefagy, mert a többi szál soha nem tudja megszerezni a zárat.

---

## A megoldás

Létrehozunk egy Guard (Őr) osztályt, amelynek egy példányának élettartama (hatókör) jelöli a kritikus szakaszt. A konstruktor lefoglalja a zárat, a destruktor pedig feloldja azt. A C++ szemantikájának köszönhetően a destruktor akkor is lefut, ha egy kivétel miatt hagyjuk el a hatókört.

---

## Szereplők

A patternnek két szereplője van:

**Lock (Zár)**: a tényleges szinkronizációs objektum, például egy `Thread_Mutex`. Tartalmaz `acquire()` és `release()` metódusokat. Wrapper Facade rejti el a platform-specifikus mutex API-t.

**Guard (Őr)**: a Scoped Locking idiom megvalósítója. Konstruktora megszerzi a zárat, destruktora feloldja. Tartalmaz egy pointert a Lock-ra és egy boolean értéket (`owner_`), amely jelzi, hogy a zárolás sikeres volt.

---

## Struktúra

A Guard osztály és a Lock kapcsolata egyszerű: a Guard tárolja a Lock pointerét és automatikusan kezeli az életciklusát.

```
{                           ← hatókör eleje
    Guard guard(lock_);     ← konstruktor: lock_.acquire()
    
    // kritikus szakasz kódja
    // akármilyen visszatérési út ide kerülhet
    
}                           ← hatókör vége: destruktor: lock_.release()
```

---

## Működés

A működés négy lépésből áll:

1. A kritikus szakasz elején a kód létrehoz egy Guard objektumot a stacken, átadva a Lock referenciát
2. A Guard konstruktora meghívja a `lock_.acquire()` metódust és az `owner_` flaget `true`-ra állítja
3. A kritikus szakasz kódja lefut
4. Amikor a vezérlés elhagyja a hatókört (bármilyen úton: normál visszatérés, kivétel, `break`, `goto`), a C++ automatikusan meghívja a Guard destruktorát, amely feloldja a zárat

---

## Implementáció lépései

**1. A Guard osztály definiálása**

A Guard osztály konstruktora tárolja a Lock pointerét és megszerzi a zárat. A destruktor feloldja, ha az `owner_` flag igaz. A másolást és értékadást le kell tiltani, mert egy Guard objektum másolásakor a zár kétszer lenne feloldva.

```cpp
class Thread_Mutex_Guard {
public:
    Thread_Mutex_Guard(Thread_Mutex &lock)
        : lock_(&lock), owner_(false) {
        lock_->acquire();
        owner_ = true; // csak sikeres acquire után igaz
    }

    ~Thread_Mutex_Guard() {
        if (owner_) lock_->release();
    }

private:
    Thread_Mutex *lock_;
    bool owner_;

    // másolás és értékadás tiltva
    Thread_Mutex_Guard(const Thread_Mutex_Guard &);
    void operator=(const Thread_Mutex_Guard &);
};
```

**2. Explicit release() lehetőség hozzáadása**

Ha a kritikus szakaszon belül explicit módon is fel kell oldani a zárat, a Guard osztályhoz `acquire()` és `release()` accessor metódusokat kell adni. Az `owner_` flag megakadályozza a kétszeres feloldást.

```cpp
void release() {
    if (owner_) {
        owner_ = false;
        lock_->release();
    }
}
~Thread_Mutex_Guard() { release(); }
```

**3. A Guard használata a kritikus szakaszban**

A Guard objektumot a stacken kell létrehozni, a kritikus szakasz első utasításaként. Nem szabad heap-en létrehozni (new operátorral), mert akkor a destruktor nem hívódna meg automatikusan a hatókör elhagyásakor.

```cpp
bool increment(const string &path) {
    Thread_Mutex_Guard guard(lock_); // konstruktor: acquire()

    Table_Entry *entry = lookup_or_create(path);
    if (entry == 0)
        return false; // destruktor: release() — automatikusan
    
    entry->increment_hit_count();
    return true; // destruktor: release() — automatikusan
}
```

<div class="callout trap" markdown="1">
**Csapda:** ha egy Guard-ot használó metódus rekurzívan hívja önmagát, és a Lock nem rekurzív mutex, önholtpont (self-deadlock) keletkezik: a metódus megpróbálja megszerezni a már általa tartott zárat, és örökre vár. A Thread-Safe Interface pattern oldja meg ezt a problémát.
</div>

---

## Ismert felhasználások

**Booch Components**: az első C++ osztálykönyvtárak egyike, amely alkalmazta a Scoped Locking idiomot többszálú programokhoz.

**ACE framework**: az `ACE_Guard` osztály a pattern referencia-implementációja, amelyet az egész ACE keretrendszerben használnak.

**Threads.h++ (Rogue Wave)**: az ACE Scoped Locking tervei alapján definiál Guard osztályokat.

**Java synchronized block**: a Java fordító a `synchronized` blokkhoz `monitorenter` és `monitorexit` bytecode utasításokat generál. Kivétel esetén a fordító automatikusan generál egy kivételkezelőt, amely biztosítja a zár feloldását.

---

## Valós példa

Egy webszerver számlálót tart fenn minden URL-hez, hogy hányszor töltötték le. Több szál párhuzamosan kezeli a kéréseket, és mindegyik módosíthatja ugyanazt a számlálót.

Scoped Locking nélkül:

```cpp
bool increment(const string &path) {
    lock_.acquire();
    Table_Entry *entry = lookup_or_create(path);
    if (entry == 0) {
        lock_.release(); // el lehet felejteni
        return false;
    }
    entry->increment_hit_count();
    lock_.release(); // el lehet felejteni
    return true;
}
```

Scoped Locking-gal:

```cpp
bool increment(const string &path) {
    Thread_Mutex_Guard guard(lock_); // acquire automatikusan

    Table_Entry *entry = lookup_or_create(path);
    if (entry == 0)
        return false; // release automatikusan
    
    entry->increment_hit_count();
    return true; // release automatikusan
}
```

Ugyanez a minta jelenik meg fájlrendszer-hozzáférésnél: egy konfigurációs fájlt egyszerre csak egy szál olvashat és módosíthat. A Guard garantálja, hogy a fájlzár minden esetben felszabadul, még akkor is, ha az olvasás közben kivétel keletkezik.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Robusztusság | Rekurzív használat deadlockra vezet |
| | Nyelvi korlátok |
| | Minden zárolástípushoz külön Guard osztály szükséges |

**Előnyök részletesen:**

**Robusztusság**: a zárak automatikusan megszerzésre és feloldásra kerülnek, amikor a vezérlés belép és elhagyja a kritikus szakaszt. Ez megszünteti a szinkronizációhoz és többszálúsághoz kapcsolódó programozási hibákat.

**Hátrányok részletesen:**

**Rekurzív használat deadlockra vezet**: ha egy metódus rekurzívan hívja önmagát, és nem rekurzív mutexet használ, önholtpont keletkezik. A Thread-Safe Interface pattern oldja meg.

**Nyelvi korlátok**: az idiom C++ destruktorszemantikán alapul, ezért nem integrálódik OS-specifikus rendszerhívásokkal. A zár nem oldódik fel automatikusan, ha egy szál az `exit()` vagy `longjmp()` hívással hagyja el a kritikus szakaszt, mert ezek nem hívják meg a C++ destruktorokat.

**Minden zárolástípushoz külön Guard osztály szükséges**: egy `Thread_Mutex_Guard` csak `Thread_Mutex`-szel működik. Ez kiküszöbölhető a Strategized Locking pattern alkalmazásával, amely template paraméterként fogadja a lock típusát.

---

## Kapcsolata más patternekkel

A **Wrapper Facade** pattern rejti el a platform-specifikus mutex API-t a Lock osztályban, amelyet a Guard használ.

A **Strategized Locking** pattern kiküszöböli a Scoped Locking hátrányát: a Guard osztályt template paraméterrel általánosítja, így bármilyen Lock típussal működik.

A **Thread-Safe Interface** pattern megoldja a rekurzív hívásból eredő önholtpont problémát: csak az interfész metódusok alkalmazzák a Scoped Locking idiomot, az implementációs metódusok nem.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Scoped Locking biztosítja, hogy egy zár automatikusan megszerzésre és feloldásra kerüljön, amikor a vezérlés belép és elhagyja a kritikus szakaszt, függetlenül a kilépés módjától. Két szereplője van: Lock és Guard. A Guard konstruktora megszerzi a zárat, destruktora feloldja. Az owner_ flag megakadályozza a kétszeres feloldást. Hátránya, hogy rekurzív metódushívásoknál önholtpontot okozhat, amelyet a Thread-Safe Interface pattern old meg, és hogy minden Lock típushoz külön Guard kell, amelyet a Strategized Locking old meg.
</div>