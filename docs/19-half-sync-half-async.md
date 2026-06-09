---
layout: default
title: Half-Sync/Half-Async
---

# Half-Sync/Half-Async (Félig Szinkron / Félig Aszinkron / Busy Restaurant)

## Mi ez a pattern

A Half-Sync/Half-Async szétválasztja a szinkron és aszinkron hívásokat egy konkurens rendszerben, hogy egyszerűsítse a programozást teljesítménycsökkenés nélkül. Olyan rendszerekben alkalmazzuk, ahol a szinkron és aszinkron feladatok egyszerre vannak jelen és kapcsolatban állnak egymással.

Ez a Concurrency kategória harmadik patternje.

---

## A probléma

Teljesítményérzékeny alkalmazásokban, például webszerverekben és operációs rendszerekben, szinkron és aszinkron feldolgozás egyaránt szükséges.

Az aszinkron műveletek a hardverközeli műveletek hatékonyságának növelésére irányulnak: a hardver-megszakítások kezelése, hálózati csomagok fogadása és protokoll-feldolgozás aszinkron módon megy végbe a legnagyobb hatékonyság érdekében.

A szinkron műveletek egyszerűségük miatt preferáltak a magasabb szintű szolgáltatásoknál: egy FTP vagy TELNET szerver egyszerűbb és karbantarthatóbb, ha szinkron `read()` és `write()` rendszerhívásokat használ.

A probléma: a két réteg közvetlen kombinálása vagy feláldozza a hatékonyságot (ha mindent szinkronra kényszerítünk) vagy feláldozza az egyszerűséget (ha mindent aszinkronra kényszerítünk).

---

## A megoldás

Válasszuk szét a műveleteket szinkron és aszinkron halmazokba. A két művelettípus összekötésére definiáljunk egy várakozó sort (Queuing Layer), amelyen keresztül a két réteg kommunikál.

Gondolj rá úgy, mint egy foglalt étteremre. Az ajtónál álló személyzet (aszinkron réteg) folyamatosan fogadja az érkező vendégeket és beírja a várólistára. A pincérek (szinkron réteg) a saját tempójukban veszik le a rendeléseket és felszolgálnak. A várólistán keresztül kommunikálnak, de egymástól függetlenül dolgoznak.

---

## Szereplők

A patternnek négy szereplője van:

**Synchronous Service Layer (Szinkron szolgáltatási réteg)**: magasabb szintű, hosszan futó szolgáltatások. Blokkolva várakozhatnak a Queuing Layer-en adatokra. Saját szálban futnak, ami lehetővé teszi a blokkolást anélkül, hogy más szolgáltatásokat megakadályoznának. Például egy webszerver munkaszálai vagy alkalmazásfolyamatok.

**Asynchronous Service Layer (Aszinkron szolgáltatási réteg)**: alacsonyabb szintű, rövid futású szolgáltatások, amelyek nem blokkolhatnak hosszabb ideig. Külső eseményforrásokkal (hardver-megszakítások, hálózati interfészek) lépnek kapcsolatba. Adatokat és eseményeket fogadnak, majd a Queuing Layer-be helyezik. Például az operációs rendszer kernel interrupt handlerei vagy egy Reactor eseményciklusa.

**Queuing Layer (Sor réteg)**: a két réteget összekötő kommunikációs csatorna. Pufferolja az aszinkron rétegből érkező adatokat, és értesíti a szinkron réteget, ha adat áll rendelkezésre. A belső állapotát szinkronizálni kell, mert mindkét réteg hozzáfér. Általában Monitor Object pattern-nel implementálják.

**External Event Source (Külső eseményforrás)**: a rendszeren kívüli eseménygenerátorok, például hálózati interfész hardver, billentyűzet, órajel megszakítások. Ezek az aszinkron réteggel lépnek kapcsolatba.

---

## Struktúra

<svg viewBox="0 0 680 380" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- External Event Source -->
  <rect x="20" y="160" width="130" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="85" y="178" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">External Event</text>
  <text x="85" y="194" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Source</text>

  <!-- Async Layer -->
  <rect x="200" y="240" width="160" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="280" y="263" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Asynchronous</text>
  <text x="280" y="279" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Service Layer</text>
  <text x="280" y="293" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">interrupt handler / Reactor</text>

  <!-- Queuing Layer -->
  <rect x="200" y="130" width="160" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="280" y="153" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Queuing Layer</text>
  <text x="280" y="169" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">Monitor Object</text>
  <text x="280" y="183" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">szinkronizált sor</text>

  <!-- Sync Layer -->
  <rect x="200" y="20" width="160" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="280" y="43" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Synchronous</text>
  <text x="280" y="59" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Service Layer</text>
  <text x="280" y="73" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">szálkészlet / blokkoló I/O</text>

  <!-- External -> Async -->
  <line x1="150" y1="182" x2="200" y2="270" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="155" y="230" font-size="9" fill="var(--color-text-secondary)">esemény</text>

  <!-- Async -> Queue -->
  <line x1="280" y1="240" x2="280" y2="190" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="290" y="220" font-size="9" fill="var(--color-text-secondary)">enqueue</text>

  <!-- Queue -> Sync -->
  <line x1="280" y1="130" x2="280" y2="80" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="290" y="110" font-size="9" fill="var(--color-text-secondary)">dequeue</text>

  <!-- Labels -->
  <text x="430" y="55" font-size="10" fill="var(--color-text-secondary)">Blokkolhat</text>
  <text x="430" y="70" font-size="10" fill="var(--color-text-secondary)">Egyszerű programozás</text>
  <text x="430" y="165" font-size="10" fill="var(--color-text-secondary)">Puffer a két réteg közt</text>
  <text x="430" y="275" font-size="10" fill="var(--color-text-secondary)">Nem blokkolhat</text>
  <text x="430" y="290" font-size="10" fill="var(--color-text-secondary)">Alacsony szintű hatékonyság</text>
