---
layout: default
title: Extension Interface
---

# Extension Interface (Kiegészítő Interfész)

## Mi ez a pattern

Az Extension Interface <mark>olyan alkalmazáskörnyezetben hasznos, ahol a komponensek interfészei fejlődnek az idő folyamán</mark>. A pattern lehetővé teszi, hogy egy komponens több különálló interfészen keresztül legyen elérhető, és ezek az interfészek egymástól függetlenül bővíthetők legyenek anélkül, hogy a meglévő klienskódot meg kellene változtatni.

Ez a <mark>negyedik és egyben utolsó pattern a Service Access and Configuration kategóriában</mark>. Szorosan kapcsolódik az Interceptor patternhez: az Interceptor alkalmazhatja az Extension Interface-t arra, hogy egyetlen objektum több különböző interceptor interfészt is megvalósítson.

---

## A probléma

Képzeld el, hogy van egy `NetworkElement` komponensed, amelyet különböző kliensek különböző célokra használnak: egy monitorozó alkalmazás az állapotát olvassa, egy perzisztencia menedzser menti és visszatölti, egy diagnosztikai eszköz hibákat keres benne.

A naiv megoldás: egyetlen nagy interfészbe teszed az összes metódust. Ez három problémát okoz:

- <mark>ha a komponens implementációja változik, az törhet olyan klienskódot is, amely az érintett funkciót egyáltalán nem használja</mark>, csak azért mert ugyanabból az interfészből függ
- <mark>ha új funkciókat adsz hozzá, a meglévő interfész egyre nagyobb és bonyolultabb lesz</mark>, és a kliens újrafordítása is szükségessé válhat
- <mark>a komponensek funkcionalitásának megváltoztatása instabillá teheti a belső architektúrát</mark>, mert minden változás az egyetlen interfészen keresztül gyűrűzik tovább

Ráadásul elosztott rendszerekben a komponensek és klienseik különböző hálózati csomópontokon lehetnek, így a komponens interfészét és implementációját mindenképpen szét kell választani.

---

## A megoldás

<mark>Exportáljuk a komponens funkcionalitását kiegészítő interfészeken keresztül, amelyek mindegyike műveletek egy szemantikusan összefüggő halmazának felel meg.</mark> Minden komponensnek legalább egy kiegészítő interfészt meg kell valósítania.

A komponens meglévő funkcionalitásának módosítása vagy kiegészítése céljából <mark>inkább hozzunk létre újabb kiegészítő interfészeket ahelyett, hogy a meglévőket módosítanánk</mark>. A klienseket úgy programozzuk, hogy a komponenseket ne az implementációjukon, hanem az interfészeiken keresztül érjék el.

Ahhoz, hogy a kliensek létre tudjanak hozni komponens példányokat és el tudják érni a kiegészítő interfészeket, vezessünk be egy **Component Factory**-t (komponens üzemet), amely a komponens példányosítására szolgál, és egy **Root Interface** (gyökér interfész) referenciát ad vissza. A Root Interface-ből kiindulva a kliens bármely más kiegészítő interfészhez eljuthat a `getExtension()` metóduson keresztül.

---

## Szereplők

A patternnek <mark>öt szereplője van</mark>:

**Component (Komponens)**: különböző típusú szolgáltatásspecifikus funkcionalitást valósít meg. Implementálja az összes kiegészítő interfészt, de a kliensek ezt közvetlenül soha nem látják.

**Extension Interface (Kiegészítő interfész)**: a komponens implementációjának egy-egy különálló szerepét exportálja. Minden kiegészítő interfész metódusainak összessége egy szemantikusan összefüggő csoportot alkot. Minden kiegészítő interfész a Root Interface-ből származik.

**Root Interface (Gyökér interfész)**: egy speciális kiegészítő interfész, amely három dolgot biztosít: a `getExtension()` metódust a többi interfész lekéréséhez, domain-független funkciókat (pl. referencia-számlálás), és domain-specifikus funkciókat, amelyeket minden komponensnek meg kell valósítania.

