---
layout: default
title: Component Configurator
---

# Component Configurator (Komponensbeállító)

## Mi ez a pattern

A Component Configurator lehetővé teszi, hogy az alkalmazások a különböző komponens implementációkat <mark>futási időben csatlakoztassák illetve leválasszák</mark> az alkalmazás módosítása, újrafordítása vagy statikus újralinkelése nélkül. Ezen kívül támogatja a komponensek átkonfigurálását különböző alkalmazásfolyamatokhoz a folyamatok újraindítása nélkül.

A **komponens** ebben a kontextusban egy <mark>önálló, cserélhető szoftvermodul</mark>, amely egy adott szolgáltatást nyújt, és egy egységes interfészen keresztül vezérelhető.

Ez a második pattern a Service Access and Configuration kategóriában, és szorosan épít a Wrapper Facade-re: a DLL-betöltő OS API-kat Wrapper Facade osztályok rejtik el.

---

## A probléma

Hosszan futó alkalmazásoknál az igények idővel változnak:

- a komponens funkcionalitása változhat, a teljesítmény bizonyulhat nem elegendőnek
- közös adminisztrációs feladatokat kell elvégezni: inicializálás, felfüggesztés, leállítás
- platform frissítés vagy megnövekedett terhelés miatt komponenseket más processzorokra kell áttelepíteni
- olyan szolgáltatásokhoz kell hozzáférni, amelyek implementációja az alkalmazás tervezésekor még nem ismert

A hagyományos megközelítés: leállítod az alkalmazást, átírod, újrafordítod, újraindítod. <mark>Egy éjjel-nappal futó szerveren ez elfogadhatatlan</mark>, mert minden leállás kiesést jelent.

---

## A megoldás

<mark>Válasszuk szét a komponensek interfészeit az implementációjuktól</mark>, továbbá a komponens kínáljon egy egységes interfészt az általa nyújtott szolgáltatás vagy funkcionalitás konkrét típusának konfigurációjára és vezérlésére.

Ez a forma a **DLL** (dynamically linked library, dinamikusan linkelt könyvtár). A DLL egy olyan fájl, amelyet az operációs rendszer <mark>futás közben tud betölteni</mark> a program memóriájába, és ugyanúgy el is tud távolítani belőle anélkül, hogy a program többi részét érintené.

Gondolj rá úgy, mint egy futballmeccsre. A csapatban 11 játékos van. Az edző az egyik játékost cserére küldi és azonnal behozza a tartalékost. A meccs nem áll meg, a többi játékos folytatja. Az edző nyilvántartja, ki van éppen pályán. Ez pontosan a Component Configurator működése: az edző a Configurator, a játékosok a komponensek, a névsor a Repository.

---

## Szereplők

A patternnek <mark>négy szereplője van</mark>. A futballmeccs-hasonlatban:

| Szereplő | Szerep | Edző-hasonlat |
|---|---|---|
| **Komponens** | egységes interfészt definiál a konfigurációra és vezérlésre | a játékos pozíciója (egységes elvárás) |
| **Konkrét komponensek** | a komponens konkrét implementációi | maguk a játékosok |
| **Komponenstár** | komponenstároló (Repository) | a névsor, ki van pályán |
| **Komponensbeállító** | a **főnök / Coach**, a vezérlő | az **edző** |

**Component (Komponens interfész)**: <mark>egységes interfészt definiál a konfigurációra és vezérlésre</mark>. Minden konkrét komponensnek tudnia kell inicializálódni, futni, szünetelni, folytatni és leállni.

```cpp
class Component {
public:
    virtual void init(int argc, const char *argv[]) = 0; // betöltéskor hívódik
    virtual void fini() = 0;                              // eltávolításkor hívódik
    virtual void suspend();                               // szüneteltetés
    virtual void resume();                                // folytatás
    virtual void info(string &status) const = 0;         // állapotlekérdezés
};
```

**Concrete Component (Konkrét komponens)**: a Component interfész konkrét implementációja, <mark>DLL fájlba csomagolva</mark>. Az alkalmazás ezeket tölti be és távolítja el futás közben.

