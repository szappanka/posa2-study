---
layout: default
title: Proactor
---

# Proactor (Üzenetrögzítő)

## Mi ez a pattern

A Proactor egy eseményvezérelt alkalmazás, amely több eseménykérést fogad egyszerre, és <mark>aszinkron módon dolgozza fel őket</mark>. Lehetővé teszi, hogy az alkalmazás aszinkron műveletek befejeződésekor automatikusan értesüljön, és a befejezési esemény kezelőjét szinkron, soros módon futtassa.

Ez az Event Handling kategória második patternje, és a Reactor aszinkron párjaként értelmezhető. A legfontosabb különbség: a Reactor jelzőeseményekre vár (az I/O még nem kezdődött el), a Proactor I/O műveletek befejezésére vár.

---

## A probléma

Egy nagy terhelésű webszerver egyszerre számos klienst kiszolgál. Minden HTTP kérés négy lépésből áll: kapcsolat felépítése, kérés olvasása, fájl olvasása, válasz küldése. Ezek mindegyike I/O műveletet jelent, amely akár milliszekundumokig is tarthat.

A Reactor alapú szinkron megközelítésnél a szerver megvárja, amíg az I/O befejezódik, csak utána lép tovább. Ez egyetlen szálban fut, tehát amíg az egyik kérés I/O-ját várjuk, a többi kliens várakozik.

Három feszítő erő áll fenn:

- a skálázhatóság növelése és a késleltetés csökkentése érdekében az I/O műveleteknek <mark>nem szabad blokkolni a szerver fő szálát</mark>
- az áteresztőképesség maximalizálásához el kell kerülni a <mark>felesleges kontextusváltást és szinkronizációt</mark>
- új funkciók integrálása a meglévő completion event kezelési mechanizmus mellé <mark>ne igényeljen extra ráfordítást</mark>

---

## A megoldás

<mark>Osszuk fel az alkalmazás minden szolgáltatását két részre</mark>: egy hosszan tartó aszinkron műveletre és ezen aszinkron művelet befejeződésének kezelőjére. Az aszinkron művelet befejezésekor a rendszer generál egy completion event-et, amelyet berak egy sorba. A Proactor figyeli ezt a sort, és sorra végrehajtja a befejezési eseményekhez tartozó kezelőket.

Gondolj rá úgy, mint egy üzenetrögzítőre. Nem kell ott ülnöd a telefon mellett és várni. Elmész, csinálod a dolgaidat. Ha valaki hív, az üzenetrögzítő felveszi. Amikor visszaérsz, végighallgatod az üzeneteket sorban, és mindegyikre reagálsz. A szerver nem blokkolódik az I/O-ra, hanem akkor dolgozza fel az eredményt, amikor az már kész.

---

## Szereplők

A patternnek <mark>kilenc szereplője van</mark>:

**Handle**: az OS által biztosított azonosító, amely egy hálózati kapcsolatot vagy fájlt azonosít. <mark>Completion event-ek generálódnak rajta</mark>, amikor egy aszinkron művelet befejeződik.

**Asynchronous Operation (Aszinkron művelet)**: egy potenciálisan hosszan futó művelet, amely <mark>a hívó szálának blokkolása nélkül fut le</mark>. Például aszinkron fájlolvasás (`ReadFile()` Win32-n, `aio_read()` POSIX-on).

**Asynchronous Operation Processor (AOP)**: <mark>végrehajtja az aszinkron műveleteket és completion event-eket generál</mark>, amikor azok befejeződnek. Általában maga az operációs rendszer tölti be ezt a szerepet. Win32-n ez a Windows NT I/O alrendszer.

**Completion Event Queue**: egy sor, amelybe az AOP behelyezi a befejezett aszinkron műveletek eredményeit tartalmazó <mark>completion event-eket</mark>. Win32-n ez a Completion Port.

**Asynchronous Event Demultiplexer**: <mark>vár a Completion Event Queue-n</mark> és eltávolítja belőle a következő completion event-et. Win32-n ez a `GetQueuedCompletionStatus()` függvény, POSIX-on az `aio_suspend()`.

