---
layout: default
title: Leader/Followers
---

# Leader/Followers (Vezető/Követők / Taxi Stands)

## Mi ez a pattern

A Leader/Followers hatékony konkurenciakezelési modellt definiál, ahol többszálú, megosztott eseményhalmazt kezelnek. Ide tartozik az eseménydetektálás, válogatás és végrehajtás.

Ez a Concurrency kategória negyedik patternje.

---

## A probléma

Nagyszámú, különböző típusú esemény kezelése többszálú környezetben. A cél egy hatékony modell definiálása, amely elkerüli a Half-Sync/Half-Async pattern hátrányait.

A Half-Sync/Half-Async Queuing Layer-e extra overhead-et jelent: kontextusváltás, szinkronizáció és adatmásolás. Ha nincs szükség sor rétegre a két réteg között, a Leader/Followers hatékonyabb alternatíva.

Négy feszítő erő áll fenn:

- az alkalmazásnak több eseményforrásból kell eseményeket fogadnia és feldolgoznia nagy teljesítménnyel
- a szálindítás és szálleállítás overhead-jét minimalizálni kell, ezért szálkészletet (thread pool) kell alkalmazni
- az eseményfeldolgozás párhuzamossága maximalizálandó
- a szinkronizációs mechanizmusok bonyolultságát minimalizálni kell

---

## A megoldás

Definiáljunk egy szálpoolt, ahol várakoznak a végrehajtó szálak. A pool tetején egy Leader thread vár egy esemény érkezésére, a többi szál (Followers) passzívan vár. Ha a Leader-hez érkezik egy esemény, kijelöli a pool-ból a következő Leader-t, és elkezdi végrehajtani az esemény funkcionalitását. Ha elvégezte a feladatát, visszakerül a pool-ba Follower-ként.

Gondolj rá úgy, mint egy taxi állomásra. Egy taxi (Leader) áll a sor elején és várja az utasokat. A többi taxi (Followers) mögötte vár. Ha megérkezik egy utas, az első taxi felveszi és elindul (feldolgozás). A mögötte álló taxi automatikusan előre lép és átveszi a Leader szerepet. Az első taxi, miután leadta az utast, visszaáll a sor végére.

---

## Szereplők

A patternnek öt szereplője van:

**Handle**: az OS-szintű azonosító, amely egy eseményforrást azonosít, például egy hálózati kapcsolatot. Egy esemény bekövetkezésekor a Handle jelzi, hogy I/O műveletet lehet rajta végrehajtani blokkolás nélkül.

**Handle Set**: Handle-ek gyűjteménye, amelyen a Leader thread egyszerre több eseményforrást figyelhet. Visszatér, ha bármelyik Handle-en esemény érkezett. UNIX-on `select()` vagy `poll()`, Win32-n `WaitForMultipleObjects()` implementálja.

**Event Handler**: absztrakt interfész, amely meghatározza a `handle_event()` hook metódus szignatúráját az alkalmazásspecifikus eseményfeldolgozáshoz.

**Concrete Event Handler**: az Event Handler konkrét implementációja, amely az alkalmazásspecifikus logikát tartalmazza.

**Thread Pool**: a szálak csoportja, amely szinkronizátort (szemafor vagy condition variable) oszt meg, és protokollt implementál a szálak Leader, Follower és Processing szerepek közti váltásához. Számon tartja, melyik szál az aktuális Leader.

---

## Struktúra

<svg viewBox="0 0 680 360" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Thread Pool -->
  <rect x="20" y="130" width="160" height="140" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="100" y="152" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Thread Pool</text>

  <!-- Leader -->
  <rect x="36" y="165" width="128" height="30" rx="6" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1"/>
  <text x="100" y="185" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Leader Thread</text>

  <!-- Followers -->
  <rect x="36" y="203" width="128" height="24" rx="6" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1"/>
  <text x="100" y="219" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">Follower Thread 1</text>
  <rect x="36" y="232" width="128" height="24" rx="6" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1"/>
  <text x="100" y="248" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">Follower Thread 2</text>

  <!-- Handle Set -->
  <rect x="250" y="50" width="150" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="325" y="73" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Handle Set</text>
  <text x="325" y="89" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">select() / poll()</text>

  <!-- Handle -->
  <rect x="250" y="170" width="150" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="325" y="188" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Handle</text>
  <text x="325" y="204" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">socket / file descriptor</text>

  <!-- Event Handler -->
  <rect x="470" y="50" width="170" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="555" y="68" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Event Handler</text>
  <text x="555" y="84" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">handle_event()</text>

  <!-- Concrete Event Handler -->
  <rect x="470" y="170" width="170" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="555" y="188" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Concrete Event</text>
  <text x="555" y="204" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Handler</text>

  <!-- Processing Thread -->
  <rect x="250" y="280" width="390" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="445" y="298" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Processing Thread</text>
  <text x="445" y="314" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">eseményt feldolgozza, majd visszaáll Follower szerepbe</text>

  <!-- Arrows -->
  <line x1="180" y1="180" x2="250" y2="80" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="192" y="125" font-size="9" fill="var(--color-text-secondary)">vár eseményre</text>

  <line x1="325" y1="110" x2="325" y2="170" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="335" y="145" font-size="9" fill="var(--color-text-secondary)">tartalmaz</text>

  <line x1="400" y1="80" x2="470" y2="80" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="435" y="74" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">dispatch</text>

  <line x1="555" y1="94" x2="555" y2="170" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="570" y="137" font-size="9" fill="var(--color-text-secondary)">implementálja</text>

  <line x1="100" y1="270" x2="250" y2="302" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="148" y="300" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">promote + process</text>
