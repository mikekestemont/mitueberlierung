# Matière assignments — for review

> **Reviewed by EdB, 27 August 2026.** Two changes accepted and applied:
> *Willehalm von Orlens* — the manual correction to France was withdrawn (it is not part of the
> Willehalm–Arabel–Rennewart trilogy and does not appear in Bastert, *Helden als Heilige*); it now
> keeps its automatic value, Other. *Ille et Galeron* — Rome → Other, following ARLIMA, which gives
> it simply as a courtly romance; Rome is a setting in the text, not its matter.

Corpus as downloaded from LostMa Heurist on **2026-08-26 14:33 UTC** (recorded in `data/heurist_sync.txt`).

Everything below is a place where a human judgement overrides, or supplies, what the
database resolved automatically. These are the only points in the pipeline where the
matière is not derived mechanically, so they are the ones worth a second opinion.

Corrections live in `data/{lang}_works_EdB.xlsx` — `work_id`, `work`, `matiere_auto`
(what the automatic resolution produced) and `override` (the corrected value). To change
an assignment, edit `override` and re-run `00-export.ipynb`; nothing else needs touching.


## French — 27 corrections out of 313 works

### Correcting a value the database got wrong (14)

| work | database says | corrected to |
|---|---|---|
| Antioche | Other | **Rome** |
| Chevalier au cygne | France | **Other** |
| Cligès | Rome | **Britain** |
| Cligès | Rome | **Britain** |
| Floire et Blancheflor, version aristocratique | Rome | **Other** |
| Floire et Blancheflor, version dite populaire | Rome | **Other** |
| Floriant et Florete | Rome | **Britain** |
| Floriant et Florete | Rome | **Britain** |
| Melusine | Britain | **Other** |
| Mélusine | Britain | **Other** |
| Parthonopeus de Blois | Rome | **Other** |
| Perceforest | Rome | **Britain** |
| Syracon | Rome | **France** |
| Tristan de Nanteuil | Britain | **France** |

### Supplying a value the database lacks (13)

These works resolve to nothing at all — no storyverse, no local matière.

| work | assigned | basis | status |
|---|---|---|---|
| Alexandre, 1re et 2e rédactions en prose | **Rome** | *Alexandre en Orient* sits in the `Alexandrian` storyverse and resolves to Rome | **NEW — needs review** |
| Alexandre, 3e rédaction en prose | **Rome** | as above — Alexandrian material, no story link in the database | **NEW — needs review** |
| Apollonius de Tyr | **Rome** | assigned during the earlier manual review | previously reviewed |
| Apollonius de Tyr | **Rome** | assigned during the earlier manual review | previously reviewed |
| Apollonius de Tyr | **Rome** | assigned during the earlier manual review | previously reviewed |
| Apollonius de Tyr | **Rome** | assigned during the earlier manual review | previously reviewed |
| Apollonius de Tyr | **Rome** | assigned during the earlier manual review | previously reviewed |
| Athis et Prophilias | **Rome** | **judgement call** — roman d'antiquité; no sibling work in the corpus to appeal to | **NEW — needs review** |
| Aventures des bruns | **Britain** | Guiron cycle: *Guiron le courtois*, its *Continuation* and *Suite Guiron* all resolve to Britain. **Not** *Brun de la Montagne*, which the database places in France | **NEW — needs review** |
| Beaudous | **Britain** | assigned during the earlier manual review | previously reviewed |
| Girart de Roussillon, abrégé | **France** | the full *Girart de Roussillon* has its own storyverse and resolves to France | **NEW — needs review** |
| Guillaume de Palerne | **Other** | **judgement call** — outside Bodel's three; no sibling work in the corpus | **NEW — needs review** |
| Le roman de Balain | **Britain** | Post-Vulgate Arthurian; every Merlin/Grail work in the corpus resolves to Britain | **NEW — needs review** |


## German — 6 corrections out of 143 works

### Correcting a value the database got wrong (6)

| work | database says | corrected to |
|---|---|---|
| Elisabeth von Nassau-Saarbrücken: 'Sibille | Other | **France** |
| Rudolf von Ems: 'Willehalm von Orlens | Other | **France** |
| Ulrich von Türheim: 'Rennewart | Other | **France** |
| Ulrich von dem Türlin: 'Arabel | Other | **France** |
| Willehalm' (Prosaroman) | Other | **France** |
| Wolfram von Eschenbach: 'Willehalm | Other | **France** |


## What most needs your eye

1. **The two judgement calls.** *Athis et Prophilias* (→ Rome) and *Guillaume de Palerne*
   (→ Other) have no sibling work in the corpus to appeal to; they rest on classification
   alone. Every other new assignment is backed by a related work whose storyverse resolves.

2. **The Willehalm group.** Six German works are assigned to France because they are the
   German reflex of the Guillaume d'Orange cycle. The database attaches them to no
   storyverse, so without this they fall to *Other* — which is what happened in an earlier
   version of the figures. The durable fix is upstream: linking the *Wilhelm*, *Arabel* and
   *Rennewart* storyverses to the Matter of France in Heurist would make five of the six
   corrections unnecessary.

3. **England.** Works the database labels *Matter of England* (insular heroes outside
   Arthurian tradition — the *Bueve de Hanstonne* branches, *Gui de Warwick*, the German
   *Oswald* redactions) are folded into *Other*, not into *Britain*. If you would rather
   they counted as Britain, that is a one-line change and not a per-work correction.


## If an assignment changes

Editing `override` and re-running the export is enough — the analysis notebook reads the
result and does no remapping of its own. Bear in mind that changes to Britain/France/Rome
assignments move works in and out of the restricted networks (Figs 2–3) and will shift the
homophily figures in §3, so the paper's numbers should be re-read from a fresh run
afterwards.
