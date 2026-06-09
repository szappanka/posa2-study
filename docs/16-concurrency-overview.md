---
layout: default
title: Concurrency
---

# 5. fejezet: Concurrency Patterns

## Miről szól ez a kategória?

Amikor több szál párhuzamosan fut, három alapkérdés merül fel: hogyan osztjuk el a munkát a szálak között, hogyan kommunikálnak egymással, és hogyan kerüljük el a versenyhelyzeteket.

A Synchronization kategória megmutatta hogyan védd a közös adatot. Ez a kategória egy szinttel feljebb lép: hogyan szervezd meg magát a párhuzamos végrehajtást.

---

## Az öt kihívás, amit ez a kategória megold

| Kihívás | Pattern |
|---|---|
| A metódus hívása ne blokkolja a hívót | Active Object |
| Egyszerre csak egy szál férjen hozzá az objektumhoz | Monitor Object |
| Gyors és lassú feldolgozás egyszerre legyen jelen | Half-Sync/Half-Async |
| Szálkészlet hatékonyan ossza el az eseményeket sor nélkül | Leader/Followers |
| Minden szálnak saját adatpéldánya legyen szinkronizáció nélkül | Thread-Specific Storage |

---

## Az öt pattern lényege

**Active Object**: a metódus hívása és végrehajtása szétválik. Te hívod a Proxy-t, kapsz egy Future-t, a háttérszálban a Servant elvégzi a munkát. Hat szereplője van: Proxy, Method Request, Activation List, Scheduler, Servant és Future.

**Monitor Object**: egyszerre csak egy szinkronizált metódus futhat az objektumon belül. A szálak `wait()`-el felfüggesztik magukat ha nem tudnak haladni, és `notify()`-jal ébresztik fel egymást. A Java `synchronized` kulcsszó pontosan ezt valósítja meg.

**Half-Sync/Half-Async**: két külön réteg van, köztük egy sorral. Az aszinkron réteg gyorsan fogadja az eseményeket és berakja a sorba, a szinkron réteg kiveszi és feldolgozza. Olyan mint a gyárban a szállítószalag és a szerelő között a tároló doboz.

**Leader/Followers**: szálkészlet, ahol egy Leader szál figyeli az eseményeket, a Followers szálak mögötte várnak. Ha esemény érkezik, a Leader feldolgozza, egy Follower előre lép és átveszi a Leader szerepet. Nincs külön sor, a szál megy az eseményhez.

**Thread-Specific Storage**: minden szálnak saját adatpéldánya van globális névvel. Nincs versenyhelyzet, nincs mutex, nincs overhead. Csak olyan adatokra alkalmazható, amelyeket a szálak nem osztanak meg egymással.

---

## A patternek kapcsolata

```
Active Object
    ↓ Activation List-et implementálja
Monitor Object
    ↓ Queuing Layer-t implementálja
Half-Sync/Half-Async
    ↓ alternatívája, sor nélkül
Leader/Followers

Thread-Specific Storage
    → független, megosztás helyett saját példány
```

**Active Object** és **Monitor Object** a legszorosabb pár: az Active Object Activation List-je Monitor Object-tel van implementálva.

**Half-Sync/Half-Async** és **Leader/Followers** alternatívák: ha kell a sor a két réteg között, Half-Sync/Half-Async, ha nem kell, Leader/Followers hatékonyabb.

**Thread-Specific Storage** független a többitől: nem szálak közti kommunikációt old meg, hanem azt kerüli el.

---

## Miért fontos ez a vizsga szempontjából?

Ez a legnehezebb kategória, mert a patternek összetettek és egymásra épülnek. Az Active Object hat szereplője visszariasztó lehet, de a lényeg egyszerű: hívás és végrehajtás szétválik. Ha ezt érted, a többi logikusan következik.

<div class="callout tip" markdown="1">
**Tanulási tipp:** az Active Object és Monitor Object párban érdemes megtanulni, mert az Active Object belül Monitor Object-et használ. A Half-Sync/Half-Async és Leader/Followers szintén párban érthetők meg, mert ugyanazt a problémát oldják meg különböző módon.
</div>

---

## Az öt pattern egy mondatban

- **Active Object**: a metódus hívása és végrehajtása két külön szálban történik, a hívó azonnal kap egy Future-t és nem blokkolódik
- **Monitor Object**: egyszerre csak egy metódus futhat az objektumon belül, a szálak wait és notify mechanizmussal koordinálnak
- **Half-Sync/Half-Async**: az aszinkron réteg gyorsan fogadja az eseményeket és sorba rakja, a szinkron réteg lassan dolgozza fel
- **Leader/Followers**: egy szálkészletben felváltva töltik be a Leader és Follower szerepet, sor nélkül, közvetlenül dolgozzák fel az eseményeket
- **Thread-Specific Storage**: minden szálnak saját adatpéldánya van, nincs megosztás, nincs szinkronizáció

---

## Tovább a patternekhez

1. [Active Object](./17-active-object.md)
2. [Monitor Object](./18-monitor-object.md)
3. [Half-Sync/Half-Async](./19-half-sync-half-async.md)
4. [Leader/Followers](./20-leader-followers.md)
5. [Thread-Specific Storage](./21-thread-specific-storage.md)