```java
// A Root Interface minimális definíciója
public interface IRoot {
    IRoot getExtension(int ID) throws UnknownEx;
}
```

**Component Factory (Komponens üzem)**: elkülöníti a komponens létrehozását és inicializálását a feldolgozástól. A kliens ezen keresztül kér új komponens példányt, és visszakap egy Root Interface referenciát.

**Client Application (Kliens alkalmazás)**: igénybe veszi a komponens szolgáltatásait kizárólag interfészeken keresztül. A komponens implementációját soha nem látja közvetlenül.

---

## Struktúra

<svg viewBox="0 0 720 380" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:720px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Client -->
  <rect x="20" y="160" width="120" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="80" y="177" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Client</text>
  <text x="80" y="194" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">interfészen át ér el</text>

  <!-- Component Factory -->
  <rect x="20" y="40" width="120" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="80" y="57" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Component</text>
  <text x="80" y="72" text-anchor="middle" font-size="11" font-weight="600" fill="var(--color-text-primary)">Factory</text>

  <!-- Root Interface -->
  <rect x="220" y="160" width="140" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="290" y="177" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Root Interface</text>
  <text x="290" y="194" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">getExtension(ID)</text>

  <!-- Extension Interface A -->
  <rect x="440" y="80" width="140" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="510" y="97" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Extension Interface A</text>
  <text x="510" y="112" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">pl. IStateMemory</text>

  <!-- Extension Interface B -->
  <rect x="440" y="240" width="140" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="510" y="257" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Extension Interface B</text>
  <text x="510" y="272" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">pl. IManagedObject</text>

  <!-- Component Implementation -->
  <rect x="600" y="160" width="100" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="650" y="177" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Component</text>
  <text x="650" y="194" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">implementáció</text>

  <!-- Client -> Factory -->
  <line x1="80" y1="160" x2="80" y2="84" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="55" y="125" font-size="9" fill="var(--color-text-secondary)">létrehoz</text>

  <!-- Client -> Root Interface -->
  <line x1="140" y1="182" x2="220" y2="182" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="180" y="176" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">használja</text>

  <!-- Factory -> Root Interface -->
  <line x1="140" y1="75" x2="280" y2="163" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="190" y="108" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">visszaad</text>

  <!-- Root -> Extension A -->
  <line x1="360" y1="172" x2="440" y2="110" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="415" y="134" font-size="9" fill="var(--color-text-secondary)">getExtension()</text>

  <!-- Root -> Extension B -->
  <line x1="360" y1="192" x2="440" y2="252" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="415" y="232" font-size="9" fill="var(--color-text-secondary)">getExtension()</text>

  <!-- Extension A -> Component -->
  <line x1="580" y1="110" x2="630" y2="160" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>

  <!-- Extension B -> Component -->
  <line x1="580" y1="252" x2="630" y2="204" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
</svg>

---

## Működés

A működés két fő szcenárióból áll:

**1. szcenárió: komponens létrehozása és interfész lekérése**

1. A kliens megkéri a Component Factory-t, hogy hozzon létre egy új komponenst és adjon vissza egy adott interfész referenciát
2. A Component Factory létrehozza a komponenst és lekéri a Root Interface referenciát
3. A Component Factory megkéri a Root Interface-t a kívánt kiegészítő interfész referenciájáért, majd visszaadja azt a kliensnek

**2. szcenárió: navigálás az interfészek között**

1. A kliens meghív egy metódust az A kiegészítő interfészen
2. A komponens végrehajtja a metódust és visszaadja az eredményt
3. A kliens meghívja a `getExtension()` metódust az A interfészen, megadva a kívánt B interfész azonosítóját
4. A komponens megkeresi a B kiegészítő interfészt és visszaadja annak referenciáját
5. A kliens meghívja a B interfész metódusait

---

## Implementáció lépései

**1. A domain elemzése és a komponens modell meghatározása**

Azonosítani kell a különböző szerepeket, amelyeket a komponens betölt. Például egy hálózati elem komponens lehet monitorozható, perzisztálható és konfigurálható. Minden szerephez egy külön kiegészítő interfész tartozik.

