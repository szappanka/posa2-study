# POSA2 Study Guide

Tanulási jegyzetek a *Pattern-Oriented Software Architecture, Volume 2* alapján,
statikus weboldalként (Jekyll + GitHub Pages).

**Élő oldal:** https://szappanka.github.io/posa2-study/

---

## Mi ez?

Egy Markdown-alapú dokumentációs oldal. A tananyag sima `.md` fájlokban van a
[`docs/`](docs/) mappában; a Jekyll ezekből generál weboldalt felső navbarral,
sötét/világos témával, oldalon belüli tartalomjegyzékkel és lapozóval.

```
posa2-study/
├── docs/                 # ← A TANANYAG ide kerül (egy .md = egy oldal)
│   ├── 01-service-access-overview.md
│   └── 02-wrapper-facade.md
├── index.md              # Kezdőlap (témakör-lista)
├── _config.yml           # Oldal-konfiguráció + NAVIGÁCIÓ (nav:)
├── _layouts/default.html # A HTML váz (navbar, TOC, lapozó, téma-váltó)
├── assets/css/style.css  # Minden stílus (témaszínek a tetején, --accent stb.)
├── assets/images/        # Logó (icon.svg) és nyíl (arrow.svg)
├── Gemfile               # Jekyll függőségek
├── serve.sh             # Lokális preview Dockerrel
└── quiz-app/             # Külön Vite/React projekt (NEM része a doksi-buildnek)
```

---

## Új tananyag hozzáadása

Két lépés: létrehozod a `.md`-t, majd beregisztrálod a navigációba.

### 1. Hozz létre egy fájlt a `docs/` mappában

Sorszámozott névvel, pl. `docs/03-component-configurator.md`. A fájl elejére
**frontmatter** kell (ez köti az oldalt a layouthoz):

```markdown
---
layout: default
title: Component Configurator
---

# Component Configurator

Ide jön a tartalom sima Markdownban…
```

> A `title` jelenik meg a böngészőfülön és a navbarban. A `layout: default`
> kötelező, enélkül nyers szöveg lenne.

### 2. Vedd fel a navigációba — `_config.yml`

A navbar és a lapozó (előző/következő) is a `_config.yml` `nav:` listájából
épül. A patternek a témakörük `children:` listája alá kerülnek:

```yaml
nav:
  - title: Kezdőlap
    url: /
  - title: Service Access & Configuration
    url: /docs/01-service-access-overview
    children:
      - title: Wrapper Facade
        url: /docs/02-wrapper-facade
      - title: Component Configurator      # ← új sor
        url: /docs/03-component-configurator
```

> ⚠️ Az `url`-ben **ne legyen `.md` kiterjesztés** — kezdjen `/docs/`-sal és
> egyezzen a fájlnévvel.

> A `_config.yml` változása **nem** töltődik újra futás közben — a lokális
> szervert ilyenkor újra kell indítani (lásd lentebb).

### 3. (Opcionális) Linkeld a kezdőlapról

A pattern a [`index.md`](index.md)-ben a témaköre alá, listaelemként kerül.

---

## Tartalmi konvenciók (amit egy generátor/AI tudjon)

A sima Markdownon túl ezek a projekt-specifikus elemek használhatók a `.md`-ben:

### Szövegkijelölő (highlight)

```markdown
Ez egy <mark>kiemelt rész</mark> a szövegben.
```

(`==...==` NEM működik — a Jekyll Kramdown-motorja nem ismeri; `<mark>` igen.)

### Tanuló-dobozok (callout)

Három típus. **Fontos:** a `markdown="1"` attribútum kell, hogy a doboz
belsejében is működjön a Markdown:

```markdown
<div class="callout tip" markdown="1">
**Tipp:** hasznos tanács, ami megkönnyíti a tanulást.
</div>

<div class="callout trap" markdown="1">
**Csapda:** gyakori hiba vagy félreértés, amire figyelni kell.
</div>

<div class="callout exam" markdown="1">
**Vizsgán mondd ezt:** a tömör, bemagolható összefoglaló.
</div>
```

