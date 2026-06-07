---
layout: default
title: Acceptor-Connector
---

# Acceptor-Connector (Manager-Titkár)

## Mi ez a pattern

Az Acceptor-Connector <mark>szétválasztja a kapcsolatépítés kezelését</mark> a kapcsolat után elvégzendő szolgáltatás végrehajtásától. A kapcsolatot felépítő és inicializáló logika nem keveredik össze az alkalmazásspecifikus feldolgozással.

Ez az Event Handling kategória negyedik patternje. Általában a Reactor vagy Proactor patternre épül: a Dispatcher szerepét ezek valamelyike tölti be.

---

## A probléma

Egy elosztott rendszerben a szolgáltatási csomópontoknak kapcsolatot kell létesíteniük egymással. Minden kapcsolat három fázisból áll: kapcsolatfelvétel, inicializálás, majd maga a szolgáltatás végrehajtása.

Ha ezt a három fázist összekeverjük, három probléma keletkezik:

- <mark>a szolgáltatási csomópontok terheltek lesznek a kapcsolatépítéstől</mark>, holott csak a tényleges adatfeldolgozással kellene foglalkozniuk
- <mark>nehéz bővíteni a rendszert</mark>, mert a kapcsolatépítési logika szétszórt az alkalmazáskódban
- <mark>a kapcsolatfelépítés késleltetése magas</mark>, különösen nagy hálózatokon, ahol szinkron módon egymás után kell kapcsolatot létesíteni sok csomóponttal

---

## A megoldás

<mark>Válasszuk szét a kapcsolatfelvételt a szolgáltatás végrehajtásától.</mark> Külön komponens felelős a kapcsolat létrehozásáért és inicializálásért, és csak ezután adja át a kapcsolatot a szolgáltatást végző komponensnek.

A megoldásban két szereplő áll szemben:

- <mark>az Acceptor passzív: vár egy bejövő kapcsolatkérésre</mark>
- <mark>a Connector aktív: ő kezdeményezi a kapcsolatot</mark> egy másik csomóponthoz

Mindkettő elvégzi a kapcsolat inicializálását, majd átadja a kapcsolatot a Service Handler-nek, amely ezután csak az alkalmazásspecifikus feldolgozással foglalkozik.

Gondolj rá úgy, mint egy manager-titkár kapcsolatra. Ha a vezérigazgató fel akar hívni egy másik vezérigazgatót, nem ő tárcsáz, hanem a titkárát kéri meg. A másik oldalon sem a vezérigazgató veszi fel a telefont, hanem az ő titkára. Miután a kapcsolat létrejött, a két titkár átadja a vonalat a két vezérigazgatónak, akik ettől fogva közvetlenül kommunikálnak. A titkárok az Acceptor és Connector, a vezérigazgatók a Service Handler-ek.

---

## Szereplők

A patternnek <mark>nyolc szereplője van</mark>:

**Transport Handle**: az OS-szintű hálózati kapcsolat végpontja. Egy <mark>passzív módú Transport Handle kapcsolatkéréseket fogad</mark> és új adatmódú Transport Handle-eket hoz létre. Az adatmódú Transport Handle-en keresztül lehet adatot küldeni és fogadni. Wrapper Facade rejti el a platform-specifikus socket API-t.

**Service Handler**: <mark>egy end-to-end szolgáltatás egyik felét valósítja meg</mark>. Tartalmaz egy Transport Handle-t, amelyen keresztül kommunikál a partner Service Handler-rel. Van egy `open()` aktiválási hook metódusa, amelyet az Acceptor vagy Connector hív meg inicializáláskor.

**Concrete Service Handler**: a Service Handler alkalmazásspecifikus implementációja. <mark>Az inicializálás után csak az adatfeldolgozással foglalkozik</mark>, a kapcsolatfelépítéssel nem.

**Acceptor**: <mark>passzív kapcsolatfelvételi factory</mark>. Inicializálja a passzív módú Transport Handle-t, regisztrálja a Dispatcher-nél, és amikor kapcsolatkérés érkezik, létrehozza és inicializálja a megfelelő Service Handler-t.

