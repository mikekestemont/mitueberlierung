# Preprocessing pipeline

How the raw Heurist database becomes the three analysis-ready files per language
(`{lang}_works.xlsx`, `{lang}_linkage.json`) that
`01-networks.ipynb` reads. `lang` is `french` or `german`.

## Pipeline overview

```
Heurist (LostMa DB)
       │  db.sync()  — cached locally in lostma.db, not committed
       ▼
00-export.ipynb
       │  steps 1-7: pull witnesses/parts/stories, resolve matière
       │             algorithmically from the storyverse hierarchy
       │  step 8:    split by language, rename to short snake_case,
       │             keep only the columns actually consumed
       ▼
{lang}_works.xlsx, {lang}_linkage.json
       │
       │  ── manual review, outside the notebook ──
       │  a colleague reviews the works and records corrections in
       │  {lang}_works_EdB.xlsx — a small hand-kept correction sheet
       ▼
{lang}_works_EdB.xlsx   (work_id, work, matiere_auto, override / is_heldenepik)
       │
       │  step 9: joined back onto {lang}_works.xlsx by work_id
       ▼
{lang}_works.xlsx  (matière final, is_heldenepik carried across)
       ▼
01-networks.ipynb  — reads {lang}_works.xlsx + {lang}_linkage.json
```

`{lang}_works.xlsx` is the single analysis-ready works file: step 9 folds the correction sheet
into it, so `01-networks.ipynb` reads it directly and never opens `{lang}_works_EdB.xlsx`.

## Step-by-step

1. **Pull source tables.** `db.witnesses(...)`, `db.parts(...)`, `db.stories(...)` fetch
   the raw Heurist tables for `fro`, `frm`, `gmh` combined. `witnesses` is restricted to
   columns with ≥5% fill across all three languages *pooled*, plus a fixed allow-list of
   columns needed downstream regardless of fill rate.
2. **Configure matière resolution.** `MATIERE_ALIASES` folds source labels onto the three
   levels of interest (e.g. `antiquity` → `Rome`); `STORYVERSE_MATTER_OVERRIDES` bridges
   storyverses not yet linked to a Matter in Heurist (e.g. the Carolingian cycle → France).
3. **Resolve matière from the storyverse network.** For each work, `matters_for` walks
   Story → Storyverse → cycle → … and collects every reachable *Matter of X* label.
4. **Attach manuscripts and parse dates.** Each witness's parts are mapped to their parent
   document; free-text date fields are parsed into numeric `start`/`end`/`mid` years.
5. **Build the three core tables** — `works`, `manuscripts`, `linkage` — deduplicated and
   still spanning all three languages combined.
6. **Verify matière coverage** — a sanity check, tallies the `matiere` distribution and
   lists works still `Unknown`.
7. **Diagnose unresolved cases** (optional) — traces one work's path through the storyverse
   chain to find where resolution dead-ends.
8. **Split by language group, simplify, and write.** `works`/`linkage` are split into `french`
   (Old + Middle French merged) and `german` (Middle High German), columns are renamed to short
   snake_case and cut to the explicit whitelist (see below), and the two files are written.
9. **Fold in the manual corrections.** If the hand-kept correction sheet
   `{lang}_works_EdB.xlsx` exists, it is joined onto `{lang}_works.xlsx` by `work_id`:
   `override` replaces `matiere` where filled, `is_heldenepik` is carried across.

## Fix 1 — a short, explicit output schema

Heurist's own column names (`TextTable_H-ID`, `TextTable_date_of_creation_mid`, …) are long,
inconsistent, and leak the source database's table layout into everything downstream. Worse, the
5%-fill threshold in step 1 is computed across `fro` + `frm` + `gmh` *pooled*, so columns that are
completely empty for one language still rode along into its export — `german_works.xlsx` shipped 16
entirely-empty `TextTable_*` columns (verse/rhyme/stanza type, author and adaptor fields, regional
writing style, …) out of 39.

Step 8 now renames to the short snake_case vocabulary the linkage file already used, and keeps an
explicit whitelist rather than guessing from fill rates:

| written column | from |
|---|---|
| `work_id` | `TextTable_H-ID` |
| `work` | `TextTable_preferred_name` |
| `language` | `TextTable_language_COLUMN` |
| `date_start` / `date_end` | `TextTable_date_of_creation_start` / `_end` (the midpoint is derived at load, not stored) |
| `matiere_source` | provenance of `matiere`, recorded during resolution |
| `matiere` | resolved in steps 2-3, corrected in step 9 |
| `is_heldenepik` | `is_Heldenepik`, carried across in step 9 (German only) |

That is exactly what `01-networks.ipynb` consumes, plus `work`/`language` so the table is readable
by eye. Works files went from 38/39 columns to 7-8; the manuscripts export was dropped entirely
(neither notebook read it, and shelfmarks reach the analysis through the linkage file).

**Dates:** `date_start`/`date_end`/`date_mid` in the works file are the work's date of
*composition*. The identically-named fields in the linkage file are the *manuscript copy's* date.
Same names, different tables — never mixed.

Nothing is lost irrecoverably: the full Heurist tables stay in `lostma.db`, so widening
`WORK_COLUMNS_KEPT` and re-running step 8 brings any dropped field back.

## Fix 2 — manual matière corrections are now actually applied

`{lang}_works_EdB.xlsx` is a small hand-kept **correction sheet** — `work_id`, `work`, the current
`matiere`, and one annotation column — not a copy of the export. Step 9 joins it onto
`{lang}_works.xlsx` by `work_id`:

- **French** — an `override` column with a corrected `matiere` value for 26 works where
  the algorithmic resolution (storyverse hierarchy, or its `matter_local` fallback) was
  wrong or landed on `Unknown`/`Other`. Examples: *Cligès* (Rome → Britain, its Arthurian
  frame outweighing the classical source material), *Antioche* (Other → Rome), five
  *Bueve de Hanstonne* branches and *Gui de Warwick* (`England` → `Other` — Bodel's scheme
  has no separate slot for "matière d'Angleterre", so these fall outside Britain/France/Rome
  rather than being folded into Britain). The full list of 26 works, before/after, is in
  the notebook output of step 9 and reproducible by re-running it.