**2. A Root Interface meghatározása**

A Root Interface-nek minimálisan a `getExtension()` metódust kell tartalmaznia. Ide kerülnek azok a domain-független funkciók, amelyeket minden komponensnek meg kell valósítania, például referencia-számlálás C++ esetén, vagy runtime reflection mechanizmus.

```java
public interface IRoot {
    IRoot getExtension(int ID) throws UnknownEx;
}
```

**3. Az interfész-kiterjesztési mechanizmus megadása**

Meg kell határozni, hogyan azonosítjuk a kiegészítő interfészeket. Ez lehet egyedi egész szám konstans, GUID, vagy string azonosító. Ha a komponens nem támogatja a kért interfészt, kivételt dob vagy null-t ad vissza.

**4. A Component Factory szerepének pontosítása**

A Factory felelős a komponens példányosításáért és az első interfész referencia visszaadásáért. Meghatározandó, hogy a Factory milyen típusú kezdeti interfészt adjon vissza, és hogy képes-e meglévő komponens példányokat is visszaadni.

<div class="callout tip" markdown="1">
**Tipp:** ha sok komponens végül ugyanazt a kiegészítő interfészt implementálja, érdemes azt a Root Interface-be felvenni. Ezzel csökken a klienskód bonyolultsága.
</div>

---

## Ismert felhasználások

**Microsoft COM és COM+**: a leghíresebb megvalósítása a patternnek. Minden COM osztálynak van egy `IClassFactory` factory interfésze. Minden COM osztály az `IUnknown` Root Interface-ből örökli a `QueryInterface(REFIID, void**)` metódust, amellyel a kliens bármely kiegészítő interfészt lekérhet. Ezt hívják interface negotiation-nek.

**CORBA 3 Component Model (CCM)**: minden komponens rendelkezik egy "equivalent" interfésszel (Root Interface szerepe). A kiegészítő interfészeket "facet"-eknek hívják, a `get_component()` metódussal érhetők el. A `ComponentHomeFinder` a Component Factory szerepét tölti be.

**Enterprise JavaBeans (EJB)**: a CORBA CCM Java-centric megvalósítása. A JNDI (Java Naming and Directory Interface) a factory finder szerepét tölti be, az EJB Home interfész a Component Factory-nak felel meg.

**OpenDoc**: az Extension Object variánst valósítja meg, ahol az interfész kiterjesztési mechanizmus közvetlenül a programozási nyelv objektummodelljére épül.

---

## Valós példa

Képzeld el, hogy egy játékban különböző entitások vannak: NPC-k, ellenségek, tárgyak. Mindegyiket más-más rendszer használja: a renderelő motor a megjelenítési adatokat olvassa, a fizikai motor az ütközési adatokat, az AI motor a döntési állapotot, a mentési rendszer a perzisztens adatokat.

A naiv megközelítés: egy nagy `GameObject` osztály az összes metódussal. Ha a fizikai motor megváltozik, az összes kliens érintett, és az osztály egyre nagyobb lesz.

Az Extension Interface megoldása:

```java
// Root Interface: minden entitás ezt implementálja
public interface IGameObject {
    IGameObject getExtension(int ID) throws UnknownEx;
}

// Kiegészítő interfészek
public interface IRenderable extends IGameObject {
    Mesh getMesh();
    Material getMaterial();
}

public interface IPhysicsBody extends IGameObject {
    CollisionShape getShape();
    Vector3 getVelocity();
}

public interface ISaveable extends IGameObject {
    byte[] serialize();
    void deserialize(byte[] data);
}
```

Az NPC osztály mindhármat implementálja, de a kliensek csak azt a részt látják, ami nekik kell:

```java
// A renderelő csak az IRenderable interfészt kéri le
IRenderable renderable = (IRenderable) npc.getExtension(IRenderable.ID);
renderable.getMesh(); // csak ezt tudja, más metódust nem lát
```