</svg>

---

## Működés

A működés három fázisból áll:

**Aszinkron fázis**: a külső eseményforrás eseményt generál, például egy hálózati csomag érkezik. Az aszinkron réteg kezeli az eseményt (pl. egy interrupt handler vagy egy Reactor event handler), gyorsan feldolgozza az alacsony szintű protokollt, és az adatot berakja a Queuing Layer-be.

**Sor fázis**: a Queuing Layer pufferolja az aszinkron rétegből érkező adatot és értesíti a szinkron réteget, hogy adat áll rendelkezésre.

**Szinkron fázis**: a szinkron réteg megfelelő szolgáltatása kiveszi az adatot a Queuing Layer-ből és feldolgozza. Blokkolhat anélkül, hogy más szolgáltatásokat megakadályozna, mert saját szálban fut.

---

## Implementáció lépései

**1. A rendszer felosztása három rétegre**

Magasabb szintű és hosszan futó szolgáltatásokat a szinkron rétegbe helyezzük. Ezek saját szálban futnak és blokkolhatnak. Alacsonyabb szintű és rövid futású szolgáltatásokat az aszinkron rétegbe helyezzük. Ezek nem blokkolhatnak hosszabb ideig.

**2. A Queuing Layer mechanizmusának definiálása**

A Queuing Layer általában Monitor Object pattern-nel implementált, szinkronizált sor. Gondoskodik arról, hogy az aszinkron réteg biztonságosan berakhasson és a szinkron réteg biztonságosan kivehessen adatot párhuzamosan.

```cpp
// Az aszinkron réteg (interrupt handler vagy Reactor event handler)
void handle_event(HANDLE h, Event_Type type) {
    Message msg = read_from_network(h);
    queuing_layer_.put(msg); // berak a szinkronizált sorba
}

// A szinkron réteg (munkaszál)
void worker_thread() {
    while (true) {
        Message msg = queuing_layer_.get(); // blokkolva kivesz
        process_application_logic(msg);     // magasabb szintű feldolgozás
    }
}
```

**3. A szinkron réteg szálainak száma**

A szinkron réteg szálainak száma a rendszer terhelésétől és az elérhető processzoroktól függ. Egy szálkészlet (thread pool) általában megfelelő. Az Active Object pattern thread pool variánsa közvetlenül alkalmazható a szinkron réteg implementálásához.

---

## Ismert felhasználások

**BSD UNIX hálózati alrendszer**: a legklasszikusabb megvalósítás. A kernel interrupt-vezérelt hálózati I/O aszinkron rétegként működik. A protokolfeldolgozás a kernel belsejében zajlik. Az alkalmazásfolyamatok (FTP, TELNET, HTTPD) szinkron `read()` és `write()` rendszerhívásokat használnak. A Socket réteg tölti be a Queuing Layer szerepét.

**CORBA ORB implementációk (MT-Orbix)**: az ORB Core-ban minden socket handle-hez külön szál tartozik az aszinkron rétegben, amely szinkron olvasással fogadja a CORBA kéréseket. A kéréseket a Queuing Layer-be rakja, majd az Active Object szálkészlet dolgozza fel a szinkron rétegben.

**ACE gateway**: az ACE framework Half-Sync/Half-Reactive variánsát alkalmazza. A Reactor az aszinkron réteget képviseli, az Active Object szálkészlet a szinkron réteget.

**Webszerverek**: az Apache és az NGINX is alkalmaz hasonló architektúrát, ahol egy eseményciklus fogadja a kapcsolatokat (aszinkron) és munkaszálak dolgozzák fel a kéréseket (szinkron).

---

## Valós példa

Egy online játék szervere két típusú munkát végez párhuzamosan: hálózati csomagok fogadása (alacsony szintű, gyors, nem blokkolhat) és játéklogika feldolgozása (magasabb szintű, bonyolult, blokkolhat).

