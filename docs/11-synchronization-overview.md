---
layout: default
title: Synchronization
---

# 4. fejezet: Synchronization Patterns

## Miről szól ez a kategória?

Amikor több szál párhuzamosan fut és ugyanahhoz az adathoz nyúl, adatsérülés keletkezhet. Ez a race condition: két szál egyszerre olvassa és írja ugyanazt az értéket, és felülírják egymást.

A megoldás a mutex: az első szál lezárja a kritikus szakaszt, elvégzi a munkát, majd feloldja. A második addig vár. Ez a kategória négy patternből áll, amelyek mind azt a kérdést járják körül, hogy hogyan használj mutex-et helyesen, biztonságosan és hatékonyan.

---

## A négy kihívás, amit ez a kategória megold

| Kihívás | Pattern |
|---|---|
| Könnyű elfelejteni feloldani a zárat | Scoped Locking |
| Különböző helyzetekben különböző zár kell | Strategized Locking |
| Egymást hívó metódusok önholtpontot okoznak | Thread-Safe Interface |
| A Singleton inicializálása feleslegesen zárol | Double-Checked Locking |

---

## Az alapprobléma

```cpp
// Race condition: két szál egyszerre növeli a számlálót
Szál 1: olvassa az értéket → 5
Szál 2: olvassa az értéket → 5
Szál 1: írja vissza → 6
Szál 2: írja vissza → 6
// Eredmény: 6, pedig 7 kellett volna
```

A mutex megoldja ezt, de helytelen használata új problémákat okoz. Ezt járja körül a négy pattern.

---

## A négy pattern kapcsolata

A négy pattern nem független egymástól, hanem egymásra épül:

**Scoped Locking** az alap. A Guard osztály automatikusan feloldja a zárat a scope elhagyásakor, így nem lehet elfelejteni.

**Strategized Locking** a Scoped Locking-ra épül: a Guard osztályt általánosítja, hogy bármilyen lock típussal működjön. Ugyanaz a kód fut egyszálú tesztkörnyezetben (Null_Mutex, zéró overhead) és többszálú éles környezetben (Thread_Mutex).

**Thread-Safe Interface** a Scoped Locking egy konkrét problémáját oldja meg: ha egy zárolt metódus meghív egy másik zárolt metódust ugyanazon az osztályon, önholtpont keletkezik. A megoldás: csak a publikus metódusok zárolnak, a privát implementációs metódusok nem.

**Double-Checked Locking** a Singleton inicializálásának problémájára ad megoldást: az első ellenőrzés zárolás nélkül fut (gyors út), csak az inicializáláskor zárol egyszer, és a záron belül másodszor is ellenőriz a versenyhelyzet kizárásához.

```
Scoped Locking
    ↓ általánosítja
Strategized Locking

Scoped Locking
    ↓ önholtpont problémáját oldja meg
Thread-Safe Interface

Scoped Locking + Double-Checked Locking
    ↓ Singleton inicializáláshoz
Double-Checked Locking Optimization
```

---

## Miért fontos ez a vizsga szempontjából?

A Synchronization patternek az összes Concurrency pattern alapját adják. Az Active Object, Monitor Object és Half-Sync/Half-Async mind ezeket használja belül a szálak közti kommunikáció biztosításához.

<div class="callout tip" markdown="1">
**Tanulási tipp:** a négy pattern mind ugyanabból a problémából indul ki: a mutex helytelen vagy felesleges használatából. Ha ezt az alapproblémát érted, a négy pattern logikusan következik egymásból.
</div>

---

## A négy pattern egy mondatban

- **Scoped Locking**: a Guard osztály automatikusan feloldja a zárat a scope elhagyásakor, elfelejteni sem lehet
- **Strategized Locking**: a lock típusa kívülről adható meg, ugyanaz a kód fut Null_Mutex-szel és Thread_Mutex-szel
- **Thread-Safe Interface**: csak a publikus metódusok zárolnak, a privát metódusok nem, hogy ne legyen önholtpont egymást hívó metódusok között
- **Double-Checked Locking**: a Singleton inicializálása szálbiztos, de csak egyszer zárol, utána minden lekérdezés zárolás nélkül fut

---

## Tovább a patternekhez

1. [Scoped Locking](./12-scoped-locking.md)
2. [Strategized Locking](./13-strategized-locking.md)
3. [Thread-Safe Interface](./14-thread-safe-interface.md)
4. [Double-Checked Locking](./15-double-checked-locking.md)