Ha holnap egy új `INetworkSync` interfészt kell hozzáadni, csak az NPC osztályt bővítjük. A renderelő, a fizikai motor és a mentési rendszer kódja egyáltalán nem változik.

Ugyanez a minta működik:

**Pluginrendszereknél**: egy szövegszerkesztő plugin lekérheti, hogy az aktuális dokumentum implementálja-e az `ISpellCheckable` interfészt. Ha igen, lefuttatja a helyesírás-ellenőrzést. Ha nem, csak kihagyja.

**Böngészőbővítményeknél**: az Angular dependency injection rendszerben a service-ek interfészeken keresztül érhetők el. Ha egy service új képességet kap, a régi kliensek nem törnek el.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Kiterjeszthetőség | Ráfordítás növekedése komponenstervezés és implementáció terén |
| Szemantikus aspektusok szétválasztása | Teljesítmény overhead |
| Polimorfizmus | Kliens komplexitásának növekedése |
| Interfész aggregáció és delegáció | |

**Előnyök részletesen:**

**Kiterjeszthetőség**: új interfészek adhatók hozzá a meglévők módosítása nélkül. A klienskód sem változik, ha olyan interfész bővül, amelyet az adott kliens nem használ.

**Szemantikus aspektusok szétválasztása**: minden kiegészítő interfész egyetlen összefüggő szerepet exportál. A kliensek pontosan azt a részt látják, amire szükségük van, a többit nem.

**Polimorfizmus**: a kliensek az interfészen, nem az implementáción keresztül dolgoznak. Futásidőben más implementáció is behelyettesíthető ugyanarra az interfészre.

**Interfész aggregáció és delegáció**: egy komponens több különböző interfészt implementálhat, és ezek tetszőleges módon kombinálhatók. A `getExtension()` mechanizmus lehetővé teszi a dinamikus navigációt az interfészek között.

**Hátrányok részletesen:**

**Ráfordítás növekedése**: a komponens tervezése és implementálása több munkát igényel, mert az összes szerepet előre fel kell tárni és interfészekre kell bontani. C-ben implementálni rendkívül bonyolult az öröklődés és polimorfizmus hiánya miatt.

**Teljesítmény overhead**: a kliensek sosem érik el közvetlenül a komponenst, ami plusz indirectiont jelent. A `getExtension()` hívások, különösen elosztott rendszerekben, extra hálózati körutakat okozhatnak.

**Kliens komplexitásának növekedése**: a kliensnek egy több lépéses protokollt kell követnie, hogy eljusson a kívánt interfészig. Referencia-számlálást is kell kezelnie C++ esetén, ami elbonyolíthatja az alkalmazáslogikát.

---

## Kapcsolata más patternekkel

A **Proxy pattern** (POSA1, GoF) alkalmazható arra az esetre, amikor a komponens és a kliens különböző címterületen van. A Proxy elrejti a hálózati kommunikációt.

A **Broker pattern** (POSA1) egy szofisztikáltabb megoldás elosztott esetben: a Broker globálisan elérhető factory finder szolgáltatást nyújt, a komponensek szerverként viselkednek.

Az **Interceptor** pattern alkalmazhatja az Extension Interface-t arra, hogy egyetlen fizikai interceptor objektum több különböző logikai interceptor interfészt valósítson meg.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

Az Extension Interface egy komponens funkcionalitását több különálló kiegészítő interfészen keresztül exportálja, ahol minden interfész egy szemantikusan összefüggő szerepet képvisel. Öt szereplője van: Component, Extension Interface, Root Interface, Component Factory és Client. A Root Interface `getExtension()` metódusán keresztül a kliens bármely kiegészítő interfészhez eljuthat. Új funkcionalitás hozzáadásához új interfészt hozunk létre a meglévők módosítása helyett. A legismertebb megvalósítása a Microsoft COM `IUnknown` és `QueryInterface()` mechanizmusa. Előnye a kiterjeszthetőség és a szemantikus szétválasztás. Hátránya a nagyobb tervezési ráfordítás és a kliens komplexitásának növekedése.
</div>