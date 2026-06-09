---
layout: default
title: Event Handling
---

# 3. fejezet: Event Handling Patterns

## Miről szól ez a kategória?

Minden szerver egyszerre több klienstől kap kéréseket, és minden kérés érkezése egy esemény. A kérdés: hogyan kezeld ezeket hatékonyan anélkül, hogy minden eseményre külön szálat indítasz, vagy sorban, blokkolva várakozol mindegyikre?

Ez a kategória négy patternből áll, amelyek az eseményvezérelt architektúra különböző aspektusait fedik le.

---

## A négy kihívás, amit ez a kategória megold

| Kihívás | Pattern |
|---|---|
| Több eseményforrást figyelj egyszerre, ne blokkold a főszálat | Reactor |
| Az I/O ne blokkoljon, csak az eredmény megérkezésekor dolgozz | Proactor |
| Párhuzamos aszinkron műveletek közt melyik handler dolgozza fel az eredményt | Async Completion Token |
| Kapcsolatfelépítés és adatfeldolgozás ne keveredjen össze | Acceptor-Connector |

---

## A kulcskülönbség: Reactor vs Proactor

Ez a két pattern a legfontosabb ebben a kategóriában, és sokan összekeverik őket.

**Reactor**: értesít, hogy valamit most el kell kezdened csinálni. A `select()` visszatér és azt mondja "az 57-es kész, most olvashatsz." Az olvasás még nem történt meg.

**Proactor**: értesít, hogy valami már kész van, csak fel kell dolgoznod. Az OS elvégezte az olvasást a háttérben, te csak az eredményt kapod meg.

Olyan mint a különbség aközött, hogy valaki szól neked hogy a pizza úton van (Reactor), vagy hogy a pizza már ott van az ajtód előtt (Proactor).

---

## A patternek kapcsolata

```
Reactor
    ↓ aszinkron párja
Proactor
    ↓ együtt használják
Async Completion Token

Reactor / Proactor
    ↓ Dispatcher szerepét tölti be
Acceptor-Connector
```

A **Reactor** és **Proactor** alternatívák: szinkron eseménykezelés vs aszinkron I/O.

Az **Async Completion Token** általában a Proactor-ral együtt jelenik meg: megoldja, hogy párhuzamos aszinkron műveletek közt melyik Completion Handler dolgozza fel az eredményt.

Az **Acceptor-Connector** a Reactor vagy Proactor-ra épül: a Dispatcher szerepét ezek valamelyike tölti be.

---

## Unity és AR/VR kontextusban

A Unity fő architektúrája Reactor-elvű: az `Update()`, az MRTK gesture recognition és az input események mind szinkron event cikluson alapulnak. A fő szál nem kérdezi folyamatosan hogy "történt valami?", hanem regisztrált handler-eket hív meg automatikusan.

A Proactor ott jelenik meg, ahol az OS vagy egy háttérszál végzi el a munkát: fájlbetöltés, hálózati kérés, szenzor adat aszinkron olvasása.

```csharp
// Reactor: Unity fő szál, szinkron event ciklus
void OnHandGestureDetected(HandGestureEventData data) { }

// Proactor: OS végzi a munkát, te csak az eredményt kapod
byte[] data = await File.ReadAllBytesAsync("map.bytes");
```

---

## Miért fontos ez a vizsga szempontjából?

A Reactor az egész Event Handling kategória alapja. Ha ezt érted, a Proactor csak egy lépéssel megy tovább (az I/O is aszinkron), az ACT egy konkrét részproblémát old meg, az Acceptor-Connector pedig a kapcsolatfelépítést választja szét.

<div class="callout tip" markdown="1">
**Tanulási tipp:** a Reactor és Proactor különbségét egy mondatban érdemes megjegyezni: a Reactor jelzőeseményekre vár (olvasni LEHET), a Proactor completion event-ekre vár (olvasás KÉSZ). Ez a különbség vezet végig az egész kategórián.
</div>

---

## A négy pattern egy mondatban

- **Reactor**: szinkron módon figyeli az összes eseményforrást egyszerre, és csak akkor hívja meg a handler-t, amikor valamelyiken esemény érkezett
- **Proactor**: aszinkron műveletek befejezésekor generált completion event-eket továbbítja a megfelelő Completion Handler-nek
- **Async Completion Token**: minden aszinkron művelethez egy token azonosítja a Completion Handler-t, így O(1) időben megtalálható melyik handler dolgozza fel az eredményt
- **Acceptor-Connector**: szétválasztja a kapcsolatfelépítést az adatfeldolgozástól, az Acceptor passzívan vár, a Connector aktívan kezdeményez, mindkettő átadja a kapcsolatot a Service Handler-nek

---

## Tovább a patternekhez

1. [Reactor](./07-reactor.md)
2. [Proactor](./08-proactor.md)
3. [Async Completion Token](./09-async-completion-token.md)
4. [Acceptor-Connector](./10-acceptor-connector.md)