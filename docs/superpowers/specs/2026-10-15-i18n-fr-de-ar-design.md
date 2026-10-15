# ZeoPlant app — French, German, Arabic languages (design)

Date: 2026-10-15 · Status: approved by Nathan (approach, per-language numbers, push live)

## Goal
Make the ZeoPlant calculator (`index.html`, single-file PWA) usable in English, French, German and
Arabic from one page, with a switcher in the header, full right-to-left layout for Arabic, and
locale-aware number formatting. No new runtime files; offline and open-as-plain-file behavior unchanged.

## Non-goals
- The original `../ZeoPlant-Dashboard.html` is not touched.
- `manifest.webmanifest` stays English (brand name; the manifest format is single-language).
- No translation of the README.

## Architecture (single file)
- **Dictionary**: `const I18N = { en:{…}, fr:{…}, de:{…}, ar:{…} }` inside the existing `<script>`.
  Keys are short dotted slugs (`nav.inputs`, `in.area`, `kpi.save`, `note.season`, …). English is the
  reference; every other language must contain every key (checked by the verification script).
- **Static text**: each translatable element gets `data-i18n="key"`. Elements whose translation contains
  inline markup (`<b>`, `<br>`) use `data-i18n-html="key"` and are set with `innerHTML`; plain ones use
  `textContent`. Attribute translations (the two `aria-label`s, `<title>`) use `data-i18n-attr="title:key"`.
- **Dynamic text** built in `calc()` goes through `t(key, vars)` with `{name}` placeholders, e.g.
  `t('water.note', {pct, m3})`. Bar labels, KPI sub-lines, `mix-sub`, `roi-sub`, `roi-note`,
  `season-days`, "years"/"yr", "on … DH" are all keyed.
- **Numbers**: `fmt()` uses `LOCALE[lang]` = `en-US`, `fr-FR`, `de-DE`, `ar-MA`. `ar-MA` is forced to
  Latin digits via `numberingSystem:'latn'` so Moroccan convention (Western digits) holds everywhere.
  Units (m², m³, kg, L/m²·d, %, DH) are never translated or re-ordered except via the dictionary string.
- **Switcher**: `<div class="lang">` with four buttons EN · FR · DE · AR in the header `brand-meta`
  block; on phones (≤920px) the same group is repeated at the start of the sticky `.mnav` chip bar.
  Active button gets `aria-pressed="true"`.
- **Selection + persistence**: `localStorage['zeoplant.lang']`. If absent, pick the first of
  `navigator.languages` whose 2-letter prefix is one of the four, else `en`.
- **Apply**: `setLang(code)` sets `document.documentElement.lang`, `dir` (`rtl` for `ar`, `ltr` otherwise),
  swaps all `data-i18n*` nodes, stores the choice, then calls `calc()` so dynamic strings and number
  formats refresh.

## Right-to-left
Physical CSS properties that affect reading order are converted to logical ones:
`margin-left/right` → `margin-inline-start/end`, `border-left` → `border-inline-start`,
`padding-left` → `padding-inline-start`, `text-align:left/right` → `start/end`, the list bullet
`left:2px` → `inset-inline-start:2px`. The decorative header circle (`right:-80px`) also flips with
`inset-inline-end`. Flex rows need nothing (they follow `dir`). Numeric table cells keep `text-align:end`.
Inputs: `<input type=number>` keeps `direction:ltr` so digits and the unit badge read naturally.
Bars/donut are unaffected (SVG and centered columns).

## Translation rules
- Brand "ZeoPlant", "DH"/"MAD", and all units stay Latin in all languages.
- French and Arabic follow Moroccan professional usage (the model prices in dirhams); German is
  standard Hochdeutsch. IWR = "besoin en eau d'irrigation" / "Bewässerungsbedarf" / "الاحتياج المائي للري",
  keep the abbreviation "IWR" in parentheses the first time it appears in each language.
- Season rows keep month ranges (May–Sep → mai–sept, Mai–Sep, ماي–شتنبر, Moroccan month names).

## Error handling
- Missing key → `t()` falls back to English, then to the key itself; never throws, so the calculator
  always renders.
- Unknown stored language → treated as absent (falls back to detection).
- `localStorage` access is wrapped in try/catch as elsewhere in the file.

## Testing / verification
A Playwright script in the session scratchpad (not shipped) opens `index.html`, and for each of the four
languages: clicks the switcher, asserts `html[lang]`/`dir`, asserts no visible `data-i18n` element still
shows an English string when lang≠en (sampled set), asserts no raw key text (contains a dot and no
space) is visible, checks the dictionary key sets are identical across languages, checks that a sample
number (1234.5) renders as `1,234.5` / `1 234,5` / `1.234,5` / `1,234.5`, and screenshots the page at
390px width for a visual check (especially Arabic RTL).

## Release
Bump `CACHE` in `sw.js` to `zeoplant-v2`, commit, push to `main` (GitHub Pages, public), confirm the
live page serves the new build.
