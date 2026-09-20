# Preprocessing pipeline

How the raw Heurist database becomes the two analysis-ready files per language,
`{lang}_works.xlsx` and `{lang}_linkage.json`, that `01-networks.ipynb` reads. `lang` is
`french` or `german`.

```
Heurist (LostMa DB)
       │  db.sync()  — cached locally in lostma.db, not committed
       ▼
00-export.ipynb
       │  steps 1-7: pull witnesses/parts/stories, resolve matière
       │             from the storyverse hierarchy
       │  step 8:    split by language, rename to short snake_case,
       │             keep only the columns actually consumed
       ▼
{lang}_works.xlsx, {lang}_linkage.json
       │
       │  ── manual review, outside the notebook ──
       │  corrections recorded in {lang}_works_EdB.xlsx
       ▼
       │  step 9: joined back onto {lang}_works.xlsx by work_id
       ▼
{lang}_works.xlsx  (matière final, is_heldenepik carried across)
       ▼
01-networks.ipynb
```

`00-export.ipynb` owns every mapping, alias, correction and normalisation; `01-networks.ipynb`
only reads. Nothing is remapped at analysis time.

## Steps

1. **Pull source tables.** `db.witnesses(...)`, `db.parts(...)`, `db.stories(...)` fetch the raw
   Heurist tables for `fro`, `frm`, `gmh` combined. `witnesses` is restricted to columns with ≥5%
   fill across the three languages pooled, plus a fixed allow-list needed downstream.
2. **Configure matière resolution.** `MATIERE_ALIASES` folds the database's labels onto the
   analysis categories: `antiquity` → `Rome`, `england` → `Other` (Bodel's scheme has no *matière
   d'Angleterre*; folding these insular-hero romances into Britain would misrepresent them as
   Arthurian). `STORYVERSE_MATTER_OVERRIDES` bridges storyverses not yet linked to a Matter in
   Heurist (the Carolingian cycle → France).
3. **Resolve matière from the storyverse network.** For each work, `matters_for` walks
   Story → Storyverse → cycle → … and collects every reachable *Matter of X* label. Where the
   hierarchy yields nothing, the story's own `Story_matter` field is used.
4. **Attach manuscripts and parse dates.** Each witness's parts are mapped to their parent
   document; free-text date fields are parsed into numeric `start`/`end` years.
5. **Build the core tables** — `works`, `manuscripts`, `linkage` — deduplicated, all languages.
6. **Verify matière coverage** — tally the distribution, list works still `Unknown`.
7. **Diagnose unresolved cases** (optional) — trace one work's path through the storyverse chain.
8. **Split by language, simplify, write.** `french` = Old + Middle French merged, `german` =
   Middle High German. Columns are renamed to short snake_case and cut to the explicit whitelist
   below. Titles are stripped of the quotes Heurist wraps many German titles in; witness status
   is normalised to title case.
9. **Fold in the manual corrections.** `{lang}_works_EdB.xlsx` is joined onto
   `{lang}_works.xlsx` by `work_id`: `override` replaces `matiere` where filled (and sets
   `matiere_source = override`), `is_heldenepik` is carried across. A work missing from the
   sheet gets `is_heldenepik = False` with a warning — left as `NaN` it would read as `True`.

## Where `is_heldenepik` comes from

It is not in LostMa. No Heurist field records Heldendichtung; the flag is a scholarly judgement
(Handschriftencensus "Helden- und Dietrichepik", Lienert 2015) that exists only in
`data/german_works_EdB.xlsx` and is carried onto `german_works.xlsx` by step 9. The correction
sheet is the only copy. It covers all 143 German works, 22 of them flagged.

## Refreshing from Heurist

`00-export.ipynb` reads the local cache (`lostma.db` + `jbcamps_gestes_schema/`) by default and
does not re-download. Set `REFRESH_FROM_HEURIST = True` in the second cell to pull a fresh copy;
without a cache the notebook downloads regardless. Only a download needs the `credentials` file
(`login = …` / `pwd = …`, excluded from git); the read path never touches it. The download date
is written to `data/heurist_sync.txt` — the current snapshot is **26 August 2026, 14:33 UTC**.

A refresh can silently change matière assignments (that is how the Willehalm cycle once lost
`France`). After any refresh, compare `matiere_source` and the matière distribution against the
previous run before trusting downstream numbers. The durable fix for the Willehalm cycle is
upstream: once the *Wilhelm*, *Arabel* and *Rennewart* storyverses are linked to the Matter of
France in Heurist, those overrides can be dropped.

## Column reference

### `{lang}_works.xlsx` — one row per work

| column | meaning |
|---|---|
| `work_id` | Heurist work ID — the join key used everywhere downstream |
| `work` | work title |
| `language` | Heurist language code (`fro`, `frm`, `gmh`) |
| `date_start`, `date_end` | date of **composition**; the midpoint is derived at load |
| `matiere` | final matière — automatic resolution with manual corrections folded in |
| `matiere_source` | which stage decided it: `storyverse` (the *Matter of …* hierarchy), `local` (fallback to `Story_matter`), `override` (manual correction) |
| `is_heldenepik` *(German only)* | Heldendichtung flag, independent of `matiere` |

### `{lang}_works_EdB.xlsx` — the correction sheet

| column | meaning |
|---|---|
| `work_id`, `work` | join key and title |
| `matiere_auto` | the matière as resolved automatically — reference only, never read by step 9 |
| `override` | corrected matière, filled only where the automatic value is wrong |
| `is_heldenepik` *(German)* | Heldendichtung flag; carried across, overrides nothing |

### `{lang}_linkage.json` — one row per witness (work ↔ manuscript)

| field | meaning |
|---|---|
| `witness_id`, `work_id`, `work` | witness and work identifiers/title |
| `manuscript_id`, `shelfmark` | the manuscript it is copied in |
| `status` | preservation status of the *witness* (`Complete`, `Defective`, `Fragmentary`, `Citation`, `Lost`), not of the manuscript: a codex can hold a complete copy of one text and a fragment of another |
| `date_start`, `date_end` | the *manuscript copy's* date, not the work's composition — same names as in the works file, different tables, never mixed |
