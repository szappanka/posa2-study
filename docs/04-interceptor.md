---
layout: default
title: Interceptor
---

# Interceptor

## Mi ez a pattern

Az Interceptor <mark>átlátszó módon egészít ki keretrendszereket különböző szolgáltatásokkal</mark>. Szolgáltatásokat lehet hozzárendelni a keretrendszer bizonyos belső eseményeihez. Az események előfordulásakor ezek a szolgáltatások automatikusan lefutnak, anélkül hogy a keretrendszer tudna róluk.

Ez a harmadik pattern a Service Access and Configuration kategóriában. Szorosan kapcsolódik a Component Configurator-hoz: interceptorokat Component Configurator segítségével is lehet dinamikusan betölteni futás közben.

---

## A probléma

Egy keretrendszer (például egy webszerver, egy ORB vagy egy alkalmazásszerver) sok helyen végez általános feladatokat: naplózás, hitelesítés, terheléselosztás, tranzakciókezelés. Ha ezeket a keretrendszerbe égeted, három probléma keletkezik:

- az új szolgáltatások hozzáadásához <mark>módosítani kell a keretrendszer kódját</mark>, ami törékeny és kockázatos
- az új szolgáltatások hozzáadása módosíthatja a keretrendszer eddigi működését és a funkcióinak igénybevételének módját
- <mark>a keretrendszer működése nem monitorozható</mark> külső beavatkozás nélkül

A cél: a keretrendszert úgy kell megtervezni, hogy <mark>mások bővíthessék</mark> anélkül, hogy a belső kódját ismernék vagy módosítanák.

---

## A megoldás

Definiáljunk a keretrendszer belsejében bizonyos eseményekhez rendelt <mark>beavatkozási pontokat (interception points)</mark>. Minden beavatkozási ponthoz definiáljunk egy külön interfészt (interceptor interface), amely egy vagy több callback metódust tartalmaz. Rendeljünk minden beavatkozási ponthoz egy dispatcher-t, amely lehetővé teszi a kliensek számára, hogy szolgáltatásokat regisztráljanak az adott beavatkozási ponthoz.

Ha a keretrendszer elér egy beavatkozási pontot, <mark>a dispatcher meghívja az összes regisztrált interceptor callback metódusát</mark>. Az interceptor egy context object-en keresztül hozzáférhet a keretrendszer belső állapotához, és szükség esetén befolyásolhatja is azt.

Gondolj rá úgy, mint a levéltovábbításra. Megváltozik a lakcímed, de nem kell mindenkinek szólnod. A posta automatikusan elfogja a régi címre érkező leveleket és továbbítja az újra. Te ezt nem is látod, az eredeti küldő sem tudja, hogy átirányítás történt.

---

## Szereplők

A patternnek <mark>hat szereplője van</mark>:

**Concrete Framework (Konkrét keretrendszer)**: a keretrendszer maga, amely <mark>eseményeket generál és beavatkozási pontokat definiál</mark>. Példányosítja a Context Object-et és értesíti a Dispatcher-t az eseményekről.

**Interceptor**: egy interfész, amely <mark>meghatározza a callback hook metódusok szignatúráját</mark>. Minden beavatkozási ponthoz egy-egy hook metódus tartozik. Megfelel az Observer pattern Observer szerepének.

**Concrete Interceptor (Konkrét interceptor)**: az Interceptor interfész <mark>alkalmazásspecifikus implementációja</mark>. A Context Object-en keresztül lekérdezi az esemény részleteit és opcionálisan befolyásolja a keretrendszer működését.

**Dispatcher**: nyilvántartja a regisztrált Concrete Interceptorokat, és esemény bekövetkezésekor <mark>meghívja azok callback metódusait</mark>. Megfelel az Observer pattern Subject szerepének. Általában Singleton-ként valósul meg.

**Context Object**: tartalmaz információkat az eseményről, és <mark>hozzáférést biztosít a keretrendszer belső állapotához</mark>. Accessor metódusokkal olvasható, mutator metódusokkal módosítható a keretrendszer viselkedése.

**Application (Alkalmazás)**: példányosítja a Concrete Interceptort és <mark>regisztrálja a megfelelő Dispatcher-nél</mark>.

---

## Struktúra

