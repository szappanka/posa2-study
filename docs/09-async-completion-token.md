---
layout: default
title: Asynchronous Completion Token
---

# Asynchronous Completion Token (ACT / Aszinkron Befejezési Token)

## Mi ez a pattern

Az Asynchronous Completion Token aszinkron műveletek eredményeinek visszajuttatása során alkalmazott <mark>azonosítási megoldás</mark>. Lehetővé teszi, hogy amikor egy aszinkron művelet befejeződik, az Initiator hatékonyan meg tudja találni azt a Completion Handler-t, amelynek a választ fel kell dolgoznia.

Az ACT <mark>szorosan kapcsolódik a Proactor patternhez</mark>, és általában azzal együtt alkalmazzák. A Proactor megmondja, hogyan kell aszinkron eseményeket kezelni, az ACT pedig megmondja, hogyan kell a befejezett eseményt visszavezetni a megfelelő kezelőhöz.

Más nevén: <mark>Active Demultiplexing vagy Magic Cookie</mark>.

---

## A probléma

Egy alkalmazás egyszerre számos aszinkron műveletet indít el. Amikor az egyik befejeződik és megérkezik a válasz, az alkalmazásnak tudnia kell, melyik Completion Handler-nek kell feldolgoznia.

A naiv megoldás: az alkalmazás fenntart egy nagy táblázatot, amely összeköti a műveleteket a Completion Handler-ekkel. Amikor válasz érkezik, végigkeresi a táblázatot. Ez két problémát okoz:

- a válasz visszajuttatási mechanizmus kezelése <mark>bonyolult és lassú lineáris kereséssel</mark>
- nagy rendszereknél (millió párhuzamos aszinkron művelet) a táblázatkeresés <mark>jelentős teljesítménybeli overhead-et okoz</mark>

Az elosztott rendszerekben ráadásul az is előfordulhat, hogy a kérés és a válasz más hálózati csomópontokon megy keresztül, ezért a keresési mechanizmusnak minimális kommunikációs overhead-del kell járnia.

---

## A megoldás

Minden aszinkron művelethíváshoz <mark>hozzunk létre egy ACT-t, amely azonosítja a Completion Handler-t</mark>. Az Initiator átadja az ACT-t a szolgáltatásnak a művelet indításakor. A szolgáltatás nem módosítja az ACT-t, csak tárolja. Amikor a művelet befejeződik, a szolgáltatás visszaküldi az ACT-t a válasszal együtt. Az Initiator az ACT alapján azonnal megtalálja a megfelelő Completion Handler-t.

Gondolj rá úgy, mint egy ruhatárra. Leadod a kabátodat, és kapsz egy számozott cetlit. A ruhatáros nem tudja, ki vagy, csak tárolja a kapcsolatot a szám és a kabát között. Amikor visszajössz és megmutatod a cetlit, azonnal kiadja a te kabátodat, nem kell végigkeresgélnie az összes kabátot. A cetli az ACT.

---

## Szereplők

A patternnek <mark>négy szereplője van</mark>:

**Service (Szolgáltatás)**: <mark>aszinkron módon hajtja végre a műveleteket</mark>. Átveszi az ACT-t az Initiator-tól, tárolja a művelet futása alatt, majd visszaküldi a válasszal együtt. Az ACT-t nem értelmezi, csak kezeli.

**Initiator (Kezdeményező)**: <mark>aszinkron műveletet hív meg a Service-en, és létrehozza az ACT-t</mark>, amely azonosítja a Completion Handler-t. Megkapja a választ az ACT-vel együtt, és az ACT alapján megtalálja és meghívja a megfelelő Completion Handler-t.

**Completion Handler**: <mark>feldolgozza az aszinkron művelet eredményét</mark>, amikor az Initiator meghívja. Általában maga az Initiator is egyben Completion Handler.

**ACT (Asynchronous Completion Token)**: egy azonosító, amelyet az <mark>Initiator hoz létre és ad át a Service-nek</mark>. Tartalmaz mindent, ami a válasz feldolgozásához szükséges: a Completion Handler referenciáját vagy az ahhoz vezető indexet. A Service számára teljesen átlátszó (opaque).

---

## Struktúra

<svg viewBox="0 0 680 300" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Initiator -->
  <rect x="20" y="110" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="90" y="133" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Initiator</text>
  <text x="90" y="150" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">ACT-t hoz létre</text>
  <text x="90" y="163" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">és demultiplexál</text>

  <!-- ACT -->
  <rect x="260" y="20" width="140" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="330" y="38" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">ACT</text>
  <text x="330" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">handler referencia / index</text>

  <!-- Service -->
  <rect x="260" y="200" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="330" y="223" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Service</text>
  <text x="330" y="240" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">tárolja az ACT-t</text>
  <text x="330" y="253" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">visszaküldi a válasszal</text>

  <!-- Completion Handler -->
  <rect x="500" y="110" width="150" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="575" y="128" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Completion Handler</text>
  <text x="575" y="144" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">feldolgozza az eredményt</text>

  <!-- Initiator -> ACT (creates) -->
  <line x1="160" y1="125" x2="260" y2="50" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="185" y="78" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">létrehozza</text>

  <!-- Initiator -> Service (invoke + ACT) -->
  <line x1="160" y1="155" x2="260" y2="220" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="185" y="205" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">indítja + ACT</text>

  <!-- Service -> Initiator (response + ACT) -->
  <line x1="260" y1="235" x2="160" y2="160" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="195" y="220" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">válasz + ACT</text>

  <!-- Initiator -> Completion Handler -->
  <line x1="160" y1="135" x2="500" y2="132" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="330" y="124" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">ACT alapján dispatch</text>
