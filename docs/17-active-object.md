---
layout: default
title: Active Object
---

# Active Object (Aktív Objektum / Chef in a Restaurant)

## Mi ez a pattern

Az Active Object szétválasztja a metódus hívását a metódus végrehajtásától annak érdekében, hogy konkurens viselkedést vigyen a rendszerbe, és egyszerűsítse az olyan objektumokhoz való hozzáférést, amelyek külön szálban futnak.

Ez a Concurrency kategória első és legalapvetőbb patternje. Más nevén: Concurrent Object.

---

## A probléma

Egy egyszálú passzív objektum ugyanabban a szálban hajtja végre a metódust, amelyikben a kliens meghívta. Ha az objektumot több kliens is meghívja párhuzamosan, három feszítő erő áll fenn:

- egy konkurens objektumon meghívott erőforrás-igényes metódus nem blokkolhatja az objektumot korlátlanul, mert ezzel degradálja a többi kliens kiszolgálását
- az olyan kliensoldali metódushívásokat, amelyekre különböző szinkronizációs kényszerek érvényesülnek, átlátszó módon kell sorosítani és ütemezni
- az alkalmazásokat úgy kell megtervezni, hogy az aktuális hardver és szoftverplatform párhuzamosítási lehetőségeit átlátszó módon tudja kezelni

---

## A megoldás

A metódus végrehajtását szétválasztjuk a metódus hívásától. A metódus hívása a klienst kezelő szálban történik, a metódus végrehajtása egy másik szálban megy végbe.

Egy Proxy reprezentálja az aktív objektum interfészét, egy Servant objektum implementálja azt. A Proxy a kliens szálában fut, a Servant egy másikban. A Proxy a metódushívásokat Method Request objektumokká alakítja, amelyeket az Activation List-be helyez. A Scheduler folyamatosan fut a Servant szálában, leveszi a kéréseket a listáról és elküldi a Servant-nak. A kliens egy Future objektumon keresztül kapja meg a metódus végrehajtásának eredményét.

Gondolj rá úgy, mint egy étteremre. A pincér (Proxy) felveszi a rendelést (Method Request) és átadja a konyhának (Activation List). A séf (Servant) a saját tempójában főzi az ételeket, a sorban következő rendelést veszi elő (Scheduler). A vendég (kliens) nem blokkolódik, hanem visszamegy az asztalhoz és vár, amíg az étel megérkezik (Future).

---

## Szereplők

A patternnek hat fő szereplője van:

**Proxy**: interfészt biztosít a kliensek számára. A kliens szálában fut. Minden metódushívást Method Request objektummá alakít, amelyet az Activation List-be helyez. Két-irányú hívásnál Future-t ad vissza a kliensnek.

**Method Request**: absztrakt osztály, amely meghatározza a `can_run()` és `call()` hook metódusok interfészét. A `can_run()` szinkronizációs feltételt értékel ki (guard), a `call()` végrehajtja a metódust a Servant-on.

**Concrete Method Request**: a Method Request egy konkrét implementációja egy adott Proxy metódushoz. Tartalmazza a metódus paramétereit és a Servant referenciáját.

**Activation List**: korlátozott méretű szinkronizált sor, amely a Proxy által létrehozott, végrehajtásra váró Method Request-eket tartalmazza. Leválasztja a kliens szálát a Servant szálától, így a kettő párhuzamosan tud futni.

**Scheduler**: a Servant szálában fut. Folyamatosan figyeli az Activation List-et, meghívja a Method Request-ek `can_run()` metódusát, és amikor egy kérés végrehajtható, eltávolítja a listáról és meghívja a `call()` metódusát.

**Servant**: az aktív objektum állapotát és viselkedését definiálja. Tartalmazza a tényleges metódus-implementációkat és predikátum metódusokat, amelyeket a Method Request-ek `can_run()` metódusai használnak.

**Future**: tárolja a kétirányú metódus eredményét. A kliens blokkolva vagy polling-gal kérdezheti le az eredményt, amikor szüksége van rá.