<svg viewBox="0 0 720 380" xmlns="http://www.w3.org/2000/svg" style="width:100%;max-width:720px">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
    <marker id="arr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto">
      <path d="M2 1L8 5L2 9" fill="none" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-linecap="round"/>
    </marker>
  </defs>

  <!-- Application -->
  <rect x="20" y="20" width="140" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="90" y="37" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Application</text>
  <text x="90" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">regisztrál</text>

  <!-- Concrete Framework -->
  <rect x="280" y="20" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="360" y="37" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Concrete Framework</text>
  <text x="360" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">eseményt generál</text>

  <!-- Context Object -->
  <rect x="520" y="20" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="600" y="37" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Context Object</text>
  <text x="600" y="54" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">get/set framework state</text>

  <!-- Dispatcher -->
  <rect x="280" y="170" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="360" y="187" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Dispatcher</text>
  <text x="360" y="204" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">register / remove / dispatch</text>

  <!-- Interceptor interface -->
  <rect x="280" y="300" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="360" y="317" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Interceptor</text>
  <text x="360" y="334" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">callback() hook metódus</text>

  <!-- Concrete Interceptor -->
  <rect x="520" y="300" width="160" height="44" rx="8" fill="var(--color-surface-raised)" stroke="var(--color-border-secondary)" stroke-width="1.5"/>
  <text x="600" y="317" text-anchor="middle" font-size="12" font-weight="600" fill="var(--color-text-primary)">Concrete Interceptor</text>
  <text x="600" y="334" text-anchor="middle" font-size="10" fill="var(--color-text-secondary)">implementálja a callback-et</text>

  <!-- Arrows -->
  <!-- App -> Dispatcher (register) -->
  <line x1="160" y1="42" x2="280" y2="185" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="195" y="108" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">register</text>

  <!-- Framework -> Dispatcher (notify) -->
  <line x1="360" y1="64" x2="360" y2="170" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="385" y="122" font-size="9" fill="var(--color-text-secondary)">értesít</text>

  <!-- Framework -> Context Object -->
  <line x1="440" y1="42" x2="520" y2="42" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="480" y="36" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">létrehozza</text>

  <!-- Dispatcher -> Interceptor -->
  <line x1="360" y1="214" x2="360" y2="300" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="385" y="262" font-size="9" fill="var(--color-text-secondary)">meghívja</text>

  <!-- Interceptor -> Concrete Interceptor -->
  <line x1="440" y1="322" x2="520" y2="322" stroke="var(--color-text-secondary)" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr)"/>
  <text x="480" y="316" font-size="9" fill="var(--color-text-secondary)" text-anchor="middle">implementálja</text>

  <!-- Concrete Interceptor -> Context Object -->
  <line x1="600" y1="300" x2="600" y2="64" stroke="var(--color-text-secondary)" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="625" y="185" font-size="9" fill="var(--color-text-secondary)">használja</text>
</svg>

---

## Működés

A működés hat lépésből áll:

1. Az alkalmazás példányosít egy Concrete Interceptort és beregisztrálja a megfelelő Dispatcher-nél
2. A Concrete Framework érzékel egy eseményt, amelyet el kell fogni
3. A Concrete Framework példányosítja az eseményspecifikus Context Object-et, amely információkat tárol az eseményről és hozzáférést biztosít a keretrendszerhez
4. A Concrete Framework értesíti a megfelelő Dispatcher-t az eseményről, átadva neki a Context Object-et
5. A Dispatcher végighalad a regisztrált Concrete Interceptorokon és meghívja azok `callback()` hook metódusát a Context Object-tel
6. Minden Concrete Interceptor opcionálisan meghívhatja a Context Object mutator metódusait, befolyásolva a keretrendszer működését. Miután az összes callback visszatért, a keretrendszer folytatja megszokott működését.

<div class="callout tip" markdown="1">
**Tipp:** a Dispatcher a callback sorrendet különböző stratégiákkal kezelheti. FIFO (érkezési sorrendben), LIFO (fordított sorrendben), prioritás alapján, vagy Chain of Responsibility alapján, ahol az első kezelni tudó interceptor fogja az eseményt.
</div>

---

## Implementáció lépései

**1. A keretrendszer eseményeinek meghatározása**

Azonosítani kell, hogy a keretrendszer mely belső eseményeihez érdemes beavatkozási pontokat definiálni. Például egy ORB-ban ilyen esemény lehet egy kimenő kérés marshaling előtt és után, vagy egy bejövő kérés érkezésekor.

**2. Beavatkozási pontok és interception group-ok definiálása**

Az összetartozó beavatkozási pontokat <mark>interception group-okba kell szervezni</mark>. Például az OutRequest interception group tartalmazza a `PreMarshalOutRequest` és `PostMarshalOutRequest` beavatkozási pontokat.

**3. Context Object-ek definiálása**

Minden beavatkozási ponthoz meg kell határozni, milyen adatokat és hozzáférési metódusokat tartalmaz a Context Object. Két stratégia létezik a Context Object átadására:

- **Per-event**: a keretrendszer <mark>minden callback híváskor új Context Object-et hoz létre</mark>. Részletesebb információt ad, de nagyobb overhead-del jár.
- **Per-registration**: a Context Object <mark>egyszer kerül átadásra</mark>, amikor az interceptor regisztrál. Kevesebb overhead, de csak általános információt tartalmaz.

**4. Interceptor interfészek definiálása**

Minden interception group-hoz egy Interceptor interfészt kell definiálni, amely egy hook metódust tartalmaz minden beavatkozási ponthoz.

**5. Dispatcher-ek implementálása**

Minden Interceptor-hoz egy Dispatcher-t kell definiálni, amely regisztrációs interfészt nyújt az alkalmazásnak és callback interfészt a keretrendszernek. A Dispatcher általában Singleton-ként valósul meg. A regisztrált interceptorokat konténerben tárolja és iterálja végig.

**6. Concrete Interceptorok implementálása**

A Concrete Interceptorok az Interceptor interfészt implementálják alkalmazásspecifikus módon. A Context Object-en keresztül lekérdezik az esemény részleteit és opcionálisan módosítják a keretrendszer viselkedését.

<div class="callout trap" markdown="1">
**Csapda:** ha túl kevés Dispatcher-t definiálunk, a keretrendszer nem lesz elég rugalmas. Ha túl sokat, a rendszer bonyolulttá és nehezen kezelhetővé válik. A helyes egyensúly megtalálása a keretrendszer várható használati eseteitől függ.
</div>

---

## Ismert felhasználások

**Komponens alapú alkalmazásszerverek (EJB, CORBA Components, COM+)**: az Interceptor Proxy variánst használják. Egy "container" keretrendszer interceptor proxyt rendel minden komponenshez, amely elvégzi az infrastrukturális szolgáltatásokat (tranzakciókezelés, biztonság, perzisztencia) anélkül, hogy a komponens kódjának erről tudnia kellene.

**CORBA implementációk (TAO, Orbix, Visibroker)**: az Interceptor pattern segítségével az alkalmazásfejlesztők integrálhatnak saját szolgáltatásokat a kérések feldolgozásához. A CORBA Portable Interceptor szabvány egységesíti ezt.

**COM**: a fejlesztők implementálhatják az `IMarshal` interceptor interfészt egyedi marshaling funkcionalitás biztosítására.

**Webböngészők**: a Netscape Communicator és az Internet Explorer lehetővé teszi, hogy harmadik felek plugin-okat regisztráljanak adott médiatípusokhoz. Amikor egy adott típusú tartalom érkezik, a böngésző automatikusan meghívja a regisztrált plugin callback-jét.

**Angular HTTP Interceptors**: az Angular keretrendszerben az `HttpInterceptor` interfész implementálásával <mark>minden kimenő HTTP kérés és bejövő válasz elfogható, módosítható</mark>, például authentikációs token hozzáadásához vagy hibakezeléshez.

---

## Valós példa

Képzeld el, hogy egy online játékhoz REST API-t fejlesztesz. Minden API végponthoz szükséged van három dologra: hitelesítés (csak bejelentkezett felhasználó hívhat), naplózás (minden kérést logolni kell), és hibakezelés (a szerver hibáit egységesen kell visszaküldeni a kliensnek).

A naiv megközelítés: minden egyes végponton belül megírod ezt a három dolgot. Ha 50 végpontod van, 50 helyen van ugyanaz a kód. Ha a hitelesítési logika változik, 50 helyen kell módosítani.

Az Interceptor megoldása: minden kérés előtt a keretrendszer eléri a beavatkozási pontot, a Dispatcher meghívja a regisztrált interceptorokat sorban.

```java
// Hitelesítési interceptor — beregisztrálva a keretrendszerbe
public class AuthInterceptor implements HttpInterceptor {
    public void onRequest(RequestContext ctx) {
        String token = ctx.getHeader("Authorization");
        if (token == null || !isValid(token)) {
            ctx.abort(401, "Unauthorized"); // leállítja a kérést
        }
    }
}

// Naplózó interceptor
public class LoggingInterceptor implements HttpInterceptor {
    public void onRequest(RequestContext ctx) {
        System.out.println("Request: " + ctx.getMethod() + " " + ctx.getPath());
    }
}
```

Az API végpont kódja nem tud ezekről az interceptorokról. Ugyanúgy van megírva, mintha nem is léteznének. A keretrendszer intéz el mindent a háttérben.