**Concrete Acceptor**: az Acceptor alkalmazásspecifikus példánya, konkrét Service Handler típussal paraméterezve.

**Connector**: <mark>aktív kapcsolatfelvételi factory</mark>. Kezdeményezi a kapcsolatot a távoli Acceptor felé szinkron vagy aszinkron módon, majd inicializálja a Service Handler-t.

**Concrete Connector**: a Connector alkalmazásspecifikus példánya.

**Dispatcher**: a központi elem, amely <mark>szétválasztja és továbbítja a kapcsolatfelvételi eseményeket</mark>. Általában a Reactor vagy Proactor tölti be ezt a szerepet.

---

## Struktúra

<svg viewBox="0 0 720 400" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:720px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Dispatcher -->
  <rect x="280" y="20" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="360" y="38" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Dispatcher</text>
  <text x="360" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">(Reactor / Proactor)</text>

  <!-- Acceptor -->
  <rect x="60" y="150" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="130" y="173" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Acceptor</text>
  <text x="130" y="189" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">passzív</text>
  <text x="130" y="203" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">accept()</text>

  <!-- Connector -->
  <rect x="520" y="150" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="590" y="173" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Connector</text>
  <text x="590" y="189" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">aktív</text>
  <text x="590" y="203" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">connect()</text>

  <!-- Service Handler A -->
  <rect x="60" y="310" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="130" y="333" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Service Handler</text>
  <text x="130" y="349" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">(szerver oldal)</text>
  <text x="130" y="363" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">open() / handle_event()</text>

  <!-- Service Handler B -->
  <rect x="520" y="310" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="590" y="333" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Service Handler</text>
  <text x="590" y="349" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">(kliens oldal)</text>
  <text x="590" y="363" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">open() / handle_event()</text>

  <!-- Transport Handle (connected) -->
  <rect x="280" y="310" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="360" y="328" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Transport Handle</text>
  <text x="360" y="344" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">kapcsolat adatcsatorna</text>

  <!-- Dispatcher -> Acceptor -->
  <line x1="280" y1="42" x2="180" y2="150" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="195" y="92" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">értesít</text>

  <!-- Dispatcher -> Connector -->
  <line x1="440" y1="42" x2="540" y2="150" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="520" y="92" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">értesít</text>

  <!-- Acceptor -> Service Handler A -->
  <line x1="130" y1="210" x2="130" y2="310" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="95" y="265" font-size="9" fill="var(--color-text-secondary)">létrehozza</text>
  <text x="95" y="278" font-size="9" fill="var(--color-text-secondary)">open() hív</text>

  <!-- Connector -> Service Handler B -->
  <line x1="590" y1="210" x2="590" y2="310" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="605" y="265" font-size="9" fill="var(--color-text-secondary)">létrehozza</text>
  <text x="605" y="278" font-size="9" fill="var(--color-text-secondary)">open() hív</text>

  <!-- Service Handlers <-> Transport Handle -->
  <line x1="200" y1="340" x2="280" y2="332" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <line x1="520" y1="332" x2="440" y2="332" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="360" y="302" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">adatcsere</text>
</svg>

---

## Működés

A működés három szcenárión keresztül érthető meg:

**1. szcenárió: passzív kapcsolatfelvétel (Acceptor oldal)**

1. Az alkalmazás meghívja az Acceptor inicializáló metódusát, amely létrehozza a passzív módú Transport Handle-t és regisztrálja a Dispatcher-nél
2. A Dispatcher figyeli a Transport Handle-t. Amikor kapcsolatkérés érkezik, értesíti az Acceptor-t
3. Az Acceptor elfogadja a kapcsolatot, létrehoz egy Service Handler-t, átadja neki a Transport Handle-t, és <mark>meghívja az `open()` hook metódusát</mark>
4. A Service Handler inicializálódik és regisztrál a Dispatcher-nél, majd ettől fogva önállóan kezeli az adatforgalmat

**2. szcenárió: szinkron aktív kapcsolatfelvétel (Connector oldal)**