</svg>

---

## Működés

A működés négy fázisból áll:

**1. Leader thread demultiplexálás**: a Leader thread vár, amíg esemény érkezik bármely Handle-en a Handle Set-ben. Ha nincs aktuális Leader (mert az összes szál feldolgoz), az OS sorba állítja az eseményeket, amíg egy szál felszabadul.

**2. Follower thread előléptetése**: miután a Leader thread detektált egy eseményt, a Thread Pool `promote_new_leader()` metódusával kiválaszt egy Follower thread-et és Leader-ré lépteti elő. Ez azonnal megtörténik, még az esemény feldolgozása előtt, hogy az új Leader már várakozhasson a következő eseményre.

**3. Esemény feldolgozása**: az előző Leader thread Processing Thread szerepbe lép. Demultiplexálja az eseményt a megfelelő Concrete Event Handler-hez, és meghívja annak `handle_event()` metódusát. A Processing Thread párhuzamosan futhat az új Leader thread-del és más Processing Thread-ekkel.

**4. Visszatérés a pool-ba**: miután a Processing Thread befejezte az esemény feldolgozását, visszaáll Follower szerepbe és várakozik a Thread Pool szinkronizátorán. Ha nincs aktuális Leader, azonnal Leader lehet.

<div class="callout tip" markdown="1">
**Tipp:** a Handle deidaktiválás és reaktiválás protokollja kritikus. Amikor a Leader eseményt detektál, ideiglenesen deaktiválja a Handle-t a Handle Set-ből, mielőtt előlépteti az új Leader-t. Ez megakadályozza, hogy az új Leader ugyanazt az eseményt kétszer demultiplexálja. A Handle-t az esemény feldolgozása után reaktiválják.
</div>

---

## Implementáció lépései

**1. A Handle és Handle Set mechanizmus kiválasztása**

Két típusú Handle létezik: konkurens Handle (több szál hozzáférhet egyszerre, pl. UDP socket) és iteratív Handle (egyszerre csak egy szál használhatja, pl. TCP socket). Iteratív Handle esetén szükséges a Handle deaktiválás/reaktiválás protokoll.

**2. Handle deaktiválás és reaktiválás implementálása**

Amikor az esemény megérkezik, a Leader három lépést végez: deaktiválja a Handle-t, előlépteti az új Leader-t, majd feldolgozza az eseményt. A Handle-t az `LF_Event_Handler` Decorator osztály reaktiválja automatikusan a feldolgozás után.

```cpp
class LF_Event_Handler : public Event_Handler {
public:
    void handle_event(HANDLE h, Event_Type et) {
        // Deaktiválja a Handle-t, hogy az új Leader ne kapja meg újra
        thread_pool_->deactivate_handle(h, et);
        // Előlépteti az új Leader-t
        thread_pool_->promote_new_leader();
        // Feldolgozza az eseményt
        concrete_event_handler_->handle_event(h, et);
        // Reaktiválja a Handle-t
        thread_pool_->reactivate_handle(h, et);
    }
private:
    Event_Handler *concrete_event_handler_;
    LF_Thread_Pool *thread_pool_;
};
```

**3. A Thread Pool implementálása**

A Thread Pool tárolja az aktuális Leader thread azonosítóját és egy condition variable-t, amelyen a Follower thread-ek várnak.

```cpp
class LF_Thread_Pool {
public:
    void join(Time_Value *timeout = 0) {
        Guard<Thread_Mutex> guard(mutex_);
        for (;;) {
            while (leader_thread_ != NO_CURRENT_LEADER)
                followers_condition_.wait(timeout); // vár Leader-ré válásra

            leader_thread_ = Thread::self(); // átveszi a Leader szerepet
            guard.release();
            reactor_->handle_events(); // vár eseményre
            guard.acquire();
        }
    }

    void promote_new_leader() {
        Guard<Thread_Mutex> guard(mutex_);
        leader_thread_ = NO_CURRENT_LEADER;
        followers_condition_.notify(); // felébreszt egy Follower-t
    }
private:
    Reactor *reactor_;
    Thread_Id leader_thread_;
    Thread_Condition followers_condition_;
    Thread_Mutex mutex_;
};
```

---

## Ismert felhasználások

