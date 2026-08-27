# mitueberlierung

Co-transmission of French and German chivalric romance: does Jean Bodel's *matière*
scheme (Britain / France / Rome) structure which works were copied together in the
same manuscripts? Research paper + the data pipeline and network analysis behind it.

## Layout

```
00-export.ipynb        pull from the LostMa Heurist database, resolve matière,
                        write the two per-language data files (see docs/)
01-networks.ipynb       co-appearance networks, matière homophily, Heldendichtung
data/                   {lang}_works.xlsx      one row per work, matière resolved
                        {lang}_linkage.json    one row per witness, work ↔ manuscript
                        {lang}_works_EdB.xlsx  hand-kept correction sheet (see docs/REVIEW.md)
                        heurist_sync.txt       when the corpus was downloaded
docs/PREPROCESSING.md   full documentation of the export/cleaning pipeline
docs/REVIEW.md          the matière assignments that rest on human judgement,
                        laid out for a reviewer
paper/                  the paper draft (.docx)
```

`lostma.db` (the local cache of the raw Heurist tables) and `credentials` (your Heurist
login) are private to your machine and excluded from this repo — see below.

## State of play

The corpus is the LostMa snapshot of 26 August 2026 (`data/heurist_sync.txt`). Matière is
resolved automatically where the database allows and corrected by hand where it does not:
27 corrections in French, 6 in German, all listed in `docs/REVIEW.md`. Seven of the French
ones are new and await review. No work is left `Unknown` in either tradition.

## Running the pipeline

1. Run `00-export.ipynb` top to bottom. It reads the cached `lostma.db` by default and needs
   no credentials. To pull a fresh copy from Heurist, copy `credentials.example` to
   `credentials`, fill in your login, and set `REFRESH_FROM_HEURIST = True` in the second cell;
   the download date is then recorded in `data/heurist_sync.txt`.
2. To change a matière, edit `override` in `data/{lang}_works_EdB.xlsx` and re-run the
   notebook's last cell. The correction sheet is hand-maintained and is the only copy of the
   manual judgements, including the German `is_heldenepik` flag, which has no counterpart in
   Heurist.
3. Run `01-networks.ipynb` for the network analysis, figures, and statistical tests
   that feed the paper.

Full detail on every step, what each column means, and what changed in this cleanup
(including a bug fix that affects some numbers already drafted in the paper — see the
"Fix 2" section) is in `docs/PREPROCESSING.md`.

## Setup

Create the project's conda environment (Python 3.12, matching what these notebooks
were last run with; everything else installs via `requirements.txt`, pinned to a
version combination that's verified to install and run together — see the comment
at the top of that file for why bambi/pymc/pytensor/arviz are pinned exactly rather
than left loose):

```
conda env create -f environment.yml
conda activate mitueberlierung
```

Changed `requirements.txt` later? Update the env in place instead of recreating it:

```
conda env update -f environment.yml --prune
```

### Using it in VS Code

The env includes `ipykernel`, so once it's created VS Code should offer it directly:

1. Open this folder in VS Code, open a notebook (`00-export.ipynb` or `01-networks.ipynb`).
2. Click the kernel picker in the top right → **Select Another Kernel** → **Python
   Environments** → `mitueberlierung`.
3. For non-notebook Python files, `Cmd+Shift+P` → **Python: Select Interpreter** →
   `mitueberlierung` does the same for the editor/terminal.

If it doesn't show up in either picker (VS Code sometimes needs a nudge to notice a
newly created conda env), run `Cmd+Shift+P` → **Python: Clear Workspace Interpreter
Setting**, then reopen the notebook and try again — or register it as a Jupyter kernel
explicitly, which always works:

```
conda activate mitueberlierung
python -m ipykernel install --user --name mitueberlierung --display-name "Python (mitueberlierung)"
```

That adds "Python (mitueberlierung)" to the kernel dropdown directly.