1. Az alkalmazás meghívja a Connector `connect()` metódusát, megadva a cél transzport címét
2. <mark>A Connector blokkolva vár, amíg a kapcsolat létrejön</mark>
3. Kapcsolat után a Connector létrehozza a Service Handler-t, átadja a Transport Handle-t és meghívja az `open()` metódust
4. A Service Handler átveszi az irányítást

**3. szcenárió: aszinkron aktív kapcsolatfelvétel (Connector oldal)**

1. A Connector aszinkron módon kezdeményezi a kapcsolatot és <mark>azonnal visszatér, nem blokkolja a hívót</mark>
2. A Connector regisztrálja magát a Dispatcher-nél kapcsolat-befejezési eseményre
3. Amikor a kapcsolat létrejön, a Dispatcher értesíti a Connector-t
4. A Connector inicializálja a Service Handler-t

<div class="callout tip" markdown="1">
**Tipp:** az aszinkron kapcsolatfelvétel lehetővé teszi, hogy a Connector egyszerre sok kapcsolatot kezdeményezzen párhuzamosan, ami különösen hosszú késleltetésű (pl. műholdas) hálózatokon nagy teljesítményelőnyt jelent.
</div>

---

## Implementáció lépései

**1. A demultiplexáló/dispatching infrastruktúra réteg implementálása**

Meg kell választani a Transport mechanizmust (Socket API, TLI stb.) és a Dispatcher típusát. A Dispatcher általában a Reactor (szinkron esetben) vagy Proactor (aszinkron esetben). A Transport Handle-eket Wrapper Facade osztályok burkolják.

**2. A kapcsolatkezelő réteg implementálása**

Az Acceptor és Connector generikus osztályokként valósulnak meg, amelyek template paraméterként kapják a Service Handler típusát. Az Acceptor tartalmaz egy passzív módú Transport Handle-t, a Connector az aktív kapcsolatkezdeményezési logikát.

**3. Az alkalmazás réteg implementálása**

A Concrete Service Handler-ek a Service Handler absztrakt osztályt specializálják. Meghatározzák a kapcsolat utáni feldolgozási stratégiát: egyszerű esetben a Reactor-ral regisztrálják magukat, összetettebb esetben Active Object vagy Monitor Object patternt alkalmaznak.

---

## Ismert felhasználások

**UNIX Network Superservers (inetd)**: az `inetd` az összes hálózati szolgáltatás kapcsolatfelvételét kezeli. Amikor kapcsolat érkezik egy adott porton, létrehozza a megfelelő szolgáltatási folyamatot (Service Handler) és átadja neki a kapcsolatot.

**Webböngészők**: a böngészők HTML-oldalak betöltésekor párhuzamosan több aszinkron HTTP kapcsolatot kezdeményeznek (képek, scriptek, CSS). A Connector aszinkron változata teszi lehetővé, hogy a böngésző fő eseményciklusa ne blokkolódjon.

**CORBA ORB Core (TAO)**: a TAO CORBA implementáció az Acceptor-Connector pattern-t használja az ORB Core kapcsolatkezeléséhez, szétválasztva a kapcsolatfelépítési logikát a CORBA kérések feldolgozásától.

**ACE framework**: a `ACE_Acceptor`, `ACE_Connector` és `ACE_Svc_Handler` osztályok a pattern generikus C++ implementációját nyújtják.

---

## Valós példa

Képzeld el, hogy egy többjátékos online játék szerverét fejleszted. Amikor egy játékos csatlakozik, három dolog kell: kapcsolat felépítése, a játékos hitelesítése és inicializálása, majd az aktív játékmenet kezelése.

Ha mindezt összekevered, a játékmenet kezelő kódjában socket inicializáló kód is lesz, ami nehezen karbantartható és tesztelhetetlen.

Acceptor-Connector-ral:

```cpp
// Az Acceptor csak a kapcsolatfelvétellel foglalkozik
class GameAcceptor : public Acceptor<GameSession, SOCK_Acceptor> {
    // Amikor kapcsolat érkezik, az Acceptor létrehozza a GameSession-t
    // és meghívja az open() metódusát - ennyi a dolga
};

// A Service Handler csak a játékmenettel foglalkozik
class GameSession : public Service_Handler<SOCK_Stream> {
public:
    void open() override {
        // Inicializálás: hitelesítés, játékos betöltése
        authenticate_player();
        reactor_->register_handler(this, READ_EVENT);
    }

    void handle_event(HANDLE, Event_Type type) override {
        // Csak a játékmenettel foglalkozik
        GameCommand cmd = read_command();
        process_game_logic(cmd);
    }
};
```