**Completion Handler**: absztrakt interfész, amely meghatározza a <mark>callback hook metódus szignatúráját</mark>, amelyet a Proactor hív meg, amikor egy completion event érkezik.

**Concrete Completion Handler**: a Completion Handler konkrét implementációja, amely az alkalmazásspecifikus logikát tartalmazza. <mark>Feldolgozza az aszinkron művelet eredményét</mark>, és szükség esetén új aszinkron műveleteket indít el.

**Initiator**: az az alkalmazáskomponens, amely <mark>meghívja az aszinkron műveletet az AOP-n keresztül</mark>, megadva a Handle-t és a Completion Handler-t. Általában maga is Concrete Completion Handler, mert ő dolgozza fel az eredményt is.

**Proactor**: a központi elem. Meghívja az Asynchronous Event Demultiplexer-t, kiveszi a completion event-eket a sorból, és <mark>az eseményhez rendelt Concrete Completion Handler hook metódusát hívja meg</mark>.

---

## Struktúra

<svg viewBox="0 0 720 400" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:720px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Initiator -->
  <rect x="20" y="20" width="120" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="80" y="38" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Initiator</text>
  <text x="80" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">aszinkron műveletet indít</text>

  <!-- AOP -->
  <rect x="220" y="20" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="300" y="38" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Async Operation</text>
  <text x="300" y="54" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Processor (OS)</text>

  <!-- Completion Event Queue -->
  <rect x="220" y="160" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="300" y="178" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Completion Event</text>
  <text x="300" y="194" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Queue</text>

  <!-- Async Event Demux -->
  <rect x="220" y="290" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="300" y="308" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Async Event</text>
  <text x="300" y="324" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Demultiplexer</text>

  <!-- Proactor -->
  <rect x="460" y="160" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="525" y="178" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Proactor</text>
  <text x="525" y="194" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">handle_events()</text>

  <!-- Completion Handler -->
  <rect x="460" y="290" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="525" y="308" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Completion Handler</text>
  <text x="525" y="324" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">handle_event()</text>

  <!-- Arrows -->
  <line x1="140" y1="42" x2="220" y2="42" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="180" y="36" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">indítja</text>

  <line x1="300" y1="64" x2="300" y2="160" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="322" y="116" font-size="9" fill="var(--color-text-secondary)">completion event</text>

  <line x1="300" y1="204" x2="300" y2="290" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="322" y="252" font-size="9" fill="var(--color-text-secondary)">dequeue</text>

  <line x1="380" y1="182" x2="460" y2="182" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="420" y="176" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">értesíti</text>

  <line x1="525" y1="204" x2="525" y2="290" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="548" y="252" font-size="9" fill="var(--color-text-secondary)">dispatch</text>
</svg>

---

## Működés

A működés öt lépésből áll:

1. Az Initiator aszinkron műveletet hív meg az AOP-n keresztül egy Handle-n, megadva a Completion Handler-t és a Completion Event Queue-t. Az AOP eltárolja ezeket a paramétereket.
2. <mark>Az Initiator és az aszinkron művelet ezután párhuzamosan futhatnak</mark>. Az Initiator közben más aszinkron műveleteket indíthat el.
3. Amikor az aszinkron művelet befejeződik, az AOP <mark>completion event-et generál</mark>, amely tartalmazza a művelet eredményét, és behelyezi a Completion Event Queue-ba.
4. Az alkalmazás meghívja a Proactor `handle_events()` metódusát. A Proactor az Asynchronous Event Demultiplexer-rel kiveszi a következő completion event-et a sorból.
5. A Proactor a completion event alapján <mark>azonosítja a Completion Handler-t, és meghívja annak hook metódusát</mark> az eredménnyel. A Completion Handler feldolgozza az eredményt, és szükség esetén új aszinkron műveleteket indít.

---

## Implementáció lépései

**1. Az alkalmazásszolgáltatások felbontása**

Minden szolgáltatást két részre kell bontani: az I/O műveletre (aszinkron) és az eredmény feldolgozására (Completion Handler). A Completion Handler-nek egyben Initiator-nak is kell lennie, hogy láncszerűen indíthasson újabb aszinkron műveleteket.

