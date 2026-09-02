# CLAUDE.md

Project overview, layout and data provenance are in `README.md`; the pipeline is
documented in `docs/PREPROCESSING.md` and the human-judgement matière calls in
`docs/REVIEW.md`. Read those first. This file covers only working conventions.

## Environment

Use the project conda env, not a hand-rolled venv or ad-hoc `pip install`:

```bash
conda env create -f environment.yml   # first time
conda activate mitueberlierung
```

`requirements.txt` pins the bambi/pymc/pytensor/arviz stack to an exact, jointly
verified version set — do not loosen those pins or install them piecemeal.

## Where code goes

**All analysis lives in the notebooks.** Do not create standalone `.py` analysis
scripts beside them; if something is worth keeping, it becomes a notebook cell.
(Throwaway tooling for *editing* a notebook is fine, but keep it out of the repo.)

- `00-export.ipynb` — Heurist pull, matière resolution, writes `data/`
- `01-networks.ipynb` — everything downstream: networks, homophily, Heldendichtung

## Running 01-networks.ipynb

The `## Modelling the difference` section (bambi/pymc: null vs additive vs
interaction, plus the posterior figure) fits three NUTS models and is slow. When a
change does not touch it, execute only the cells above it and leave that section's
cached outputs alone rather than re-running the whole notebook.

`matiere_node_table` feeds those models via its `within` / `total` columns.
**Those two columns' semantics must stay stable** (weighted ties to other
Britain/France/Rome works only) — add new columns rather than redefining them.

## Methodological conventions settled so far

- **Weighted statistics are primary** in the paper text. Edge weight = number of
  manuscripts a pair of works shares, and the argument turns on repeated
  co-transmission, not mere co-occurrence. The unweighted figures are kept
  alongside for reference.
- **Newman assortativity comes from networkx**, not a hand-rolled formula.
  networkx has no `weight` parameter for categorical assortativity, so the
  weighting is expressed by replicating each edge into `weight` parallel edges on
  a `MultiGraph` and calling `nx.attribute_assortativity_coefficient` on that.
  Verified identical to the closed-form equivalent.
- **The permutation test is the honest significance check** (each link counted
  once). The binomial models describe and visualise the matière-by-tradition
  pattern with credible intervals; they do not add independent significance
  claims, because each link contributes to both its endpoints.
- **Other is included, not dropped, in the network figure**, with Heldendichtung
  split out of it (`mat5`, purple) and only genuine residual Other left grey.
  Other is not a coherent category, so never report a single "share of ties to
  Other" figure — split it into Heldendichtung vs residual. This matters most for
  German Britain, whose apparent cohesion drops sharply once Other is restored,
  almost entirely because of Heldendichtung contact.
- Tables 2–3 still *restrict* to Britain/France/Rome; only the figure shows
  everything. Keep that distinction explicit in captions.

## Accuracy discipline

Numbers in the paper draft (`paper/*.docx`, and the Google Doc version) have drifted
from the notebook before, after matière corrections landed. Before quoting any
figure in prose, re-run the relevant cell and check against it. Bracketed
`[CHECK ...]` notes in the draft mark places where this was already suspected.

## Gotchas

- A second, **unversioned copy** of this project has existed at
  `~/Desktop/mitueberlierung`. It is stale. This repo is authoritative.
- Figure/table numbering in the notebook is independent of the paper's; don't
  assume "Figure 2" means the same thing in both.
