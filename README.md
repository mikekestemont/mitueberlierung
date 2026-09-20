# mitueberlierung

Code and data behind

> Elisabeth de Bruijn & Mike Kestemont, *Bodel und das Modell. Gattung und
> Textgemeinschaften in der handschriftlichen Überlieferung französischer und
> deutscher Heldenerzählungen* (in review).

Does Jean Bodel's *matière* scheme (Britain / France / Rome) structure which French and
German chivalric narratives were copied together in the same manuscripts?

## Layout

```
00-export.ipynb              pull from the LostMa Heurist database, resolve matière,
                             write data/
01-networks.ipynb            the analysis: co-appearance networks, matière homophily,
                             Bayesian cohesion models, communities; writes docs/figures/
01-networks-no-ambras.ipynb  sensitivity analysis: the same notebook with the Ambraser
                             Heldenbuch removed (paper, n. 6); writes docs/figures/no_ambras/
data/                        {lang}_works.xlsx      one row per work, matière resolved
                             {lang}_linkage.json    one row per witness, work ↔ manuscript
                             {lang}_works_EdB.xlsx  hand-kept correction sheet (docs/REVIEW.md)
                             heurist_sync.txt       date of the LostMa snapshot
                             network_comparison*.csv  Table 1 of each analysis notebook
docs/PREPROCESSING.md        the export pipeline and every column it writes
docs/REVIEW.md               the matière assignments that rest on human judgement
docs/figures/                the figures as used in the paper
paper/                       current draft
```

## Data

The corpus is the LostMa snapshot of 26 August 2026 (`data/heurist_sync.txt`): 313 French
and 143 German works with all their manuscript witnesses. Matière is resolved from the
storyverse hierarchy where the database allows and corrected by hand where it does not
(28 corrections in French, 5 in German, all listed in `docs/REVIEW.md`). The German
*Heldendichtung* flag exists only in `data/german_works_EdB.xlsx`; it has no counterpart
in Heurist.

## Reproducing

```
conda env create -f environment.yml
conda activate mitueberlierung
```

`requirements.txt` pins every package, including the commit of `Heurist-analyser`, to the
versions the notebooks were last run with.

- `01-networks.ipynb` reads only `data/` and runs top to bottom in under a minute; every
  number and figure in the paper comes from it (the paper's figure numbers match the
  notebook's, table numbers are the notebook's own). The model cells are seeded, so the
  committed outputs reproduce exactly.
- `00-export.ipynb` rebuilds `data/` from the raw Heurist tables. It reads a local cache
  (`lostma.db`, not committed) and, if there is none, downloads one — which needs a LostMa
  Heurist login in a `credentials` file (copy `credentials.example`). To change a matière,
  edit `override` in `data/{lang}_works_EdB.xlsx` and re-run the notebook's last cell.

## AI assistance

The notebook code, its documentation and this repository's text were written with the
help of Anthropic's Claude Opus models, used through Claude Code, and reviewed by the
authors. The research questions, the matière assignments and the paper are the authors' own.