Például egy HTTP szerveren: az `HTTP_Acceptor` aszinkron `AcceptEx()` hívással fogadja a kapcsolatokat, majd a completion event-ben létrehozza az `HTTP_Handler`-t, amely aszinkron `ReadFile()` hívással olvassa a kérést.

```cpp
class HTTP_Handler : public Completion_Handler {
public:
    void handle_event(HANDLE h, Event_Type type, ACT &act) override {
        if (type == READ_EVENT) {
            process_http_request(act.buffer());
            // Új aszinkron művelet indítása: válasz küldése
            async_stream_.async_write(response_buffer, response_size);
        }
    }
    void start() {
        // Aszinkron olvasás indítása
        async_stream_.async_read(buffer_, MAXBUF);
    }
};
```

**2. Aszinkron Completion Handler-ek megvalósítása**

Az ACT (Asynchronous Completion Token) pattern alkalmazható arra, hogy a Completion Handler-t a completion event-hez rendeljük. Az ACT tartalmazza a Completion Handler referenciáját, így a Proactor O(1) időben meg tudja találni a megfelelő handlert a demultiplexáló tábla helyett.

```cpp
class Async_Stream {
public:
    void async_read(void *buf, u_long n_bytes) {
        // Win32-n: ReadFile() overlapped I/O módban
        // Az ACT tartalmazza a completion_handler_ referenciát
        ReadFile(handle_, buf, n_bytes, nullptr, &overlapped_);
    }
};
```

---

## Ismert felhasználások

**Windows Completion Ports**: <mark>a Win32 platform natívan támogatja a Proactor pattern-t</mark>. Az aszinkron műveletek (time-out, hálózati kapcsolat fogadása, fájl és socket olvasás/írás) eredményei Completion Port-okon kerülnek vissza az alkalmazáshoz a `GetQueuedCompletionStatus()` hívással.

**POSIX aszinkron I/O**: a `aio_read()`, `aio_write()` és `aio_suspend()` függvények a Proactor pattern UNIX megvalósítását képviselik. UNIX signal-ok segítségével preemptív aszinkron Proactor is megvalósítható.

**Operációs rendszerek eszközmeghajtói és megszakításkezelői**: minden hardver megszakítás (IRQ) kezelése lényegében Proactor-elvű: a hardver elvégzi a műveletet a háttérben, majd megszakítással jelzi a befejezést.

**ACE Proactor Framework**: az ACE keretrendszer hordozható Proactor implementációt nyújt, amely Win32 Completion Port-okat és POSIX AIO-t egyaránt tud használni a Wrapper Facade pattern mögött elrejtve.

**.NET aszinkron modell**: a .NET `async/await` mechanizmusa lényegében a Reactor, ACT és thread pool kombinációjára épül, ahol a `Task` completion event-ként, az `await` pedig Completion Handler-ként értelmezhető.

---

## Valós példa

Képzeld el ugyanazt a chat szervert, de most 10 000 egyidejű kapcsolattal. A Reactor egyetlen szálban fut, de minden üzenet küldése I/O műveletet jelent, amely akár milliszekundumokig tart. Ha 10 000 klienstől egyszerre érkezik üzenet, az egyszálú Reactor sorban várja meg mindegyik I/O befejezését.

Proactor-ral a szerver aszinkron `ReadFile()` hívásokat indít el minden kapcsolaton. Az operációs rendszer a háttérben kezeli az összes I/O-t. Amint bármelyik kész lesz, egy completion event kerül a sorba. A Proactor kiveszi és feldolgozza a következő kész eseményt.

```cpp
class ChatHandler : public Completion_Handler {
public:
    void start() {
        // Aszinkron olvasás indítása: nem blokkolja a szálat
        async_stream_.async_read(buffer_, MAXBUF);
    }

    void handle_event(HANDLE, Event_Type type, ACT &act) override {
        if (type == READ_EVENT) {
            broadcast_message(buffer_);
            // Azonnal új aszinkron olvasást indít a következő üzenetért
            async_stream_.async_read(buffer_, MAXBUF);
        }
    }
private:
    char buffer_[MAXBUF];
    Async_Stream async_stream_;
};
```