</svg>

---

## Működés

A működés négy lépésből áll:

1. Az Initiator létrehoz egy ACT-t, amely tartalmazza a Completion Handler referenciáját vagy indexét, majd meghívja a Service-t és átadja az ACT-t
2. A Service végrehajtja az aszinkron műveletet, <mark>tárolja az ACT-t, de nem módosítja</mark>. Az Initiator közben más munkát végezhet
3. Amikor a művelet befejeződik, a Service visszaküldi a választ az eredeti ACT-vel együtt
4. Az Initiator megkapja a választ és az ACT-t, az ACT alapján azonnal <mark>O(1) időben megtalálja a megfelelő Completion Handler-t</mark>, és meghívja azt

---

## Implementáció lépései

**1. Az ACT reprezentációjának meghatározása**

Az ACT háromféleképpen reprezentálható:

- <mark>Pointer</mark>: közvetlen memóriacím a Completion Handler-re. Leggyorsabb, de nem biztonságos, ha a memória újrafoglalásra kerülhet (pl. összeomlás utáni újraindítás esetén)
- <mark>Objektumreferencia</mark>: típusbiztos hivatkozás. Java-ban ez a legtermészetesebb megközelítés
- <mark>Index</mark>: egy tábla indexe, amelyből a Completion Handler megtalálható. Biztonságosabb, mint a pointer, mert az index érvényes marad akkor is, ha a memória átrendeződik

**2. Az ACT tárolásának meghatározása**

Két stratégia létezik:

- <mark>Implicit tárolás</mark>: az ACT bele van ágyazva a Service által kezelt adatstruktúrába (pl. Win32-n az OVERLAPPED struktúrába). A Service automatikusan visszaküldi
- <mark>Explicit tárolás</mark>: az Initiator maga tartja nyilván az ACT-ket egy táblázatban, és a Service csak egy azonosítót küld vissza

**3. Az ACT felhasználásának meghatározása**

- <mark>Sor alkalmazása</mark>: az ACT-k egy sorban várnak feldolgozásra, FIFO sorrendben
- <mark>Callback függvény</mark>: az ACT maga tartalmazza a callback függvény pointerét, amelyet a Service közvetlenül meg tud hívni

---

## Ismert felhasználások

**HTTP Cookie-k**: a legszemléletesebb valós példa az ACT pattern-re. A webszerver elküldi az űrlapot a böngészőnek, és csatol hozzá egy cookie-t. A cookie az ACT: azonosítja, hogy a felhasználó melyik kéréshez töltötte ki az űrlapot. <mark>A böngésző nem értelmezi a cookie-t, csak visszaküldi változatlanul</mark> a szerver válaszával együtt.

**Windows NT Overlapped I/O**: Win32-n az `OVERLAPPED` struktúra az ACT megvalósítása. Amikor aszinkron I/O műveletet indítasz, átadsz egy `OVERLAPPED` struktúrát. Amikor a művelet befejeződik, a `GetQueuedCompletionStatus()` visszaadja ugyanezt a struktúrát, amelyből a Proactor azonosítja a Completion Handler-t.

**POSIX aszinkron I/O**: a POSIX `aiocb` struktúra az ACT-t tartalmazza: az `aio_sigevent` mező meghatározza, hogyan értesítse a rendszer a Completion Handler-t, amikor a művelet befejeződik.

**CORBA TAO ORB**: a TAO CORBA implementáció az ACT pattern-t alkalmazza az aktív demultiplexáláshoz (active demultiplexing). Amikor a kliens kérést küld, a kérés sorszáma ACT-ként szolgál. Amikor a szerver válaszol, visszaküldi a sorszámot, amellyel a TAO O(1) időben megtalálja a várakozó kliens szálat.

**Jini**: a Jini elosztott eseményspecifikációjában a `handback` objektum az ACT megfelelője. Az eseményforrás visszaküldi a fogyasztónak, aki az alapján azonosítja, melyik regisztrációhoz tartozott az esemény.

---

## Valós példa

Képzeld el, hogy egy étteremben vagy. Rendeléskor a pincér ad egy sorszámot. Elmész az asztalodhoz, csinálod a dolgaidat. Amikor kész az étel, a pult feletti kijelzőn megjelenik a sorszámom. Te megnézed, felismered a saját számodat, és elmész a pultra. A pincér nem tudja, hol ülsz, nem keres végig minden asztalt: a sorszám elintéz mindent.

