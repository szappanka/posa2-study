---
layout: default
title: Service Access & Configuration
---

# 2. fejezet: Service Access & Configuration Patterns

## Miről szól ez a kategória?

Minden szoftverrendszernek szüksége van arra, hogy **hozzáférjen valamilyen szolgáltatáshoz** (pl. OS, hálózat, adatbázis) és **konfigurálja azt** (pl. betöltse, lecserélje, bővítse). <mark>Ez triviálisnak hangzik, de komoly tervezési kihívásokat rejt.</mark>

A kategória <mark>négy patternből áll, amelyek együtt lefedik a szolgáltatás-hozzáférés és konfiguráció teljes problématerét</mark>.

---

## A négy kihívás, amit ez a kategória megold

| Kihívás | Pattern |
|---|---|
| Az OS API csúnya és platformfüggő | Wrapper Facade |
| A komponenseket futás közben kell betölteni/lecserélni | Component Configurator |
| A keretrendszert átláthatóan kell bővíteni | Interceptor |
| Egy komponens több szerepet tölt be, de a kliensek csak a sajátjukat lássák | Extension Interface |

---

## Miért fontos ez a vizsga szempontjából?

Ez az **alapozó fejezet**. A többi kategória (Event Handling, Synchronization, Concurrency) mind erre épül — <mark>szinte minden pattern belül Wrapper Facade-et használ az OS eléréséhez</mark>.

Ha ezt a fejezetet értjük, a többi sokkal könnyebb lesz.

<div class="callout tip" markdown="1">
**Tanulási tipp:** előbb a Wrapper Facade-et értsd meg alaposan — a többi pattern szinte mind ezt használja az OS eléréséhez, így ez a befektetés sokszorosan megtérül.
</div>

---

## A négy pattern egy mondatban

- **Wrapper Facade** — <mark>Becsomagolja a csúnya C API-kat OOP osztályokba</mark>
- **Component Configurator** — <mark>Komponenseket tölt be és cserél le futás közben, újraindítás nélkül</mark>
- **Interceptor** — <mark>Átlátható bővítési pontokat ad egy keretrendszerhez</mark>
- **Extension Interface** — <mark>Egy komponens több interfészt valósít meg anélkül, hogy a klienseket érintené</mark>

---

## Hogyan kapcsolódnak egymáshoz?

```
Wrapper Facade
    ↓ (ezt használja belül)
Component Configurator  →  Interceptor
                               ↓
                        Extension Interface
```

A **Wrapper Facade** az alap — <mark>erre épül minden</mark>.
A **Component Configurator** és **Interceptor** együtt dolgoznak: <mark>a Configurator betölti a komponenst, az Interceptor bővíti</mark>.
Az **Extension Interface** pedig <mark>megakadályozza, hogy a bővítés tönkretegye a meglévő klienseket</mark>.

---

## Tovább a patternekhez

1. [Wrapper Facade](./02-wrapper-facade.md)
2. [Component Configurator](./03-component-configurator.md)
3. [Interceptor](./04-interceptor.md)
4. [Extension Interface](./05-extension-interface.md)
