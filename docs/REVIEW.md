# Matière assignments — for review

> **Reviewed by EdB, 27 August 2026.** Two changes accepted and applied. *Willehalm von Orlens*:
> the manual correction to France was withdrawn — it is not part of the Willehalm–Arabel–Rennewart
> trilogy and is absent from Bastert, *Helden als Heilige*; the French source text does not make the
> matière French. Its override was cleared rather than set to `Other`, so it now simply keeps the
> automatic value. *Ille et Galeron*: Rome → Other, following ARLIMA, which gives it as a courtly
> romance; Rome is a setting in the text, not its matter.

Corpus as downloaded from LostMa Heurist on **2026-08-26 14:33 UTC** (recorded in `data/heurist_sync.txt`).

Everything below is a point where a human judgement overrides, or supplies, what the database
resolved automatically. These are the only places in the pipeline where the matière is not derived
mechanically, so they are the ones worth a second opinion.

Corrections live in `data/{lang}_works_EdB.xlsx` — `work_id`, `work`, `matiere_auto` (what the
automatic resolution produced) and `override` (the corrected value). To change an assignment, edit
`override` and re-run `00-export.ipynb`. **Editing the sheet alone is not enough**: the works file
is rebuilt by the notebook's last cell, and until it runs the analysis still reads the old value.
Clearing an override is the right move when the automatic value is already correct.


## French — 28 corrections out of 313 works

### Correcting a value the database got wrong (15)

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
| Ille et Galeron | Rome | **Other** |
| Melusine | Britain | **Other** |
| Mélusine | Britain | **Other** |
| Parthonopeus de Blois | Rome | **Other** |
| Perceforest | Rome | **Britain** |
| Syracon | Rome | **France** |
| Tristan de Nanteuil | Britain | **France** |

### Supplying a value the database lacks (13)

These works resolve to nothing at all — no storyverse, no local matière.

| work | assigned | basis |
|---|---|---|
| Alexandre, 1re et 2e rédactions en prose | **Rome** | *Alexandre en Orient* sits in the `Alexandrian` storyverse and resolves to Rome |
| Alexandre, 3e rédaction en prose | **Rome** | as above — Alexandrian material, no story link in the database |
| Apollonius de Tyr | **Rome** | assigned during the manual review |
| Apollonius de Tyr | **Rome** | assigned during the manual review |
| Apollonius de Tyr | **Rome** | assigned during the manual review |
| Apollonius de Tyr | **Rome** | assigned during the manual review |
| Apollonius de Tyr | **Rome** | assigned during the manual review |
| Athis et Prophilias | **Rome** | roman d'antiquité; no sibling work in the corpus to appeal to |
| Aventures des bruns | **Britain** | Guiron cycle: *Guiron le courtois*, its *Continuation* and *Suite Guiron* all resolve to Britain. **Not** *Brun de la Montagne*, which the database places in France |
| Beaudous | **Britain** | assigned during the manual review |
| Girart de Roussillon, abrégé | **France** | the full *Girart de Roussillon* has its own storyverse and resolves to France |
| Guillaume de Palerne | **Other** | outside Bodel's three; no sibling work in the corpus |
| Le roman de Balain | **Britain** | Post-Vulgate Arthurian; every Merlin/Grail work in the corpus resolves to Britain |


## German — 5 corrections out of 143 works

### Correcting a value the database got wrong (5)

| work | database says | corrected to |
|---|---|---|
| Elisabeth von Nassau-Saarbrücken: 'Sibille | Other | **France** |
| Ulrich von Türheim: 'Rennewart | Other | **France** |
| Ulrich von dem Türlin: 'Arabel | Other | **France** |
| Willehalm' (Prosaroman) | Other | **France** |
| Wolfram von Eschenbach: 'Willehalm | Other | **France** |


## Open points from the review

**Preservation status is not in the works sheet, and cannot be.** It is a property of the
*witness*, not of the work: a composite codex may carry one text complete and another only as a
fragment. It lives in `{lang}_linkage.json` (`Complete`, `Defective`, `Fragmentary`, `Citation`,
`Lost`), one value per witness, and every witness in both traditions carries one. Analyses can
therefore be based on it — Fig. 1's status panel already is — just not from the works sheet.

**Works with no storyverse are where the assignments get fragile.** EdB noted that the original
spreadsheet placed every Elisabeth von Nassau-Saarbrücken text in France except *Sibille*. That is
the same underlying gap as the Willehalm cycle: none of these works is attached to a storyverse, so
each depends on whatever was recorded by hand, and inconsistencies are invisible until someone
reads the list. Linking those storyverses upstream in Heurist would remove the whole class of
problem — see the note on *Wilhelm*, *Arabel* and *Rennewart* in `PREPROCESSING.md`.


## If an assignment changes

Edit `override`, re-run the export, and re-read the figures: changes to Britain/France/Rome move
works in and out of the restricted networks (Figs 2–3) and shift the homophily numbers in §3. The
two changes above moved German assortativity from .676 to .740, putting it above French (.712) and
reversing the direction the draft currently reports.
