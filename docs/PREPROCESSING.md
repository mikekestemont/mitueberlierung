# Preprocessing pipeline

How the raw Heurist database becomes the three analysis-ready files per language
(`{lang}_works_EdB.xlsx`, `{lang}_manuscripts.xlsx`, `{lang}_linkage.json`) that
`01-networks.ipynb` reads. `lang` is `french` or `german`.

## Pipeline overview

```
Heurist (LostMa DB)
       │  db.sync()  — cached locally in lostma.db, not committed
       ▼
00-export.ipynb
       │  steps 1-7: pull witnesses/parts/stories, resolve matière
       │             algorithmically from the storyverse hierarchy
       ▼
{lang}_works.xlsx, {lang}_manuscripts.xlsx, {lang}_linkage.json   (step 8, pruned)
       │
       │  ── manual review, outside the notebook ──
       │  a colleague opens {lang}_works.xlsx in Excel, adds an
       │  annotation column, saves as {lang}_works_EdB.xlsx
       ▼
{lang}_works_EdB.xlsx  (hand-annotated copy)
       │
       │  00-export.ipynb, step 9: re-run the notebook (or just its
       │  last cell) to fold the annotation into `matiere`
       ▼
{lang}_works_EdB.xlsx  (matière now final)
       ▼
01-networks.ipynb  — reads matiere directly, no further correction logic
```

`{lang}_works_EdB.xlsx` is therefore the file `01-networks.ipynb` should always read for
`works` — never the plain `{lang}_works.xlsx`, which only carries the algorithmic,
pre-manual-review `matiere`.

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
8. **Split by language group, prune, and write.** `works`/`manuscripts`/`linkage` are split
   into `french` (Old + Middle French merged) and `german` (Middle High German), each split
   is pruned of columns that don't survive it (see below), and the three files are written.
9. **Fold in manual matière corrections.** If a hand-reviewed `{lang}_works_EdB.xlsx`
   already exists, its annotation column is merged into `matiere` and the file is re-saved.

## Fix 1 — column pruning is now per language, not pooled

Step 1's 5%-fill threshold is computed once, across `fro` + `frm` + `gmh` pooled. A column
that clears that pooled bar can still be completely empty once the data is split — French
metadata was propping up columns that carry no German content, and vice versa. Before this
cleanup, the exported `german_works_EdB.xlsx` shipped **16 entirely-empty columns**
(`TextTable_length`, `verse_type`, `rhyme_type`, `stanza_type`, `is_written_by`,
`is_adapted_by`, `author_freetext`, `regional_writing_style`, `scripta_freetext`,
`date_of_creation_source`, `date_freetext`, `in_stemma`, `place_of_creation` — all 0.0%
filled for German) plus several near-empty ones, out of 39 total.

`prune_sparse_columns` (step 8, in `00-export.ipynb`) re-checks fill rate *within* each
language's own split and drops anything under 5% that isn't one of the columns every
downstream step depends on (identifiers, dates, language, form, and the three matière
columns). Column counts before → after:

| file | before | after | dropped |
|---|---:|---:|---:|
| `french_works.xlsx` / `french_works_EdB.xlsx` | 38 / 39 | 32 / 33 | 6 |
| `german_works_EdB.xlsx` | 39 | 20 | 19 |
| `french_manuscripts.xlsx` | 15 | 15 | 0 |
| `german_manuscripts.xlsx` | 15 | 11 | 4 |

Nothing is lost silently: `prune_sparse_columns` prints exactly what it drops and why, and
the raw Heurist tables (via `db.sync()` / `lostma.db`) always have the full column set if a
dropped field turns out to matter later — re-run without the pruning step, or lower
`MIN_FILL`, to get it back.

## Fix 2 — manual matière corrections are now actually applied

Two `_works_EdB.xlsx` files carry a hand-review pass by a colleague, on top of the
algorithmic `matiere`:

- **French** — an `override` column with a corrected `matiere` value for 26 works where
  the algorithmic resolution (storyverse hierarchy, or its `matter_local` fallback) was
  wrong or landed on `Unknown`/`Other`. Examples: *Cligès* (Rome → Britain, its Arthurian
  frame outweighing the classical source material), *Antioche* (Other → Rome), five
  *Bueve de Hanstonne* branches and *Gui de Warwick* (`England` → `Other` — Bodel's scheme
  has no separate slot for "matière d'Angleterre", so these fall outside Britain/France/Rome
  rather than being folded into Britain). The full list of 26 works, before/after, is in
  the notebook output of step 9 and reproducible by re-running it.
- **German** — an `is_Heldenepik` boolean flagging works belonging to the Dietrich/Nibelungen
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

## Credentials

`00-export.ipynb` no longer hardcodes the Heurist login/password. It reads them from a
local `credentials` file (`login = ...` / `pwd = ...`, one per line) that is excluded from
git via `.gitignore`. Copy `credentials.example` to `credentials` and fill in your own
values before running `db.sync()`.

## Final column reference

### `{lang}_works_EdB.xlsx` — one row per text

| column | meaning |
|---|---|
| `TextTable_H-ID` | Heurist work ID — the join key used everywhere downstream |
| `TextTable_preferred_name` | work title |
| `TextTable_language_COLUMN` | Heurist language code (`fro`, `frm`, `gmh`) |
| `TextTable_literary_form` | verse / prose / mixed, etc. |
| `TextTable_is_hypothetical` | flags reconstructed/hypothetical works |
| `TextTable_is_derived_from H-ID` / `Name` | source work, if adapted/derived |
| `TextTable_nature_of_derivations` | free text on how it derives from its source |
| `TextTable_tradition_status` | manuscript-tradition status |
| `TextTable_date_of_creation*` | free-text date plus parsed `_certainty`, `_source` (French only, ≥5% filled there), `_start`, `_end`, `_mid` numeric years |
| `matieres` | raw storyverse-derived matière set (list), before aliasing/fallback |
| `matter_local` | raw local `Story_matter` field, kept for comparison |
| `matiere` | **final, analysis-ready matière** — algorithmic resolution with manual corrections folded in (see Fix 2) |
| `storyverses`, `stories` | the storyverse(s)/stor(y/ies) a work belongs to |
| `override` *(French only)* | the manual correction column a colleague filled in; preserved for provenance after being folded into `matiere` |
| `is_Heldenepik` *(German only)* | manual genre flag, independent of `matiere` (see Fix 2) |

French additionally keeps `TextTable_length`, `length_freetext`, `verse_type`,
`rhyme_type`, `stanza_type`, `regional_writing_style H-ID`/`Name`, `scripta_freetext`,
`date_of_creation_source`, `date_freetext`, `is_written_by H-ID`/`Name`,
`author_freetext` — all above 5% fill for French but pruned for German, where they
were empty.

### `{lang}_manuscripts.xlsx` — one row per document

| column | meaning |
|---|---|
| `DocumentTable_H-ID` | Heurist manuscript ID — the join key |
| `DocumentTable_current_shelfmark` | current shelfmark |
| `DocumentTable_location_known` / `collection_of_fragments` | booleans |
| `DocumentTable_old_shelfmark` | superseded shelfmark, where recorded |
| `Repository_*` | holding repository: name, VIAF, city |

French additionally keeps `DocumentTable_collection`, `digitization_freetext`,
`Digitization_H-ID`, `Digitization_URI` (all 0% filled for German, pruned there).

### `{lang}_linkage.json` — one row per witness (work ↔ manuscript)

| field | meaning |
|---|---|
| `witness_id`, `work_id`, `work` | witness and work identifiers/title |
| `manuscript_id`, `shelfmark` | the manuscript it's copied in |
| `siglum`, `status` | witness siglum and preservation status (e.g. `Fragmentary`) |
| `is_excerpt` | whether the witness is an excerpt |
| `date_start`, `date_end`, `date_mid` | the *manuscript copy's* date, not the work's date of composition — the two are never mixed (see `01-networks.ipynb`, cell 1) |
| `language` | Heurist language code |
