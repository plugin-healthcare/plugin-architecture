# AGENTS.md

Summary of all changes and customizations made to this Quarto book project by AI agents.

---

## Project overview

This is a Quarto book using the **arc42 Dutch architecture documentation template** (v8.2-NL).
It renders to **HTML** (Typst/PDF config is currently commented out). The architecture is
modeled with [LikeC4](https://likec4.dev/) and interactive diagrams are embedded in the HTML
output via the `<likec4-view>` web component.

The brand font is **Figtree**, defined in `_brand.yml` with `source: google`.

The project includes:
- A **test suite** (vitest) that validates the LikeC4 model
- **CI/CD** via GitHub Actions that builds and deploys the Quarto site + LikeC4 SPA to GitHub Pages

---

## Changes made

### 1. Project structure: arc42 Dutch chapter files

Replaced the default Quarto book stubs with 14 Dutch arc42 chapter files sourced from
`arc42-template-NL.md`:

| File | Arc42 section |
|------|--------------|
| `index.qmd` | Voorwoord (preamble) |
| `01-inleiding-doelen.qmd` | 1. Inleiding en doelstellingen |
| `02-architectuur-kaders.qmd` | 2. Architectuur kaders |
| `03-context-systeem-scope.qmd` | 3. Context en systeemscope |
| `04-oplossing-strategie.qmd` | 4. Oplossing strategie |
| `05-bouwstenen-view.qmd` | 5. Bouwstenen view |
| `06-runtime-view.qmd` | 6. Runtime view |
| `07-deployment-view.qmd` | 7. Deployment view |
| `08-crosscutting-concepten.qmd` | 8. Crosscutting concepten |
| `09-architectuur-beslissingen.qmd` | 9. Architectuurbeslissingen |
| `10-kwaliteit-eisen.qmd` | 10. Kwaliteitseisen |
| `11-risicos-technische-schuld.qmd` | 11. Risico's en technische schuld |
| `12-woordenlijst.qmd` | 12. Woordenlijst |
| `references.qmd` | Referenties |
| `architectuur-dataspaces.qmd` | Appendix A: Architectuur van data spaces |

---

### 2. `_quarto.yml` — book metadata and format configuration

- Set `book.title`: `"PLUGIN architectuur"`
- Set `book.author`: `["Daniel Kapitan", "Madou Derksen", "Yannick Vinkesteijn", "Marlou van der Sande"]`
- Set `book.date`: `"01/04/2026"`
- Updated `book.chapters` to list all 14 arc42 chapter files
- Added `book.appendices`: `[architectuur-dataspaces.qmd]`
- Added `bibliography: plugin.bib`
- Added `brand: _brand.yml`
- **HTML format settings:**
  - `theme: [cosmo, brand]`
  - `lang: nl`
  - `include-in-header` — injects a `<script module>` tag that loads `likec4-views.js`
    from the co-deployed LikeC4 SPA at `/c4/`:

    ```yaml
    include-in-header:
      - text: |
          <script module src="https://plugin-healthcare.github.io/plugin-architecture/c4/likec4-views.js"></script>
    ```

    This script registers the `<likec4-view>` custom web component globally for all
    HTML pages in the book. The URL points to the GitHub Pages deployment of the
    LikeC4 SPA (built by `likec4 build --base "${BASE}/c4" --output _site/c4` in CI).

- **Typst/PDF format settings (currently commented out):**
  - `font-paths: fonts`
  - `mainfont: Figtree`
  - `template-partials: [typst-show.typ]`

---

### 3. `fonts/` directory — embedded Figtree font files

Downloaded all 14 Figtree `.ttf` variants from Google Fonts (via GitHub) into `arc42/fonts/`
so the font is self-contained in the project (used when Typst output is enabled).

---

### 4. `typst-show.typ` — custom Typst template partial (currently unused)

This file exists for when Typst/PDF output is re-enabled. It places font rules *before*
`#show: book.with(...)` so they reach the cover page and TOC. See git history for
detailed documentation of why this is necessary.

---

### 5. LikeC4 web component embeds

LikeC4 interactive diagrams are embedded in the HTML output using the `<likec4-view>`
custom web component (registered by `likec4-views.js` injected via `include-in-header`).

Each diagram is placed in a raw HTML block (`{=html}`) so Quarto passes it through
unchanged:

````markdown
```{=html}
<likec4-view
   view-id="<id>"
   dynamic-variant="sequence">
</likec4-view>
```
````

#### Attributes used

| Attribute | Description |
|-----------|-------------|
| `view-id` | Matches the named view ID from `src/model.views.c4` or auto-generated implicit IDs |
| `dynamic-variant` | How dynamic views render: `diagram` or `sequence` (default `diagram`) |

#### Why `browser="true"` is NOT used

The `<likec4-view>` web component parses its attributes via a Zod schema. The `browser`
attribute is defined as a boolean with a default of `true`. When `browser="true"` is set
in HTML, `getAttribute('browser')` returns the **string** `"true"`, which fails Zod's
boolean validation. When validation fails, the **entire props object** falls back to
defaults (`{viewId: 'index'}`), causing the default view to render regardless of the
specified `view-id`.

Since `browser` defaults to `true` in the schema, simply omitting the attribute gives
the correct behavior.

#### Embedded views

**`05-bouwstenen-view.qmd`** (9 embeds):

| `view-id` | Level | Description |
|-----------|-------|-------------|
| `index` | 1 | Top-level PLUGIN system overview |
| `datastation` | 2 | Datastation subsystem |
| `__plugin_datastation_plugin-lake` | 3 | PLUGIN-Lake (implicit view) |
| `__plugin_datastation_plugin-rosetta` | 3 | PLUGIN-Rosetta (implicit view) |
| `__plugin_datastation_vantage6-node` | 3 | vantage6 node (implicit view) |
| `ph` | 2 | Processing Hub subsystem |
| `__plugin_ph_plugin-ml` | 3 | PLUGIN-ML (implicit view) |
| `__plugin_ph_plugin-hub` | 3 | PLUGIN-Hub (implicit view) |
| `__plugin_ph_some-server` | 3 | PLUGIN-Analytics (implicit view) |

**`06-runtime-view.qmd`** (1 embed):

| `view-id` | Type | Description |
|-----------|------|-------------|
| `federated-learning` | dynamic view | Federated learning sequence diagram |

#### View ID conventions

- Explicitly named views (e.g. `view index`, `view ph`) use their declared ID directly
- Implicit/auto-generated views use a double-underscore prefix followed by the element
  hierarchy path separated by underscores: `__<system>_<subsystem>_<component>`
- These implicit IDs are generated by LikeC4 for elements that don't have an explicit
  `view of <element>` declaration and may change if the model hierarchy is restructured

#### Integration flow

```
src/model.views.c4          (LikeC4 source: named views)
        ↓  likec4 build
_site/c4/likec4-views.js    (web component bundle, deployed at /c4/)
        ↓  <script module> in <head> (injected by _quarto.yml include-in-header)
*.html                      (Quarto HTML output: <likec4-view> tags resolved at runtime)
```

**Typst/PDF output:** The `{=html}` blocks are silently ignored by the Typst renderer,
so the PDF output simply omits the interactive diagrams.

---

### 6. LikeC4 source model (`src/`)

The C4 architecture model is defined in four source files:

| File | Purpose |
|------|---------|
| `_spec.c4` | Specification: element kinds (system, actor, ui, container, component, datastore, informationmodel, group) with custom colors and shapes |
| `model.c4` | Main model: actors, PLUGIN system with datastation and processing hub subsystems, all components and relationships |
| `model.views.c4` | View definitions: `index`, `ph`, `datastation`, `dagster`, and `federated-learning` (dynamic view) |
| `dagster.c4` | Separate model documenting Dagster's conceptual architecture (assets, jobs, sensors, schedules, etc.) |

Custom colors defined in `_spec.c4`:
- `plugin-dark-blue` (#053c5b) — systems
- `plugin-teal` (#0588a6) — UIs
- `plugin-light-blue` (#bbe2ee) — containers
- `plugin-orange` (#f28729) — actors
- `plugin-light-grey` (#F2F2F2) — future/placeholder components

---

### 7. Test infrastructure

The project uses **vitest** to validate the LikeC4 model:

- `vitest.config.ts` — vitest configuration
- `test/globalSetup.ts` — global setup for tests
- `test/likec4-model.ts` — auto-generated TypeScript model (via `likec4 gen model`)
- `test/validate-model.spec.ts` — model validation tests

Run tests with `bun run test` (or `vitest run`). The `postinstall` script regenerates
the TypeScript model automatically.

---

### 8. CI/CD (`.github/workflows/`)

**`pages.yml`** — main deployment workflow (triggered on push to `main`):
1. **validate** — runs `bun run test` to validate the LikeC4 model
2. **build-pages** — renders Quarto book to `_site/`, then builds LikeC4 SPA to `_site/c4/`
3. **deploy-pages** — deploys combined `_site/` to GitHub Pages

**`on-redeploy.yml`** — triggers redeployment (e.g. when upstream dependencies change)

---

## Known issues

### Lightbox backdrop too transparent

When clicking on a LikeC4 diagram to open the fullscreen overlay (lightbox), the backdrop
is only 60% opaque. This is hardcoded in the LikeC4 bundle CSS:

```css
::backdrop {
  background: color-mix(in oklab, var(--colors-likec4-overlay-backdrop) 60%, transparent);
}
```

The `::backdrop` pseudo-element for `<dialog>` elements inside Shadow DOM **cannot be
styled from outside**:
- CSS custom properties do NOT inherit into `::backdrop` (it inherits from initial values only)
- `adoptedStyleSheets` appended to the shadow root do NOT affect `::backdrop` in practice
- Appending `<style>` elements to the shadow root also does not work

**Resolution:** Requires an upstream fix in LikeC4 (expose a configurable CSS property)
or post-processing the bundle during CI to change the 60% value.

### Implicit view IDs are fragile

Auto-generated view IDs like `__plugin_datastation_plugin-lake` depend on LikeC4's
internal naming convention derived from the model hierarchy. If elements are renamed
or restructured, these IDs will change and the embedded views will fall back to the
default view. Consider defining explicit named views in `model.views.c4` for all
embedded diagrams.

---

## File inventory

```
plugin-architecture/
├── .github/workflows/
│   ├── pages.yml                # CI: validate + build + deploy to GitHub Pages
│   └── on-redeploy.yml          # CI: trigger redeployment
├── src/
│   ├── _spec.c4                 # LikeC4 specification (element kinds, colors, shapes)
│   ├── model.c4                 # LikeC4 main model (actors, systems, components)
│   ├── model.views.c4           # LikeC4 view definitions
│   └── dagster.c4               # LikeC4 model of Dagster concepts
├── test/
│   ├── globalSetup.ts           # Vitest global setup
│   ├── likec4-model.ts          # Auto-generated TypeScript model
│   └── validate-model.spec.ts   # Model validation tests
├── arc42/
│   ├── _quarto.yml              # Quarto book config (title, authors, chapters, formats)
│   ├── _brand.yml               # Brand config (Figtree font)
│   ├── typst-show.typ           # Custom Typst template partial (unused while PDF disabled)
│   ├── fonts/                   # 14 Figtree .ttf files
│   ├── images/                  # Static images used in chapters
│   ├── index.qmd                # Voorwoord
│   ├── 01-inleiding-doelen.qmd
│   ├── 02-architectuur-kaders.qmd
│   ├── 03-context-systeem-scope.qmd
│   ├── 04-oplossing-strategie.qmd
│   ├── 05-bouwstenen-view.qmd   # Embeds 9 <likec4-view> web components
│   ├── 06-runtime-view.qmd      # Embeds 1 <likec4-view> (federated-learning)
│   ├── 07-deployment-view.qmd
│   ├── 08-crosscutting-concepten.qmd
│   ├── 09-architectuur-beslissingen.qmd
│   ├── 10-kwaliteit-eisen.qmd
│   ├── 11-risicos-technische-schuld.qmd
│   ├── 12-woordenlijst.qmd
│   ├── references.qmd
│   ├── architectuur-dataspaces.qmd  # Appendix A
│   ├── scratchpad.qmd           # Working notes (not in book chapters)
│   ├── plugin.bib               # Bibliography
│   └── AGENTS.md                # This file
├── likec4.config.ts             # LikeC4 project config
├── package.json                 # Dependencies (likec4, vitest, typescript)
├── pnpm-lock.yaml
├── tsconfig.json
└── vitest.config.ts
```