---

## Struktúra

<svg viewBox="0 0 720 420" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:720px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Client thread label -->
  <rect x="10" y="10" width="320" height="390" rx="10" fill="none" stroke="var(--color-border-tertiary)" stroke-width="1" stroke-dasharray="6 3"/>
  <text x="20" y="30" font-size="10" fill="var(--color-text-secondary)">Kliens szál</text>

  <!-- Active object thread label -->
  <rect x="370" y="10" width="340" height="390" rx="10" fill="none" stroke="var(--color-border-tertiary)" stroke-width="1" stroke-dasharray="6 3"/>
  <text x="380" y="30" font-size="10" fill="var(--color-text-secondary)">Aktív objektum szála</text>

  <!-- Client -->
  <rect x="30" y="50" width="100" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="80" y="68" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Client</text>
  <text x="80" y="84" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">metódust hív</text>

  <!-- Proxy -->
  <rect x="170" y="50" width="130" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="235" y="73" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Proxy</text>
  <text x="235" y="89" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">method request-et</text>
  <text x="235" y="102" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">hoz létre</text>

  <!-- Future -->
  <rect x="30" y="200" width="120" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="90" y="218" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Future</text>
  <text x="90" y="234" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">eredményt tárol</text>

  <!-- Activation List -->
  <rect x="390" y="50" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="460" y="73" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Activation List</text>
  <text x="460" y="89" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">várakozó kérések</text>
  <text x="460" y="102" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">szinkronizált sor</text>

  <!-- Scheduler -->
  <rect x="390" y="200" width="140" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="460" y="223" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Scheduler</text>
  <text x="460" y="239" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">can_run() ellenőriz</text>
  <text x="460" y="252" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">call() meghív</text>

  <!-- Servant -->
  <rect x="560" y="200" width="120" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="620" y="223" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Servant</text>
  <text x="620" y="239" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">tényleges logika</text>
  <text x="620" y="252" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">predikátumok</text>

  <!-- Method Request -->
  <rect x="390" y="340" width="140" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="460" y="358" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Method Request</text>
  <text x="460" y="374" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">can_run() / call()</text>

  <!-- Arrows -->
  <line x1="130" y1="72" x2="170" y2="72" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="150" y="66" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">hív</text>

  <line x1="300" y1="72" x2="390" y2="72" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="345" y="66" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">berak</text>

  <line x1="235" y1="110" x2="90" y2="200" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="135" y="165" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">Future-t ad vissza</text>

  <line x1="460" y1="110" x2="460" y2="200" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="480" y="160" font-size="9" fill="var(--color-text-secondary)">levesz</text>

  <line x1="530" y1="230" x2="560" y2="230" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="545" y="224" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">dispatch</text>

  <line x1="460" y1="260" x2="460" y2="340" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="480" y="305" font-size="9" fill="var(--color-text-secondary)">tartalmaz</text>

  <line x1="620" y1="260" x2="530" y2="340" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="595" y="315" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">használja</text>

  <line x1="90" y1="244" x2="560" y2="244" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="320" y="238" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">eredményt ír bele</text>
</svg>

---

## Működés

A működés három fázisból áll:

**1. Method Request létrehozása és ütemezése**: a kliens meghív egy metódust a Proxy-n. A Proxy létrehozza a megfelelő Concrete Method Request objektumot, amely tartalmazza a metódus paramétereit. Ha a metódus kétirányú (visszatérési értékkel rendelkezik), a Proxy egy Future-t ad vissza a kliensnek. A Proxy berakja a Method Request-et az Activation List-be.

**2. Method Request végrehajtása**: a Scheduler folyamatosan fut a Servant szálában. Végigmegy az Activation List-en, meghívja minden Method Request `can_run()` metódusát, amely a Servant predikátumait használja a szinkronizációs feltétel kiértékeléséhez. Amikor egy Method Request végrehajtható, a Scheduler eltávolítja az Activation List-ből és meghívja a `call()` metódusát, amely végrehajtja a Servant megfelelő metódusát.