**Component Repository (Komponenstár)**: <mark>nyilvántartja az éppen betöltött és aktív komponenseket</mark>. Az edző névsorának felel meg.

```cpp
class Component_Repository {
public:
    void add(const string &name, Component *component);
    Component *find(const string &name);
    void remove(const string &name);
    void suspend(const string &name);
    void resume(const string &name);
};
```

**Component Configurator (Komponensbeállító)**: elvégzi a tényleges munkát. Értelmez egy konfigurációs szkriptet, <mark>betölti a megfelelő DLL-t, meghívja a komponens `init()` metódusát, bejegyzi a Repository-ba</mark>, és szükség esetén eltávolítja. Egyetlen példányban létezik az alkalmazásban (Singleton façade).

```cpp
class Component_Configurator {
public:
    void process_directives(const string &script_name); // szkriptfájl feldolgozása
    void process_directive(const string &directive);    // egyetlen direktíva
    Component_Repository *component_repository();
    static Component_Configurator *instance();          // Singleton hozzáférés
};
```

---

## Struktúra

<svg viewBox="0 0 680 340" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:680px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>
  <rect x="240" y="20" width="200" height="60" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="340" y="44" text-anchor="middle" font-size="13" font-weight="600" fill="var(--color-text-primary)">Component</text>
  <text x="340" y="62" text-anchor="middle" font-size="11" fill="var(--color-text-secondary)">init() / fini() / suspend() / resume()</text>
  <rect x="60" y="150" width="160" height="50" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="140" y="170" text-anchor="middle" font-size="13" font-weight="600" fill="var(--color-text-primary)">Concrete Component A</text>
  <text x="140" y="188" text-anchor="middle" font-size="11" fill="var(--color-text-secondary)">pl. cristian.dll</text>
  <rect x="260" y="150" width="160" height="50" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="340" y="170" text-anchor="middle" font-size="13" font-weight="600" fill="var(--color-text-primary)">Concrete Component B</text>
  <text x="340" y="188" text-anchor="middle" font-size="11" fill="var(--color-text-secondary)">pl. berkeley.dll</text>
  <line x1="200" y1="80" x2="160" y2="150" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <line x1="340" y1="80" x2="340" y2="150" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <rect x="460" y="150" width="180" height="50" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="550" y="170" text-anchor="middle" font-size="13" font-weight="600" fill="var(--color-text-primary)">Component Repository</text>
  <text x="550" y="188" text-anchor="middle" font-size="11" fill="var(--color-text-secondary)">add / find / remove</text>
  <rect x="240" y="265" width="200" height="55" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="340" y="287" text-anchor="middle" font-size="13" font-weight="600" fill="var(--color-text-primary)">Component Configurator</text>
  <text x="340" y="305" text-anchor="middle" font-size="11" fill="var(--color-text-secondary)">process_directives(script)</text>
  <line x1="440" y1="290" x2="490" y2="200" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="492" y="255" font-size="10" fill="var(--color-text-secondary)" text-anchor="middle">kezeli</text>
  <line x1="290" y1="265" x2="190" y2="200" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="218" y="242" font-size="10" fill="var(--color-text-secondary)" text-anchor="middle">betölti / eltávolítja</text>
</svg>

---

## Működés

A komponens életciklusa három fázisból áll:

**Komponens elindítás**: a Component Configurator <mark>dinamikusan csatol egy komponenst</mark> az alkalmazáshoz és inicializálja azt. A sikeres inicializáció után a komponenst hozzáadja a Component Repository-hoz.

**Komponens használat**: az alkalmazáshoz való csatlakozás után a komponens különböző feladatokat végezhet, például más komponensekkel való üzenetváltás vagy kérések kiszolgálása. A Configurator <mark>ideiglenesen felfüggesztheti és újraindíthatja</mark> a komponenst, például a többi komponens átkonfigurálása esetén.

**Komponens leállítás**: a Configurator leállítja azokat a komponenseket, amelyekre nincsen szükség. Leállítás során a komponenseknek <mark>fel kell szabadítaniuk az erőforrásaikat</mark>. A sikeres leállás után a Configurator eltávolítja a komponenst a Repository-ból és az alkalmazás címteréből.