A GameAcceptor nem tud arról, hogy a GameSession mit csinál. A GameSession nem tud arról, hogyan jött létre a kapcsolat.

Ha holnap SSL-t kell hozzáadni, csak az Acceptor-t cseréled, a játéklogika érintetlen marad. Ha a játéklogika megváltozik, az Acceptor változatlan marad.

Ugyanez a szétválasztás jelenik meg:

**Chat alkalmazásoknál**: az Acceptor fogadja a kapcsolatokat, a ChatHandler kezeli az üzeneteket. Új üzenetformátum esetén csak a ChatHandler változik.

**Fájlszerver esetén**: a Connector aszinkron módon kapcsolódik sok tartalomszolgáltatóhoz párhuzamosan. Ha mind a 100 kapcsolat szinkron lenne, egymás után kellene megvárni mindegyiket.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Hordozhatóság, újrafelhasználhatóság, kiterjeszthetőség | Többletköltség és komplexitás |
| Hatékonyság | Dedikált funkció |
| Robusztusság | Single point of failure |
| Könnyű implementáció | |

**Előnyök részletesen:**

**Hordozhatóság, újrafelhasználhatóság, kiterjeszthetőség**: <mark>a kapcsolatfelépítési és inicializálási logika egyszer van megírva</mark>, osztálykönyvtárba helyezve, és újrafelhasználható öröklődéssel. A Service Handler-ek alkalmazásonként változhatnak, a kapcsolatkezelő kód nem.

**Hatékonyság**: az aszinkron Connector lehetővé teszi, hogy <mark>sok kapcsolatot egyszerre kezdeményezzünk párhuzamosan</mark>, kihasználva a hálózat és a hoszt inherens párhuzamosságát.

**Robusztusság**: a Service Handler és az Acceptor szigorú szétválasztása megakadályozza, hogy egy passzív módú Transport Handle-t véletlenül adatolvasásra vagy írásra használjanak. Ez típusbiztonságot nyújt.

**Hátrányok részletesen:**

**Többletköltség és komplexitás**: a közvetlen socket API-hoz képest <mark>extra indirection és több osztály</mark> jelenik meg. Kis rendszereken ez felesleges overhead lehet.

**Dedikált funkció**: az Acceptor és Connector csak kapcsolatfelépítésre specializálódtak, más célra nem használhatók fel.

**Single point of failure**: ha a Dispatcher megáll, <mark>az egész kapcsolatfelvételi mechanizmus leáll</mark>. Redundanciát külön kell biztosítani.

---

## Kapcsolata más patternekkel

A **Reactor** és **Proactor** pattern-ek töltik be a Dispatcher szerepét. A Reactor szinkron kapcsolatfelvételnél, a Proactor aszinkron esetben.

A **Wrapper Facade** pattern elfedi a Transport Handle-ek platform-specifikus socket API-jait.

Az **Active Object** és **Monitor Object** pattern-ek alkalmazhatók a Concrete Service Handler-ekben a párhuzamos adatfeldolgozáshoz.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

Az Acceptor-Connector szétválasztja a kapcsolatfelépítést az alkalmazásspecifikus szolgáltatás végrehajtásától. Nyolc szereplője van: Transport Handle, Service Handler, Concrete Service Handler, Acceptor, Concrete Acceptor, Connector, Concrete Connector és Dispatcher. Az Acceptor passzív módon vár kapcsolatokra és inicializálja a Service Handler-t. A Connector aktívan kezdeményezi a kapcsolatot szinkron vagy aszinkron módon. Miután a kapcsolat létrejött, a Service Handler önállóan kezeli az adatforgalmat. Előnye a hordozhatóság, az aszinkron kapcsolatfelvételi hatékonyság és a robusztusság. Hátránya a többletköltség és a single point of failure kockázata.
</div>