Half-Sync/Half-Async nélkül vagy mindent aszinkronra kényszerítünk (a játéklogika is interrupt-szerűen fut, ami rendkívül bonyolult), vagy mindent szinkronra (a hálózati réteg elveszíti a hatékonyságát).

Half-Sync/Half-Async-kal:

```cpp
// Aszinkron réteg: Reactor event handler
// Gyorsan fut, nem blokkolhat
class NetworkHandler : public Event_Handler {
public:
    void handle_event(HANDLE h, Event_Type type) {
        GamePacket packet = read_packet(h);
        game_queue_.put(packet); // berak és visszatér
    }
private:
    MonitorQueue<GamePacket> &game_queue_;
};

// Szinkron réteg: munkaszálak
// Blokkolhat, egyszerűen programozható
class GameWorker {
public:
    void run() {
        while (true) {
            GamePacket p = game_queue_.get(); // blokkolva vár
            update_game_state(p);             // komplex logika
            send_updates_to_clients();
        }
    }
private:
    MonitorQueue<GamePacket> &game_queue_;
};
```

A hálózati réteg sosem blokkolódik a játéklogika miatt. A játéklogika egyszerűen, szinkron módon programozható.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Egyszerűsített programozás | Határátlépési overhead |
| Szétválasztott szinkronizációs policy-k | A magasabb szintű szolgáltatások nem feltétlenül profitálnak az aszinkron I/O hatékonyságából |
| Centralizált rétegek közti kommunikáció | Debuggolás és tesztelés összetettsége |

**Előnyök részletesen:**

**Egyszerűsített programozás**: a szinkron réteg programozói nem kell, hogy az aszinkron I/O bonyolultságával foglalkozzanak. A magasabb szintű üzleti logika szinkron, blokkoló stílusban írható.

**Szétválasztott szinkronizációs policy-k**: mindkét réteg saját szinkronizációs mechanizmust alkalmazhat. Az aszinkron réteg alacsony szintű mechanizmusokat (interrupt szintek) használhat, a szinkron réteg magasabb szintűeket (Monitor Object, szemaforok).

**Centralizált rétegek közti kommunikáció**: a Queuing Layer egyetlen pontja az összes rétegközti kommunikációnak. Ez megszünteti a lock-ok és sorosítás komplexitását, amelyek szükségesek lennének, ha a két réteg közvetlenül hozzáférne egymás memóriájához.

**Hátrányok részletesen:**

**Határátlépési overhead**: amikor adat kerül át az aszinkron és szinkron réteg között a Queuing Layer-en keresztül, kontextusváltás, szinkronizáció és adatmásolás overhead keletkezik. Az operációs rendszerek általában a user-level és kernel-level határán valósítják meg ezt, ahol a határátlépés jelentős teljesítményveszteséggel jár. Zero-copy technikával csökkenthető.

**A magasabb szintű szolgáltatások nem profitálnak az aszinkron I/O hatékonyságából**: a szinkron réteg elveszíti az aszinkron I/O hatékonyságát, mert az adatok másoláson mennek keresztül a Queuing Layer-ben.

**Debuggolás és tesztelés összetettsége**: a két réteg párhuzamos futása és a köztük lévő aszinkron kommunikáció hasonló kihívásokat okoz, mint a Reactor és Proactor pattern-eknél.

---

## Kapcsolata más patternekkel

A **Reactor** pattern az aszinkron réteget implementálja a Half-Sync/Half-Reactive variánsban. A Reactor eseményciklusa az aszinkron réteg, az Active Object szálkészlet a szinkron réteg.

Az **Active Object** pattern thread pool variánsa közvetlenül alkalmazható a szinkron réteg megvalósítására, ahol a munkaszálak a Queuing Layer-ből vesznek fel feladatokat.

A **Monitor Object** pattern implementálja a Queuing Layer-t, szinkronizált bounded buffer-ként.

A **Proactor** pattern az Half-Sync/Half-Async kiterjesztéseként tekinthető, ahol az aszinkron vezérlés és adatoperációk egészen a magasabb szintű szolgáltatásokig propagálnak.

A **Leader/Followers** pattern alternatívaként alkalmazható, ha nincs szükség Queuing Layer-re az aszinkron és szinkron rétegek között.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Half-Sync/Half-Async szétválasztja a szinkron és aszinkron szolgáltatásfeldolgozást egy konkurens rendszerben. Négy szereplője van: Asynchronous Service Layer (alacsony szintű, nem blokkolhat), Queuing Layer (szinkronizált puffer a két réteg közt), Synchronous Service Layer (magasabb szintű, blokkolhat) és External Event Source. Az aszinkron réteg fogadja az eseményeket és a Queuing Layer-be rakja, a szinkron réteg munkaszálai kivesznek és feldolgozzák. Előnye az egyszerűsített programozás és a szétválasztott szinkronizációs policy-k. Hátránya a határátlépési overhead és a debuggolás összetettsége.
</div>