A konfiguráció egy `comp.conf` nevű szkriptből olvasódik be:

```bash
# Time_Server betöltése a cristian.dll-ből
dynamic Time_Server Component * cristian.dll:make_Time_Server() "-p $TIME_SERVER_PORT"

# Clerk betöltése
dynamic Clerk Component * cristian.dll:make_Clerk() "-h tango.cs:$TIME_SERVER_PORT"
```

---

## Implementáció lépései

**1. A komponens konfigurációs és vezérlési interfész meghatározása**

Minden komponensnek <mark>támogatnia kell az `init()`, `fini()`, `suspend()`, `resume()` és `info()` műveleteket</mark>. Az interfész megvalósítható öröklődés alapon (közös absztrakt osztály) vagy üzenetalapú stratégiával.

**2. A Component Repository implementálása**

A Repository nyilvántartja az összes betöltött komponenst. `add()`, `find()`, `remove()`, `suspend()` és `resume()` metódusokat tartalmaz.

**3. A Component Configurator implementálása**

A Configurator <mark>Singleton Facade-ként valósul meg</mark>. Értelmez egy szkriptfájlt, amely megmondja, melyik DLL-ből melyik factory függvényt kell meghívni. A DLL-betöltő OS API-kat Wrapper Facade rejti el:

```cpp
class DLL {
public:
    DLL(const string &dll_name) { handle_ = OS::dlopen(dll_name.c_str()); }
    ~DLL() { OS::dlclose(handle_); }
    void *symbol(const string &name) { return OS::dlsym(handle_, name.c_str()); }
private:
    HANDLE handle_;
};
```

**4. Az átkonfigurálás kiváltási stratégiája**

Kétféle stratégia létezik arra, hogy mikor futjon le az újrakonfiguráció:

- **In-band**: <mark>szinkron módon</mark>, például Socket kapcsolaton vagy CORBA operáción keresztül érkezik a jel. Kevésbé hajlamos race condition-re, ezért <mark>általában ezt érdemes választani</mark>.
- **Out-of-band**: <mark>aszinkron esemény váltja ki</mark>, például UNIX SIGHUP szignál. Ez megszakítja a futó folyamatot.

---

## Ismert felhasználások

**Windows Service Control Manager (SCM)**: a Windows szolgáltatásokat (Windows Update, tűzfal, nyomtatási sor) az SCM kezeli. Minden szolgáltatás reagál a <mark>PAUSE, RESUME és TERMINATE üzenetekre</mark>. Ez a Component Configurator mintájának közvetlen megvalósítása.

**Java appletek**: a böngésző dinamikusan tölt le és indít el Java appleteket. A `java.applet.Applet` `init()`, `start()`, `stop()` és `destroy()` metódusai pontosan a Component Configurator életciklus-interfészét valósítják meg.

**Operációs rendszer driverek**: Linux és Windows egyaránt támogatja, hogy eszközmeghajtókat a rendszer újraindítása nélkül be és ki lehessen tölteni. Az SVR4 UNIX `init()`, `fini()` és `info()` hook függvényeket definiál erre.

**ACE Service Configurator framework**: az ACE a Component Configurator mintát valósítja meg kommunikációs szolgáltatás komponensekre, a Reactor, Acceptor-Connector és Active Object patternekkel együtt használva.

---

## Valós példa

Képzeld el, hogy egy online többjátékos játékot fejlesztesz. A szerveren több komponens fut: fizikai motor, AI motor és matchmaking rendszer.

Játék közben kiderül, hogy a fizikai motor hibás: a karakterek egy edge case-ben a falba ragadnak. A hiba javítva van, de a szerveren 5000 játékos játszik.

Nélküle: leállítod a szervert, mindenkit kidobsz, felrakod a javítást, újraindítod. 5000 ember elveszíti a haladását.

Component Configurator-ral a `comp.conf` szkriptet módosítod:

```bash
# Régi fizikai motor leállítása
remove PhysicsEngine

# Javított verzió betöltése
dynamic PhysicsEngine Component * physics_v2.dll:make_PhysicsEngine()
```

