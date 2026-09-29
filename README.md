# Lx Search F4

A lookup tool for the insurance formulary. You can search about 1,106 medication records by name, brand, class, form, strength and dispensing authority (Gp / Sp / Con / MS). It is a single, self-contained `index.html` with no build step, no backend and no dependencies apart from Google Fonts.

## Entry points
- `index.html` is the app
- `index.html?q=…&auth=Con,Sp&class=…&form=…&sort=name:a,class:d&d=comfortable` restores the app state from the URL
- `index.html?selftest=1` runs the built-in check suite: recall on 2,000 corrupted queries, latency, build time, heap, UI checks

## What changed in F4 compared with F3
**Search engine**
- The tokenizer now indexes decimals (`0.5`, `12.5`) and dotted abbreviations (`E.D` becomes `ed`, `M.I.U` becomes `miu`). Queries like `entecavir 0.5`, `e.d timolol` and `m.i.u` now match.
- A stray letter before or after a dose is split off before matching (`entecavir0.5`, `h3`).
- A short last word is treated as a prefix you are still typing (`metformin x` finds the XR items, `amox 5` works).
- Dose typos fall back to the nearest listed strength (`imatinib 4000` finds Imatinib 400, `verapamil 24` finds 240, `cisplatin 05` finds 50). If none is close, the app shows all strengths for that drug.
- Doubled-letter typos resolve directly (`spiramyciin`).
- When no record matches every word, the app now tries dropping several different words, starting with the least useful ones and very short noise tokens. Before, it dropped only the single most common word.
- The top 40 results are reranked by similarity to the full drug name.
- Scoring runs on reused typed arrays instead of creating Maps for every query, so memory use per 1,000 queries went from about 5 MB to under 1 MB.
- Name hyphens are no longer read as NOT: `levo -thyroxin` works. `-tab` still excludes results.
- Self-test results: recall@1 went from 89.9% to 94.1%, recall@5 from 98.6% to 99.6%, and missed top-5 queries from 28 to 8. p95 latency stays under 1 ms and index build stays under 40 ms.

**Interface**
- New mobile header. Row 1 has the logo, name and a few icons. Row 2 has a full-width 46 px search field and a square filter button with a count badge.
- `type="search"` input brings up the search key on phone keyboards.
- Softer corner radius, pill-shaped chips, colored authority pills in the row sub-line, and the brand's magnifier logo.
- The disclaimer is smaller on phones and respects the safe area. Entry chips scroll sideways on phones.

**Bug fixes**
- The page now fills the viewport exactly and only the list scrolls. Before, the page itself could scroll and the virtual list's viewport was measured wrongly.
- On tablets (720–1079 px) the filter button was visible but the filter sheet was hidden with `display:none`. The sheet now works there.
- Downloads use object URLs instead of base64 data URLs, which is faster and fixes large exports failing on iOS.
- The browser address-bar color now follows the manual theme toggle.
- Pressing Esc now closes open menus and overlays before it clears the selection.
- Typing in the facet filter boxes no longer triggers global shortcuts such as `c` or `[`.
- Global arrow and space shortcuts no longer fire behind open sheets or modals.
- The "Browse" class chips are computed once instead of on every focus.

## Data
The records are embedded in the page as `RAW_DATA` (fields: Name, Conc, form, Authority, class). Recent queries, theme, density and a small ranking-personalisation map are stored in `localStorage`. The project uses no tables or server storage.

## Not done yet / next steps
- Two variants of Acetyl Salicylic Acid are still weak when spelled phonetically.
- A service worker could make the app work offline.
