---
layout: default
title: Reactor
---

# Reactor (Reaktor / Telefonközpont)

## Mi ez a pattern

A Reactor egy eseményvezérelt alkalmazás, amely több eseménykérést fogad egyszerre, és <mark>szinkron módon, sorosan dolgozza fel őket</mark>. A beérkező jelzőesemények szétosztását szétválasztja az alkalmazásspecifikus feldolgozástól.

Ez az <mark>Event Handling kategória első és legalapvetőbb patternje</mark>. A Proactor, az Acceptor-Connector és a Half-Sync/Half-Async mind épít rá.

---

## A probléma

Egy szerver egyszerre több klienstől kap kéréseket. Minden kérés érkezése egy jelzőeseménnyel (indication event) jár, például egy CONNECT esemény jelzi, hogy új kliens kapcsolódott, egy READ esemény jelzi, hogy adat érkezett.

Ha a szerver minden eseményre egyenként vár, miközben az egyiket feldolgozza, a többi kliens kérése várakozik. Ha viszont minden klienshez külön szálat indít, az rengeteg memóriát és szinkronizációs overhead-et jelent.

Négy erő feszül egymással:

- <mark>a szerver nem blokkolhat egyetlen eseményforrásra sem</mark>, mert az rontja a többi kliens kiszolgálásának késleltetését
- <mark>felesleges kontextusváltást, szinkronizációt és adatmozgatást minimalizálni</mark> kell az áteresztőképesség maximalizálásához
- új szolgáltatások integrálása a meglévő eseménykezelési mechanizmusba <mark>minimális erőfeszítéssel járjon</mark>
- az alkalmazáskód legyen <mark>védve a többszálúság és szinkronizáció bonyolultságától</mark>

---

## A megoldás

Szinkron módon várjunk jelzőesemények érkezésére egy vagy több eseményforrásból, és <mark>válasszuk szét az esemény-szétosztási mechanizmust az alkalmazásspecifikus feldolgozástól</mark>.

Minden szolgáltatáshoz vezessünk be egy külön Event Handler-t, amely az adott eseményforrásból érkező adott típusú eseményeket kezeli. Az Event Handler-ek regisztrálnak egy Reactor-nál, amely <mark>szinkron eseményszétválasztót (Synchronous Event Demultiplexer, SED) használ</mark> a jelzőeseményekre való várakozáshoz. Ha esemény érkezik, a SED értesíti a Reactor-t, amely szinkron módon elindítja az eseményhez rendelt Event Handler-t.

Gondolj rá úgy, mint egy telefonközpontra. A központ figyeli az összes vonalat egyszerre. Ha valamelyiken csörög a telefon, a központ azonnal átkapcsol a megfelelő kezelőhöz. A kezelő elvégzi a munkát, majd visszaáll a várakozásba. A többi vonal közben tovább figyelt.

---

## Szereplők

A patternnek <mark>öt szereplője van</mark>:

**Handle (Eseményleíró)**: az operációs rendszer által biztosított azonosító, amely egy eseményforrást azonosít, például egy hálózati kapcsolatot vagy fájlt. <mark>Amikor egy jelzőesemény bekövetkezik, a Handle "kész" állapotba kerül</mark>, jelezve hogy az adott forrásból olvasni lehet blokkolás nélkül. UNIX-on ezek fájlleírók (file descriptor), Win32-n HANDLE pointerek. *(az éttermes példában: az asztal száma)*

**Synchronous Event Demultiplexer (SED)**: az OS-szintű mechanizmus, amely <mark>egyszerre figyeli az összes regisztrált Handle-t</mark> és blokkolva vár, amíg legalább egy "kész" állapotba kerül. UNIX-on ez a `select()` vagy `poll()` függvény, Win32-n a `WaitForMultipleObjects()`. Visszatér, ha bármely Handle-n esemény érkezett, és megmondja melyiken. *(az éttermes példában: a szemed, amellyel egyszerre figyeled az összes asztalt)*

**Event Handler**: absztrakt interfész, amely <mark>meghatározza a callback hook metódusok szignatúráját</mark>, amelyeket a Reactor hív meg esemény bekövetkezésekor. Két metódusa van: `handle_event()` az esemény feldolgozásához és `get_handle()` a Handle lekéréséhez. *(az éttermes példában: a szabály, hogy egy adott asztalnál mit kell csinálni)*