- **German** — an `override` column plus an `is_heldenepik` boolean. The overrides cover 7 works
  the storyverse hierarchy cannot place: the Willehalm cycle (*Willehalm*, *Rennewart*, *Arabel*,
  *Willehalm von Orlens*, the prose *Willehalm*, and Elisabeth von Nassau-Saarbrücken's *Sibille*)
  → `France`, and *Alpharts Tod* → `Other`. These works have **no** storyverse-derived matière at
  all (`matieres` is empty) and a `matter_local` of `Other`, so the automatic resolution puts them
  in `Other`; as the German reflex of the Guillaume d'Orange cycle they belong to the matière de
  France. They were previously recorded by overwriting `matiere` in place, which meant any
  re-export silently reverted them — hence the explicit `override` column.

  The durable fix is upstream: once the *Wilhelm*, *Arabel* and *Rennewart* storyverses are linked
  to the Matter of France cycle in Heurist, these overrides can be dropped. A
  `STORYVERSE_MATTER_OVERRIDES` entry would cover the five works that sit in those storyverses, but
  not *Sibille*, which has no storyverse at all.
- **German** — the `is_heldenepik` boolean flagging works belonging to the Dietrich/Nibelungen
  cycle, *Kudrun*, *Ortnit/Wolfdietrich*, etc. This is **not** a `matiere` override — it's an
  independent genre flag used later, in `01-networks.ipynb`, to test whether *Heldendichtung*
  is a cohesive sub-block of the German `Other` category. It was already wired correctly.

**The French correction was never actually being applied.** `01-networks.ipynb`'s
`apply_edb_override` looked for a column literally named `EdB`, which never existed (the
real column is `override`) — so the function was a silent no-op. Even with the name fixed,
it wrote into a column called `matieres` (plural, the raw storyverse-derived set), while the
function that actually assigns matière to network nodes (`matiere_map`) reads `matiere`
(singular) — so the correction would still never have reached the graphs. Both bugs are
now moot: step 9 in `00-export.ipynb` merges the correction directly into `matiere` in the
data file itself, and `01-networks.ipynb`'s `load_all`/`assign_matiere` just reads that
column — no alias or override logic left in the analysis notebook at all.

Measured effect of the French overrides, running step 9 on the algorithmic export:
`France 146→147, Britain 59→62, Rome 47→44, Other 42→53, Unknown 13→7`, and `England 6→0` —
the England label disappears entirely, since Bodel's scheme has no such category and those
works are reassigned to `Other` or `Britain`. All 26 annotated works change category.

**This changes results that were already drafted.** All 26 corrected French works actually
move category (none were no-ops), and several move across the Britain/France/Rome boundary
used for the restricted network in Figures 2-3 and Tables 2-4 of the paper draft (e.g. five
works moving from `England`→`Britain`-via-old-alias to `Other`, i.e. now *excluded* from
that restricted network; *Cligès*, *Floriant et Florete*, *Perceforest* moving into
`Britain`; *Syracon*, *Tristan de Nanteuil* moving into `France`). **The French matière
distribution, homophily percentages, and any figure/table derived from it should be
recomputed and rechecked against the paper text before submission.** The German pipeline
was already correct and is unaffected in kind, though its numbers will shift slightly too
since `load_all` no longer re-derives `matiere` at all (previously harmless for German, but
worth a fresh run to confirm).

## Refreshing from Heurist

`00-export.ipynb` reads the local cache (`lostma.db` + `jbcamps_gestes_schema/`) by default and
does **not** re-download. Set `REFRESH_FROM_HEURIST = True` in the second cell to pull a fresh copy;
if no cache is present the notebook downloads regardless, saying so first.

Credentials are needed **only** when downloading. `login`/`password` reach `HeuristAPIConnection`
through `sync()` alone — `witnesses()`, `parts()` and `stories()` query `lostma.db` directly — so
on the default path the notebook constructs `LostmaDB("", "")` and never touches the credentials
file. Re-running the carpentry therefore needs no secrets at all.

Beware that a refresh can silently change matière assignments: that is how the Willehalm cycle
lost `France` (see Fix 2). After any refresh, compare `matiere_source` and the matière
distribution against the previous run before trusting downstream numbers.

## Credentials

`00-export.ipynb` no longer hardcodes the Heurist login/password. It reads them from a
local `credentials` file (`login = ...` / `pwd = ...`, one per line) that is excluded from
git via `.gitignore`. Copy `credentials.example` to `credentials` and fill in your own
values before running `db.sync()`.

## Final column reference

### `{lang}_works.xlsx` — one row per text (the analysis file)

| column | meaning |
|---|---|
| `work_id` | Heurist work ID — the join key used everywhere downstream |
| `work` | work title |
| `language` | Heurist language code (`fro`, `frm`, `gmh`) |
| `date_start`, `date_end`, `date_mid` | date of **composition**, parsed to numeric years |
| `matiere` | final matière — algorithmic resolution with manual corrections folded in |
| `matiere_source` | which stage decided it: `storyverse` (the *Matter of …* hierarchy), `local` (fallback to `Story_matter`), `override` (manual correction, step 9), or `none` (nothing resolved → `Unknown`) |
| `is_heldenepik` *(German only)* | genre flag for Heldendichtung, independent of `matiere` |

### `{lang}_works_EdB.xlsx` — the correction sheet

| column | meaning |
|---|---|
| `work_id` | join key back onto the works file |
| `work` | title, so a reviewer can see what they are annotating |
| `matiere_auto` | the matière as resolved automatically — what is being reviewed. Reference only; step 9 never reads it |
| `override` *(French)* | corrected matière, filled in only where the automatic value is wrong |
| `is_heldenepik` *(German)* | genre flag; carried across rather than overriding anything |

### `{lang}_linkage.json` — one row per witness (work ↔ manuscript)

| field | meaning |
|---|---|
| `witness_id`, `work_id`, `work` | witness and work identifiers/title |
| `manuscript_id`, `shelfmark` | the manuscript it's copied in |
| `siglum`, `status` | witness siglum and preservation status (e.g. `Fragmentary`) |
| `is_excerpt` | whether the witness is an excerpt |
| `date_start`, `date_end`, `date_mid` | the *manuscript copy's* date, not the work's date of composition — the two are never mixed (see `01-networks.ipynb`, cell 1) |
| `language` | Heurist language code |
