# mitueberlierung

Co-transmission of French and German chivalric romance: does Jean Bodel's *matière*
scheme (Britain / France / Rome) structure which works were copied together in the
same manuscripts? Research paper + the data pipeline and network analysis behind it.

## Layout

```
00-export.ipynb        pull from the LostMa Heurist database, resolve matière,
                        write the three per-language data files (see docs/)
01-networks.ipynb       co-appearance networks, matière homophily, Heldendichtung
data/                   pipeline output — {lang}_works_EdB.xlsx, {lang}_manuscripts.xlsx,
                        {lang}_linkage.json, network_comparison.csv
docs/PREPROCESSING.md   full documentation of the export/cleaning pipeline
paper/                  the paper draft (.docx)
```

`lostma.db` (the local cache of the raw Heurist tables) and `credentials` (your Heurist
login) are private to your machine and excluded from this repo — see below.

## Running the pipeline

1. Copy `credentials.example` to `credentials` and fill in your own Heurist login.
2. Run `00-export.ipynb` top to bottom. First run: `db.sync()` pulls from Heurist and
   caches to `lostma.db`; later runs read the cache unless you ask it to refresh.
3. If a colleague hand-reviews `data/{lang}_works.xlsx` in Excel and saves corrections
   as `data/{lang}_works_EdB.xlsx`, re-run 00-export's last cell (or the whole notebook)
   to fold those corrections into the final `matiere` column.
4. Run `01-networks.ipynb` for the network analysis, figures, and statistical tests
   that feed the paper.

Full detail on every step, what each column means, and what changed in this cleanup
(including a bug fix that affects some numbers already drafted in the paper — see the
"Fix 2" section) is in `docs/PREPROCESSING.md`.

## Setup

```
pip install -r requirements.txt
```