**Concrete Event Handler**: <mark>az Event Handler konkrét implementációja</mark>, amely az alkalmazásspecifikus logikát tartalmazza. Regisztrálják a Reactor-nál, és a Reactor hívja meg őket automatikusan. Például egy logging szerveren a `Logging_Acceptor` az új kapcsolatokat fogadja, a `Logging_Handler` a beérkező adatokat olvassa. *(az éttermes példában: a konkrét szabály egy adott asztalhoz)*

**Reactor**: <mark>a központi elem</mark>. Nyilvántartja a regisztrált Event Handler-eket és azok Handle-jeit, meghívja a SED-et, és esemény bekövetkezésekor az eseményhez rendelt Event Handler `handle_event()` metódusát hívja meg. Általában Singleton-ként valósul meg. *(az éttermes példában: te magad, a pincér)*

## Struktúra

<svg viewBox="0 0 720 360" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:720px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Application -->
  <rect x="20" y="20" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="85" y="38" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Application</text>
  <text x="85" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">regisztrál</text>

  <!-- Reactor -->
  <rect x="220" y="20" width="160" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="300" y="40" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Reactor</text>
  <text x="300" y="56" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">register_handler()</text>
  <text x="300" y="70" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">handle_events()</text>

  <!-- SED -->
  <rect x="220" y="180" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="300" y="197" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Sync Event</text>
  <text x="300" y="213" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Demultiplexer</text>

  <!-- Event Handler -->
  <rect x="460" y="20" width="150" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="535" y="40" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Event Handler</text>
  <text x="535" y="56" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">handle_event()</text>
  <text x="535" y="70" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">get_handle()</text>

  <!-- Concrete Event Handler A -->
  <rect x="390" y="180" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="455" y="197" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Concrete Handler A</text>
  <text x="455" y="213" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">pl. Logging_Acceptor</text>

  <!-- Concrete Event Handler B -->
  <rect x="550" y="180" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="615" y="197" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Concrete Handler B</text>
  <text x="615" y="213" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">pl. Logging_Handler</text>

  <!-- Handle -->
  <rect x="460" y="290" width="150" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="535" y="308" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Handle</text>
  <text x="535" y="324" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">OS eseményleíró</text>

  <!-- Arrows -->
  <line x1="150" y1="42" x2="220" y2="42" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="300" y1="80" x2="300" y2="180" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="315" y="135" font-size="9" fill="var(--color-text-secondary)">select()</text>
  <line x1="380" y1="50" x2="460" y2="50" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="420" y="44" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">dispatch</text>
  <line x1="500" y1="80" x2="455" y2="180" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <line x1="570" y1="80" x2="615" y2="180" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <line x1="535" y1="80" x2="535" y2="290" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="550" y="190" font-size="9" fill="var(--color-text-secondary)">get_handle()</text>
</svg>

---

## Működés

A működés hat lépésből áll:

1. Az alkalmazás indulásakor a Concrete Event Handler-ek regisztrálnak a Reactor-nál a `register_handler()` hívással, megadva, hogy milyen típusú eseményekre kíváncsiak
2. A Reactor összegyűjti az eseményleírókat a `get_handle()` hívásokkal
3. Az alkalmazás elindítja a Reactor eseményciklusát a `handle_events()` hívással. A Reactor meghívja a SED-et az összegyűjtött Handle-listával
4. A SED visszatér, ha egy vagy több Handle "kész" állapotba kerül, azaz esemény érkezett
5. A Reactor azonosítja a Handle alapján az Event Handler-t, és meghívja annak `handle_event()` metódusát
6. Az Event Handler beolvassa az adatokat, elvégzi a kérés kiszolgálását, és visszatér. Ezután a Reactor visszaáll a várakozásba

<div class="callout tip" markdown="1">
**Tipp:** a Reactor megfordítja a vezérlés irányát. Nem az alkalmazás hívja a Reactor-t, hanem a Reactor hívja az alkalmazás Event Handler-eit. Ezt hívják Hollywood-elvnek: "Ne hívj minket, mi hívunk téged."
</div>

---

## Implementáció lépései

**1. Az Event Handler interfész definiálása**

Az Event Handler interfész meghatározza a `handle_event()` és `get_handle()` hook metódusokat. Ezeket a Reactor hívja meg. A `handle_event()` kap egy Event_Type paramétert (READ, WRITE, ACCEPT, CLOSE stb.), amely alapján a Concrete Event Handler eldönti, mit kell tenni.

```cpp
class Event_Handler {
public:
    virtual void handle_event(HANDLE, Event_Type) = 0;
    virtual HANDLE get_handle() const = 0;
    virtual ~Event_Handler() {}
};
```