**3. Befejezés**: ha a metódus kétirányú volt, az eredmény bekerül a Future-be. A kliens bármikor lekérdezheti a Future-t blokkolva (megvárja az eredményt) vagy polling-gal (rendszeresen ellenőrzi, hogy kész-e).

---

## Implementáció lépései

**1. A Servant implementálása**

A Servant tartalmazza az aktív objektum állapotát és a tényleges logikát. Nem tartalmaz szinkronizációs mechanizmusokat, mert a Scheduler gondoskodik a sorosításról. Predikátum metódusokat is tartalmaz, amelyeket a Method Request-ek `can_run()` metódusai használnak.

```cpp
class MQ_Servant {
public:
    void put(const Message &msg);
    Message get();
    bool empty() const;
    bool full() const;
private:
    // belső sor reprezentáció, szinkronizáció nélkül
};
```

**2. A Method Request implementálása**

Minden Proxy metódushoz egy Concrete Method Request osztályt kell definiálni. A `can_run()` a Servant predikátumait használja, a `call()` meghívja a Servant megfelelő metódusát.

```cpp
class Get : public Method_Request {
public:
    Get(MQ_Servant *rep, const Message_Future &f)
        : servant_(rep), result_(f) {}

    bool can_run() const override {
        return !servant_->empty(); // guard: csak ha van elem
    }

    void call() override {
        result_ = servant_->get(); // eredmény a Future-be
    }
private:
    MQ_Servant *servant_;
    Message_Future result_;
};
```

**3. Az Activation List implementálása**

Az Activation List szinkronizált, korlátozott méretű sor. Általában a Monitor Object pattern segítségével implementálják condition variable-ekkel és mutex-szel. Robusztus iterátort biztosít a Scheduler számára a bejáráshoz és eltávolításhoz.

**4. A Scheduler implementálása**

A Scheduler saját szálban fut, amelyet a konstruktorban indít el. A `dispatch()` metódusa végtelen ciklusban fut, végigmegy az Activation List-en, ellenőrzi a guard-okat és végrehajtja a futtatható kéréseket.

**5. A Proxy implementálása**

A Proxy ugyanolyan interfészt nyújt, mint a Servant. Minden metódushívás létrehoz egy Method Request-et, berakja az Activation List-be, és visszaadja a Future-t.

---

## Ismert felhasználások

**Java java.util.Timer és TimerTask**: a `Timer` osztály háttérszálban hajtja végre a `TimerTask` feladatokat. A `TimerTask` a Method Request szerepét tölti be, a `Timer` belső szála a Scheduler-ét.

**Symbian operációs rendszer**: az Active Object pattern a Symbian párhuzamosítási modelljének alapja. Minden aszinkron műveletet Active Object-ként modelleznek.

**Siemens Automatic Call Distribution**: az Active Object pattern-t használják hívásközpont rendszerekben a hívások és kezelők szétválasztásához.

**ACE framework**: az `ACE_Method_Request`, `ACE_Activation_Queue` és `ACE_Future` osztályok a pattern referencia-implementációját nyújtják.

---

## Valós példa

Egy online játék chat rendszerében a játékosok üzeneteket küldenek egymásnak. Minden üzenet adatbázisba kerül naplózás céljából. Az adatbázis-írás lassú, és nem blokkolhatja a játékmenetet.

Active Object nélkül: minden üzenetküldéskor a szerver megvárja az adatbázis-írást. Ha lassú a kapcsolat, a játékos várakozik.

Active Object-tel az adatbázis-naplózó aktív objektumként valósul meg:

```cpp
// A Proxy fogadja az üzeneteket a kliens szálában
class ChatLogProxy {
public:
    Future<bool> log(const Message &msg) {
        Future<bool> result;
        Method_Request *mr = new LogRequest(&servant_, msg, result);
        scheduler_.insert(mr); // berakja a sorba, nem blokkolja a klienst
        return result;         // azonnal visszatér
    }
private:
    ChatLogServant servant_;
    MQ_Scheduler scheduler_;
};

// A Servant a saját szálában ír az adatbázisba
class ChatLogServant {
public:
    bool log(const Message &msg) {
        db_.write(msg); // lassú, de a kliens szálát nem blokkolja
        return true;
    }
};
```