A sorszám az ACT. A pincér (Service) nem tudja, ki vagy, csak visszaküldi a számodat az étellel. Te (Initiator) az ACT alapján azonnal tudod, hogy a te rendelésed érkezett meg, és melyik barátodnak szól (Completion Handler).

Ugyanez kódban, egy egyszerű fájlletöltő alkalmazásban:

```java
// Az Initiator létrehozza az ACT-t minden letöltéshez
Map<Integer, DownloadHandler> handlers = new HashMap<>();

void startDownload(String url, DownloadHandler handler) {
    int token = generateUniqueToken(); // ez az ACT
    handlers.put(token, handler);      // eltárolja a kapcsolatot
    asyncService.download(url, token); // átadja az ACT-t a Service-nek
}

// Amikor a Service visszatér a válasszal és az ACT-vel
void onDownloadComplete(byte[] data, int token) {
    DownloadHandler handler = handlers.get(token); // O(1) keresés
    handler.handle(data);                           // meghívja a handlert
    handlers.remove(token);
}
```

Nincs lineáris keresés, nincs bonyolult táblázatkezelés. Az ACT azonnal megmutatja, melyik handler dolgozza fel az eredményt.

Ugyanez a minta jelenik meg:

**Pizzarendelésnél online**: a rendelési azonosítód az ACT. Amikor a futár csörög, megmutatod az azonosítót, ő azonnal tudja melyik rendelés az.

**Bankátutalásoknál**: a tranzakció azonosítója az ACT. Amikor a bank visszaigazol, a tranzakció ID alapján tudod, melyik átutalásra érkezett a visszajelzés.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Egyszerűbb adatstruktúra a kezdeményezőnél | Biztonsági kockázat |
| Alacsony tárigény | Memóriaszivárgás veszélye |
| Rugalmasság | |
| A kliensek működését nem befolyásolja a szerver | |

**Előnyök részletesen:**

**Egyszerűbb adatstruktúra**: az Initiator-nak <mark>nem kell komplex, kereshető táblázatot fenntartania</mark>. Az ACT maga hordozza a szükséges információt, így a demultiplexálás O(1) idejű.

**Alacsony tárigény**: az ACT csak minimális extra adatot jelent a művelet paraméterei mellett. Nagy rendszerekben, ahol millió aszinkron művelet fut párhuzamosan, ez fontos.

**Rugalmasság**: az ACT pointer, index vagy objektumreferencia formájában is megvalósítható. A Service számára mindig átlátszó marad, tehát a belső reprezentáció bármikor cserélhető.

**A kliensek működését nem befolyásolja a szerver**: a szerver <mark>nem tudja és nem is kell tudnia, mit jelent az ACT</mark>. A kliens oldal szabadon változtathatja az ACT belső szerkezetét anélkül, hogy a szervert módosítania kellene.

**Hátrányok részletesen:**

**Biztonsági kockázat**: ha az ACT egy közvetlen pointer, és a Service rosszindulatú vagy hibás, <mark>módosíthatja az ACT-t</mark>, ami a memória sérüléséhez vezet. Index alapú ACT-vel ez elkerülhető.

**Memóriaszivárgás veszélye**: ha egy aszinkron művelet soha nem fejeződik be (pl. hálózati hiba), az Initiator sosem kapja vissza az ACT-t, és a hozzá tartozó Completion Handler referencia a memóriában marad. Explicit timeout mechanizmus szükséges ennek kezelésére.

---

## Kapcsolata más patternekkel

A **Proactor** pattern az ACT leggyakoribb felhasználási kontextusa. A Proactor az ACT-t használja arra, hogy a completion event-eket O(1) időben demultiplexálja a megfelelő Completion Handler-ekhez.

A **Memento** pattern (GoF) hasonló szerkezetű: az Originator létrehoz egy Memento-t, amelyet a Caretaker tárol és visszaad. A különbség: a Memento objektum állapotát menti, az ACT pedig aszinkron műveletek befejezéséhez vezeti vissza a vezérlést.

A **Reactor** pattern is alkalmazhatja az ACT-t, bár általában demultiplexáló táblázatot használ. Az ACT gyorsabb alternatívát nyújt nagy rendszerekben.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

Az Asynchronous Completion Token aszinkron műveletek befejezésekor hatékonyan vezeti vissza a vezérlést a megfelelő Completion Handler-hez. Négy szereplője van: Service, Initiator, Completion Handler és maga az ACT. Az Initiator létrehozza az ACT-t és átadja a Service-nek a művelet indításakor. A Service nem módosítja, csak visszaküldi a válasszal együtt. Az Initiator az ACT alapján O(1) időben megtalálja a Completion Handler-t. Az ACT lehet pointer, objektumreferencia vagy index. Leghíresebb valós megvalósítása a HTTP cookie és a Win32 OVERLAPPED struktúra. Előnye az egyszerű, gyors demultiplexálás. Hátránya a biztonsági kockázat és a memóriaszivárgás veszélye.
</div>