**2. Az eseménykezelés dispatch stratégiájának meghatározása**

Meg kell dönteni, hogyan rendeli össze a Reactor a Handle-eket és az Event Handler-eket. Ez általában egy demultiplexáló táblával valósul meg, amely `<handle, event_handler, event_type>` hármasokat tárol. UNIX-on a Handle-ek egész számok, így a tábla tömbként implementálható O(1) keresési idővel.

**3. A Reactor interfész definiálása és implementálása**

A Reactor két fő felelőssége: regisztráció (`register_handler()`, `remove_handler()`) és az eseményciklus futtatása (`handle_events()`). A Reactor általában Bridge pattern alapján valósul meg: van egy absztrakt Reactor interfész és egy konkrét Reactor implementáció, amelyek szétválasztják az interfészt az implementációtól.

**4. A SED mechanizmus meghatározása**

A SED a platformtól függ. UNIX-on a `select()` a legportábilisabb, a `poll()` valamivel rugalmasabb, Win32-n a `WaitForMultipleObjects()` áll rendelkezésre. A Wrapper Facade pattern elfedi ezeket a különbségeket.

```cpp
// A Reactor belsejében, a handle_events() metódusban
fd_set read_fds, write_fds, except_fds;
demux_table.convert_to_fd_sets(read_fds, write_fds, except_fds);

int result = select(max_handle + 1, &read_fds, &write_fds, &except_fds, timeout);

for (HANDLE h = 0; h <= max_handle; ++h) {
    if (FD_ISSET(&read_fds, h))
        demux_table.table_[h].event_handler_->handle_event(h, READ_EVENT);
}
```

**5. A szükséges Reactor-ok számának meghatározása**

A legtöbb alkalmazás <mark>egyetlen Reactor Singleton-nel is megoldható</mark>. Ha azonban több eseménytér (például I/O és timer) külön kezelendő, több Reactor-t is lehet alkalmazni.

---

## Ismert felhasználások

**Windows eseménykezelési mechanizmus**: a Win32 üzenetciklusa (`GetMessage` / `DispatchMessage`) pontosan a Reactor működését valósítja meg. Az ablakeljárás (`WndProc`) az Event Handler, a `DispatchMessage` a Reactor.

**X Window System**: az X szerver eseménykezelése is Reactor alapú. A `XNextEvent()` a SED szerepét tölti be.

**CORBA ORB Core**: számos CORBA implementáció (TAO, ORBacus) a Reactor-t használja klienskérések szétválasztására és továbbítására.

**Call Management System**: a telekommunikációs hívásmenedzsment rendszerek a Reactor-t használják a sok egyidejű hívásból érkező jelzőesemények szétválasztására és a megfelelő kezelőhöz továbbítására.

**ACE framework**: a Reactor pattern referencia-implementációja az ACE (Adaptive Communication Environment) keretrendszerben található, ahol az `ACE_Reactor` osztály Singleton-ként érhető el.

---

## Valós példa

Képzeld el, hogy egy chat szervert fejlesztesz. Száz kliens csatlakozott, mindegyikük bármikor küldhet üzenetet. A szerver nem tudja, melyik kliens mikor ír.

Szálak nélküli naiv megközelítés: a szerver sorban végigpásztázza az összes klienst, és megkérdezi, hogy küldött-e valamit. Ez polling, rendkívül pazarló.

Reactor nélküli, szálakkal: minden klienshez egy szál. 100 kliensnél 100 szál, amelyek többsége csak vár. Ez is pazarló és szinkronizációs problémákhoz vezet.

Reactor-ral:

```cpp
// Minden klienskapcsolathoz egy Event Handler
class ChatHandler : public Event_Handler {
public:
    void handle_event(HANDLE h, Event_Type type) override {
        if (type == READ_EVENT) {
            string msg = read_message(h);
            broadcast_to_all_clients(msg);
        }
        if (type == CLOSE_EVENT) {
            reactor_->remove_handler(this);
            delete this;
        }
    }
    HANDLE get_handle() const override { return socket_handle_; }
private:
    HANDLE socket_handle_;
    Reactor* reactor_;
};
```

A `handle_events()` ciklus egyetlen szálban fut. A `select()` figyeli az összes 100 socket-et egyszerre. Ha az 57-es kliens üzenetet küld, a `select()` visszatér az 57-es Handle-lel, a Reactor meghívja a `ChatHandler` `handle_event()` metódusát, az feldolgozza az üzenetet, majd a ciklus visszaáll a várakozásba.