| Osztály | Ikon | Mire való |
|---------|------|-----------|
| `tip`   | 💡   | tanulási tipp, hasznos megjegyzés |
| `trap`  | ⚠️   | csapda, gyakori hiba |
| `exam`  | 🎯   | vizsga-összefoglaló |

### Struktúra-diagramok (inline SVG)

A pattern-oldalak "Struktúra" szakaszában inline `<svg>` ábrák vannak. Hogy
sötét és világos témán is **látszódjanak**, a `fill`/`stroke` értékekben
**kizárólag ezt a négy CSS-változót** szabad használni (ezek a `style.css`-ben
a témaszínekre vannak leképezve, és témaváltáskor automatikusan átszíneződnek):

| Változó | Mire való |
|---------|-----------|
| `var(--color-surface-raised)`  | dobozok háttere |
| `var(--color-text-primary)`    | doboz-címek (erős szöveg) |
| `var(--color-text-secondary)`  | alcímek, nyíl-feliratok, vonalak |
| `var(--color-border-secondary)`| doboz-keretek |

> ⚠️ **Ne használj fix színt** (pl. `fill="#000"` vagy `fill="black"`) az
> SVG-ben — az a sötét témán fekete dobozt, fekete szöveget eredményez, és
> nem látszik. Mindig a fenti változókat add meg.

### Automatikus elemek (ezekkel nem kell foglalkozni)

- **Tartalomjegyzék** — a `##` (h2) címekből épül automatikusan, jobb oldalt.
  Csak annyi a dolgod, hogy `##`-okkal tagold a tartalmat.
- **Előző/Következő lapozó** — a `_config.yml` nav-sorrendjéből generálódik.
- **Kódblokk, táblázat, idézet** — sima Markdown, automatikusan stílusozva.

---

## Lokális futtatás (preview)

Docker kell hozzá (nem kell se Ruby, se Jekyll a gépre telepíteni).

```bash
./serve.sh
```

Majd nyisd meg: **http://localhost:4000/posa2-study/**

- Leállítás: `Ctrl+C`
- Tartalom / CSS / layout szerkesztése után: elég a böngészőt frissíteni.
- **`_config.yml` szerkesztése után:** `Ctrl+C`, majd `./serve.sh` újra.

<details>
<summary>Mi van a <code>serve.sh</code>-ban?</summary>

```bash
docker run --rm -v "$PWD":/srv/jekyll -p 4000:4000 jekyll/jekyll:4.2.2 \
  sh -c "bundle install && jekyll serve --host 0.0.0.0 --watch --baseurl /posa2-study"
```
</details>

---

## Publikálás

A `main` branchre pusholt változások automatikusan élesednek a GitHub Pages-en
(pár perc alatt). Külön build-lépés nincs.

```bash
git add .
git commit -m "Add: <pattern neve>"
git push
```

> **Megjegyzés a buildkörnyezetről:** az éles oldalt a GitHub Pages
> *klasszikus* buildje generálja (a `main` branchből, GitHub Actions nélkül).
> Ez a saját, rögzített Jekyll-csomagjával épít, és a `Gemfile`-t figyelmen
> kívül hagyja — a `Gemfile` csak a lokális (Dockeres) preview-hoz kell.
> A `jekyll-relative-links` plugin a `_config.yml > plugins:` alatt van
> felsorolva, ezért élesben is fut. Ha új plugint vennél fel, előbb ellenőrizd,
> hogy a GitHub Pages whitelistjén szerepel-e, különben csak lokálisan hatna.

---

## Témaszínek

A színek a [`assets/css/style.css`](assets/css/style.css) tetején, CSS-változókban
vannak (sötét és világos témához külön). A fő kék az `--accent`
(`#58a6ff` sötét / `#0969da` világos). Egy változó átírása az egész oldalon
végigfut.

---

## A `quiz-app/` mappáról

Külön Vite + React projekt, **nem része** a doksi-buildnek (a `_config.yml`
`exclude` kizárja). Saját `npm run dev` / `npm run build` kell hozzá — később
kerül az oldalhoz.