A játékszerver azonnal visszatér a kliensnek, az adatbázis-írás a háttérben megy végbe. A kliens nem érzékeli a késleltetést.

Ugyanez a minta jelenik meg:

**Képfeldolgozásnál**: egy játék textúra-betöltője Active Object-ként tölt be képeket a háttérben, miközben a játék fut tovább.

**Hálózati kommunikációban**: egy gateway message queue-t Active Object-ként modellez. A `put()` hívás azonnal visszatér, a `get()` megvárja, amíg van üzenet a sorban.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Konkurencia növelése és szinkronizációs komplexitás csökkentése | Teljesítmény overhead |
| Átlátszó párhuzamosítás | Debuggolás nehézsége |
| | A végrehajtás sorrendje eltérhet a hívás sorrendjétől |

**Előnyök részletesen:**

**Konkurencia növelése és szinkronizációs komplexitás csökkentése**: a kliens és a Servant párhuzamosan futhatnak anélkül, hogy explicit szinkronizációs kódot kellene írni. A Scheduler kezeli a szinkronizációs kényszereket a `can_run()` guard-okon keresztül.

**Átlátszó párhuzamosítás**: a kliens ugyanolyan interfészen keresztül hívja az Active Object-et, mint egy passzív objektumot. A párhuzamos végrehajtás transzparens, a kliens kódját nem kell módosítani.

**Hátrányok részletesen:**

**Teljesítmény overhead**: a Method Request objektumok létrehozása, az Activation List kezelése és a kontextusváltás extra overhead-et jelent. Rövid, gyorsan lefutó metódusok esetén ez meghaladhatja a párhuzamosítás előnyét.

**Debuggolás nehézsége**: a hívás és a végrehajtás különböző szálakban és időpontokban történik, ami megnehezíti a hibakeresést.

**A végrehajtás sorrendje eltérhet a hívás sorrendjétől**: ha a Scheduler prioritás-alapú ütemezést alkalmaz, a Method Request-ek más sorrendben hajtódhatnak végre, mint ahogy a Proxy-n meghívták őket.

---

## Kapcsolata más patternekkel

A **Monitor Object** pattern az Activation List implementálásához szükséges, szinkronizált bounded buffer-ként. Az Active Object és Monitor Object együtt a legelterjedtebb konkurens tervezési megközelítés.

A **Proxy pattern** (GoF, POSA1) az Active Object Proxy komponensének alapja: az interfészt biztosítja és elrejti, hogy a Servant másik szálban fut.

A **Scheduler** komponens a **Command Processor pattern** (POSA1) megvalósítása: a Method Request-ek a parancsok, a Scheduler a parancsprocesszor.

A **Future** mechanizmus az **Asynchronous Completion Token** patternhez hasonlóan működik: az eredmény tárolójaként és a kliens értesítési mechanizmusaként szolgál.

A **Half-Sync/Half-Async** pattern általánosabb képet ad: az Active Object egy speciális esete, ahol pontosan egy szinkron és egy aszinkron réteg van.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

Az Active Object szétválasztja a metódus hívását a végrehajtásától: a Proxy a kliens szálában hívódik meg, a Servant saját szálában hajtja végre a metódust. Hat szereplője van: Proxy, Method Request, Concrete Method Request, Activation List, Scheduler és Servant, plusz a Future az eredmény visszajuttatásához. A Proxy Method Request-eket hoz létre és rakja az Activation List-be. A Scheduler folyamatosan fut, ellenőrzi a can_run() guard-okat, és végrehajtja a futtatható kéréseket a Servant-on. Előnye a transzparens párhuzamosítás és az egyszerűbb szinkronizáció. Hátránya a teljesítmény overhead és a debuggolás nehézsége.
</div>