Egyetlen szál, nulla polling, nulla felesleges szinkronizáció.

Ugyanez a minta működik:

**Webszervereknél (JAWS)**: a `select()` figyeli az összes HTTP kapcsolatot. Új kapcsolat érkezésekor az `HTTP_Acceptor` handle_event()-je fut le, amely létrehozza az `HTTP_Handler`-t.

**GUI alkalmazásoknál**: a Java AWT `EventDispatchThread`-je pontosan a Reactor mintáját követi. A gombnyomás Handle-ként, az `ActionListener` Event Handler-ként viselkedik.

**Játék eseménykezelőknél**: a játékmotor fő ciklusa Reactor-szerűen figyeli az egér, billentyűzet és hálózat eseményeit, és az első beérkező eseményt adja át a megfelelő kezelőnek.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Modularitás, újrafelhasználhatóság és konfigurálhatóság | Szinkron várakozás |
| Szemantikus aspektusok szétválasztása | Debuggolás nehézsége |
| Hordozhatóság | Nem preemptív |
| Kontrolálható konkurenciakezelés | |
| Egyszerűség | |

**Előnyök részletesen:**

**Modularitás, újrafelhasználhatóság és konfigurálhatóság**: az esemény-szétosztási mechanizmus és az alkalmazásspecifikus feldolgozás <mark>szét van választva</mark>. A Reactor komponens újrafelhasználható, az Event Handler-ek egymástól függetlenül fejleszthetők.

**Szemantikus aspektusok szétválasztása**: a kapcsolatfelvétel és az adatfeldolgozás külön Concrete Event Handler-ekben valósul meg. Ez megkönnyíti a karbantartást.

**Hordozhatóság**: <mark>a Reactor interfész elrejti a platform-specifikus SED különbségeket</mark> (`select` vs `WaitForMultipleObjects`). Az alkalmazáskód nem függ az OS-től.

**Kontrolálható konkurenciakezelés**: az Event Handler-ek <mark>szinkron, soros feldolgozással futnak</mark>, így nincs szükség komplex szinkronizációra az egyszálú esetben.

**Egyszerűség**: az alkalmazásfejlesztő csak a Concrete Event Handler-t írja meg és regisztrálja. A demultiplexálás és dispatch mechanizmusa a Reactor-ban van elrejtve.

**Hátrányok részletesen:**

**Szinkron várakozás**: <mark>ha egy Concrete Event Handler sokáig fut, a Reactor nem tud más eseményt feldolgozni addig</mark>. Ez csökkenti a szerver válaszkészségét. Megoldás: az Event Handler-ek csak rövid, nem blokkoló munkát végezzenek.

**Debuggolás nehézsége**: az invertált vezérlési folyam (Hollywood-elv) megnehezíti a hibakeresést, mert a hívási lánc nem lineáris.

**Nem preemptív**: egyszálú alkalmazásban egy hosszan futó Event Handler megakadályozza a többi esemény feldolgozását.

---

## Kapcsolata más patternekkel

A **Proactor** pattern az aszinkron párja a Reactor-nak. A Reactor jelzőeseményekre vár (az I/O még nem kezdődött el), a Proactor I/O műveletek befejezésére vár.

Az **Acceptor-Connector** pattern a Reactor-ra épül: a kapcsolatfelvételt és az adatfeldolgozást külön Concrete Event Handler-ekbe szervezi.

A **Half-Sync/Half-Async** pattern a Reactor-t aszinkron réteggé teszi: a Reactor fogadja az eseményeket, majd egy szinkron szálkészletnek adja át feldolgozásra.

A **Wrapper Facade** pattern elrejti a platform-specifikus SED hívásokat (select, WaitForMultipleObjects) a Reactor implementációja elől.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Reactor eseményvezérelt alkalmazásokban szinkron módon vár jelzőesemények érkezésére, majd szétválasztja és továbbítja azokat a megfelelő Event Handler-eknek. Öt szereplője van: Handle (OS eseményleíró), Synchronous Event Demultiplexer (select/poll), Event Handler (absztrakt interfész), Concrete Event Handler (alkalmazásspecifikus logika) és Reactor (a központi elem). Az Event Handler-ek regisztrálnak a Reactor-nál, a Reactor a SED-del vár eseményre, majd a Handle alapján meghívja a megfelelő handler `handle_event()` metódusát. Előnye a modularitás, hordozhatóság és egyszerűség. Hátránya, hogy egy hosszan futó handler blokkolja a teljes eseményciklust.
</div>