Az operációs rendszer 10 000 aszinkron olvasást kezel párhuzamosan. A szerver szála csak akkor dolgozik, amikor valamelyik ténylegesen kész. Nincs polling, nincs blokkolás, minimális szinkronizáció.

Ugyanez a minta működik fájlszerverek esetén is: a szerver nagy fájlok küldésekor aszinkron `WriteFile()` hívásokat indít, és csak a completion event-nél kapja vissza a vezérlést.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Teljesítmény | Korlátozott felhasználhatóság |
| Egyszerű szinkronizáció a válasz visszajuttatásában | Debuggolás nehézsége |
| Hordozhatóság | Csak akkor használjuk, ha muszáj |

**Előnyök részletesen:**

**Teljesítmény**: az aszinkron I/O lehetővé teszi, hogy <mark>a szerver szála soha ne blokkoljon</mark>. A processzoridő kizárólag tényleges feldolgozásra megy el, nem várakozásra. Nagy I/O terhelés esetén ez jelentős teljesítményelőnyt jelent a Reactor-hoz képest.

**Egyszerű szinkronizáció**: <mark>a Completion Handler-ek soros, egyszálú végrehajtásban futnak</mark>. Nincs szükség mutex-ekre vagy más szinkronizációs mechanizmusokra a completion event feldolgozása közben.

**Hordozhatóság**: a Wrapper Facade pattern elrejti a platform-specifikus aszinkron API különbségeket (`GetQueuedCompletionStatus` vs `aio_suspend`). Az alkalmazáskód platformfüggetlen marad.

**Hátrányok részletesen:**

**Korlátozott felhasználhatóság**: a Proactor csak akkor hatékony, ha az operációs rendszer <mark>natívan támogatja az aszinkron I/O-t</mark>. Ha nem, szimulálni kell szálakkal, ami elveszi a teljesítményelőnyt.

**Debuggolás nehézsége**: az invertált vezérlési folyam és az aszinkronitás kombinációja rendkívül megnehezíti a hibakeresést. Egy hiba előfordulhat az aszinkron művelet indításakor, de csak a completion event feldolgozásakor válik láthatóvá.

**Csak akkor használjuk, ha muszáj**: a Proactor-nak kilenc szereplője van, <mark>jóval bonyolultabb a Reactor-nál</mark>. Ha a szinkron megközelítés elegendő teljesítményt nyújt, a Reactor vagy a Half-Sync/Half-Async egyszerűbb alternatíva.

---

## Kapcsolata más patternekkel

A **Reactor** a szinkron párja a Proactor-nak. A Reactor jelzőeseményekre (indication events) vár, a Proactor completion event-ekre. Mindkettő demultiplexál és dispatch-el, de eltérő ponton a folyamatban.

Az **Asynchronous Completion Token (ACT)** pattern általában együtt használják a Proactor-ral: az ACT azonosítja a Completion Handler-t minden aszinkron művelethez, O(1) idejű demultiplexálást téve lehetővé a demultiplexáló tábla helyett.

A **Wrapper Facade** elrejti a platform-specifikus aszinkron I/O API-kat az aszinkron műveletek és az AOP implementációja mögött.

Az **Active Object** pattern hasonló abban, hogy elválasztja a metódus meghívását a végrehajtástól. A Proactor esetén az AOP végzi el ezt a szétválasztást az aszinkron műveletek háttérben való futtatásával.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Proactor eseményvezérelt alkalmazásokban aszinkron műveletek befejezésekor generált completion event-eket demultiplexál és továbbít a megfelelő Completion Handler-eknek. Kilenc szereplője van: Handle, Asynchronous Operation, Asynchronous Operation Processor, Completion Event Queue, Asynchronous Event Demultiplexer, Completion Handler, Concrete Completion Handler, Initiator és Proactor. Az Initiator aszinkron műveletet indít, az AOP háttérben futtatja, completion event-et generál a sorba, a Proactor kiveszi és meghívja a Completion Handler hook metódusát. Előnye a nagy I/O terhelés esetén nyújtott teljesítmény és az egyszerű szinkronizáció. Hátránya a bonyolultság és a korlátozott OS-támogatás.
</div>