A Configurator leállítja a régi fizikai motort, betölti az újat. Az AI, a matchmaking és a hálózati kezelő fut tovább, a játékosok a szerveren maradnak.

Ugyanez a módszer működik webszervereknél is: az Apache és az Nginx modulokat tölt be futás közben. Egy új SSL modul bekapcsolható a szerver leállítása nélkül.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Egységes komponenshozzáférés | Biztonság csökkenése |
| Központi adminisztrálhatóság | Teljesítmény overhead |
| Dinamikus komponenscsatlakozás és vezérlés | Bonyolultság növekedése |
| Modularitás és optimalizálás | Túl szűk közös interfész |

**Előnyök részletesen:**

**Egységes komponenshozzáférés**: <mark>minden komponens ugyanazt az interfészt implementálja</mark>. Az adminisztrátor egyforma módon kezeli az összes komponenst, függetlenül attól, hogy az adott komponens mit csinál. Ez a "principle of least surprise" elvét érvényesíti.

**Központi adminisztrálhatóság**: az inicializálás, szüneteltetés és leállítás <mark>egyetlen helyen, a Configurator-ban van kezelve</mark>. Nem kell minden komponens kódjába beleírni az adminisztrációs logikát.

**Dinamikus komponenscsatlakozás és vezérlés**: egy komponens újrakonfigurálható a meglévő kód módosítása, újrafordítása vagy <mark>a többi komponens újraindítása nélkül</mark>.

**Modularitás és optimalizálás**: a komponensek önálló DLL-ekbe vannak csomagolva, így <mark>külön fejleszthetők és tesztelhetők</mark>. A rendszer alkalmazkodhat a változó terheléshez.

**Hátrányok részletesen:**

**Biztonság csökkenése**: egy DLL-ből betöltött komponens <mark>megszemélyesíthet egy legitim komponenst</mark>. Egy hibás komponens megrongálhatja a megosztott állapotot.

**Teljesítmény overhead**: a Configurator inicializálja a komponenseket és linkeli őket a Repository-ba, ami <mark>plusz indirectiont jelent</mark>. Dinamikus linkelés esetén a fordítók extra szintű indirectiont adnak hozzá a metódushívásokhoz.

**Bonyolultság növekedése**: a Configurator, a Repository és a DLL-betöltési mechanizmus mind plusz kódrétegek, amelyek <mark>nehezíthetik a hibakeresést</mark>.

**Túl szűk közös interfész**: egyes komponensek inicializálása olyan összetett, hogy <mark>nem fér bele az egységes `init()` és `fini()` interfészbe</mark>.

---

## Kapcsolata más patternekkel

A Component Configurator belül **Wrapper Facade**-eket használ a <mark>DLL-betöltő OS API-k eléréséhez</mark>. A `dlopen()`, `dlsym()` és `dlclose()` UNIX hívásokat, illetve a Windows `LoadLibrary()`, `GetProcAddr()` és `CloseHandle()` hívásokat egy `DLL` Wrapper Facade osztály rejti el.

A **Reactor** patternnel együtt használják: a komponensek regisztrálhatják magukat egy Reactor-ban, amely eseményeket küld nekik. A Component Configurator <mark>felelős a komponensek betöltéséért, a Reactor a futásukért</mark>.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

A Component Configurator lehetővé teszi, hogy futó alkalmazásokból komponenseket ki és be lehessen cserélni újraindítás nélkül. Négy szereplője van: Component interfész, Concrete Component, Component Repository és Component Configurator. A komponensek egységes interfészt valósítanak meg (init, fini, suspend, resume), DLL-ekbe vannak csomagolva, és a Configurator tölti be, regisztrálja és távolítja el őket szkriptfájl alapján. Az átkonfigurálást in-band (szinkron) vagy out-of-band (aszinkron szignál) stratégiával lehet kiváltani. Előnye a futás közbeni rugalmasság és a központosított adminisztráció. Hátránya a biztonsági kockázatok, a teljesítmény overhead és a bonyolultabb infrastruktúra.
</div>