Ugyanez a módszer működik:

**Naplózásnál**: minden adatbázis hívás előtt és után automatikusan lefut egy naplózó interceptor. Az adatbázis réteg kódját nem kell módosítani.

**Terheléselosztásnál**: egy CORBA interceptor elfogja a kimenő kérést és átirányítja a legkevésbé terhelt szerverre, anélkül hogy a kliens erről tudna.

**Biztonsági tokeneknél**: egy kliens oldali interceptor automatikusan hozzáadja az autentikációs tokent minden kimenő kéréshez. Egy szerver oldali interceptor ugyanezt ellenőrzi. Az alkalmazás kódja mindkét esetben érintetlen marad.

---

## Mérleg

| Előnyök | Hátrányok |
|---|---|
| Kiterjeszthetőség és rugalmasság | Komplex tervezés |
| Monitorozhatóság | Hibás interceptorok problémát okoznak |
| Újrafelhasználhatóság | Teljesítmény overhead |
| Aspektusorientált megközelítés támogatása | Interception kaszkád veszélye |

**Előnyök részletesen:**

**Kiterjeszthetőség és rugalmasság**: új szolgáltatások hozzáadhatók, módosíthatók és eltávolíthatók <mark>a keretrendszer módosítása nélkül</mark>.

**Monitorozhatóság**: az interceptorok és a Context Object-ek lehetővé teszik a <mark>keretrendszer dinamikus megfigyelését és vezérlését</mark>. Ez segíti az adminisztrációs eszközök, debuggerek és terheléselosztók fejlesztését.

**Újrafelhasználhatóság**: az <mark>interceptor kód elválik az alkalmazás kódjától</mark>, így ugyanaz a naplózó vagy hitelesítési interceptor több alkalmazásban is felhasználható.

**Aspektusorientált megközelítés támogatása**: az interceptorok olyan <mark>aspektusokként tekinthetők</mark>, amelyek átszövik az alkalmazást. A programozók az alkalmazáslogikára koncentrálhatnak az infrastrukturális szolgáltatások helyett.

**Hátrányok részletesen:**

**Komplex tervezés**: <mark>nehéz előre meghatározni</mark>, hogy a felhasználók milyen eseményeket akarnak majd elfogni. Túl kevés Dispatcher csökkenti a rugalmasságot, túl sok bonyolulttá teszi a rendszert.

**Hibás interceptorok problémát okoznak**: ha egy interceptor <mark>nem tér vissza, az egész alkalmazás megblokkolódhat</mark>. Ha hibát okoz, nehéz megakadályozni, hogy ez a keretrendszer többi részét is érintse, mivel azonos memóriaterületen futnak.

**Teljesítmény overhead**: a Dispatcher <mark>végigiterál az összes regisztrált interceptoron minden eseménynél</mark>, és Context Object-eket hoz létre, ami plusz memória és processzoridő.

**Interception kaszkád veszélye**: ha egy interceptor <mark>maga is eseményt vált ki</mark>, amelyre szintén van interceptor regisztrálva, kaszkád-szerű híváslánc alakulhat ki.

---

## Kapcsolata más patternekkel

Az **Observer pattern** (GoF) közvetlen rokona az Interceptor-nak. A Dispatcher <mark>a Subject szerepét, a Concrete Interceptor az Observer szerepét tölti be</mark>.

A **Component Configurator** segítségével az interceptorok <mark>dinamikusan tölthetők be futás közben</mark> DLL-ekből, ahelyett hogy statikusan regisztrálnák őket.

Az **Extension Interface** pattern alkalmazható arra, hogy <mark>minimalizáljuk a különböző interceptor típusok számát</mark>. Egy "fizikai" objektum több "logikai" interceptort is megvalósíthat különböző interfészeken keresztül.

---

<div class="callout exam" markdown="1">
**Tömör összefoglaló:**

Az Interceptor átlátszó módon egészít ki keretrendszereket különböző szolgáltatásokkal anélkül, hogy a keretrendszer kódját módosítani kellene. Hat szereplője van: Concrete Framework, Interceptor interfész, Concrete Interceptor, Dispatcher, Context Object és Application. A keretrendszer beavatkozási pontokat (interception points) definiál, a Dispatcher nyilvántartja a regisztrált interceptorokat, és esemény bekövetkezésekor meghívja azok callback metódusait. A Context Object hozzáférést biztosít a keretrendszer belső állapotához. Előnye a kiterjeszthetőség, monitorozhatóság és újrafelhasználhatóság. Hátránya a komplex tervezés, a hibás interceptorok veszélye és a teljesítmény overhead.
</div>