**CORBA ORB implementációk**: a TAO (The ACE ORB) a Leader/Followers pattern-t alkalmazza az ORB Core-ban az I/O események hatékony kezeléséhez. Az ORB szálkészlet szálai versenyeznek a Leader szerepért a közös Handle Set-en.

**Webszerverek**: számos webszerver, köztük az Apache Worker MPM variánsa, Leader/Followers-szerű architektúrát alkalmaz, ahol egy szálkészlet osztozik a beérkező kapcsolatokon.

**Transaction Monitor rendszerek (OLTP)**: nagyszámú párhuzamos tranzakció-kérés kezelésénél a Leader/Followers minimalizálja a szinkronizációs overhead-et, mivel nincs szükség külön Queuing Layer-re.

**Taxi állomások**: a taxisok sorban állnak, az első taxi (Leader) veszi fel a következő utast. Amint elmegy, a következő taxi előre lép. Pontosan a Leader/Followers dinamikája.

---

## Valós példa

Egy játékszerver hálózati rétege több száz klienst kezel párhuzamosan. Minden klienstől érkezhetnek játékesemények (mozgás, lövés, chat üzenet).

Half-Sync/Half-Async-kal: a hálózati szál berakja az eseményeket egy sorba, a munkaszálak kiolvassák. Ez extra overhead-et jelent minden eseménynél (adatmásolás, szinkronizáció).

Leader/Followers-szel nincs sor. A szálak maguk figyelik a Handle Set-et és közvetlenül dolgozzák fel az eseményeket:

```cpp
// Minden munkaszál részt vesz a pool-ban
void *worker_thread(void *arg) {
    LF_Thread_Pool *pool = static_cast<LF_Thread_Pool *>(arg);
    pool->join(); // alternál Leader és Follower szerepek közt
    return 0;
}

int main() {
    LF_Thread_Pool pool(Reactor::instance());
    // Munkaszálak indítása
    for (int i = 0; i < MAX_THREADS - 1; ++i)
        Thread_Manager::instance()->spawn(worker_thread, &pool);
    // A fő szál is csatlakozik
    pool.join();
}
```

Amikor 500 kliens küldi egyszerre az eseményeit, a szálkészlet szálai egymás után veszik át a Leader szerepet, detektálják és feldolgozzák az eseményeket anélkül, hogy Queuing Layer közbülső overhead-je lenne.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Egyszerűsített konkurenciakezelés | Rugalmatlanság |
| Performancia | Hálózati hiba esetén nehézkes |
| Egyszerű programozás | |

**Előnyök részletesen:**

**Egyszerűsített konkurenciakezelés**: a szálkészlet szálai nem igényelnek saját szinkronizációs mechanizmusokat. A Thread Pool kezeli az összes koordinációt.

**Performancia**: nincs Queuing Layer, így elmarad a Half-Sync/Half-Async határátlépési overhead-je. Nincs szükség adatmásolásra vagy extra kontextusváltásra az aszinkron és szinkron réteg között.

**Egyszerű programozás**: a Concrete Event Handler-ek ugyanúgy implementálhatók, mint a Reactor pattern-ben. A thread pool mechanizmus transzparens marad az alkalmazáskód számára.

**Hátrányok részletesen:**

**Rugalmatlanság**: a szálkészlet méretét előre meg kell határozni. Ha az események feldolgozása sokáig tart, az összes szál foglalttá válhat, és az új események sorban állnak az OS-ben.

**Hálózati hiba esetén nehézkes**: ha egy Handle hibát jelez, az aktuális Leader thread-nek kell kezelnie. Ha a hibakezelés hosszú ideig tart, blokkolja a Leader szerepet és lassítja az eseményfeldolgozást.

---

## Kapcsolata más patternekkel

A **Reactor** pattern általában a Handle Set demultiplexálásához használják a Leader/Followers implementációban. A Leader thread a Reactor `handle_events()` metódusát hívja, amely blokkolva vár egy eseményre.

A **Half-Sync/Half-Async** pattern alternatívája a Leader/Followers-nek. Ha nincs szükség Queuing Layer-re, a Leader/Followers hatékonyabb. Ha szükséges a szinkron és aszinkron rétegek explicit szétválasztása, a Half-Sync/Half-Async megfelelőbb.

A **Wrapper Facade** pattern elfedi a platform-specifikus Handle Set API-kat (select, WaitForMultipleObjects).

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Leader/Followers hatékony szálkészlet-alapú konkurenciakezelési modell, ahol a szálak felváltva töltik be a Leader, Processing és Follower szerepeket. Öt szereplője van: Handle, Handle Set, Event Handler, Concrete Event Handler és Thread Pool. Az egyetlen Leader thread vár eseményre a Handle Set-en, detektálás után előléptet egy Follower-t új Leader-ré, majd feldolgozza az eseményt. Előnye, hogy nincs Queuing Layer overhead és hatékonyabban skálázható mint a Half-Sync/Half-Async. Hátránya a rugalmatlanabb szálkészlet-méret és a hálózati hibakezelés összetettsége.
</div>