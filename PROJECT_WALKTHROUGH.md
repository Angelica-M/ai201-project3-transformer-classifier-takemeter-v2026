# TakeMeter — Project Walkthrough

> **Who this is for:** someone with no prior context on this project or on
> fine-tuning. Every technical term is explained on first use. Numbers come
> from files and commands actually read and run, not estimated.
>
> **How this file is built:** a diary and a manual together, one unit at a
> time. Earlier entries are never rewritten or reordered — only added to.
>
> **On accuracy:** the author's instruction for this file was explicit —
> *"the repo may contradict what was in the original details on Unit 5 and 6;
> in such cases, always trust the repo as the true reality."* Every claim
> below was checked against the actual repository (git history, `README.md`,
> `criteria.md`, `data/labels.csv`, `build_dataset.py`, `takemeter.ipynb`,
> `results.json`) rather than assumed from the course brief. Where the repo
> and the brief disagree, the disagreement is named, not smoothed over.
>
> **Scope:** this file currently covers **Unit 5** only. Unit 6 content
> (baseline comparison, staff-label agreement, three-seed testing,
> improvement) exists in the repository already — `baseline.py`,
> `agreement.py`, `results_three_seeds_before/after.json` are all present —
> but is deliberately not documented here until that unit's brief and
> milestones are supplied.

---

## 1. Unit Introduction

### Unit 5 — Fine-Tuning a Model

The prior two units built and tested an agent that calls a general-purpose,
pre-trained model. This unit does something different: it trains a
**specialist** — a small model taught one specific distinction, from
examples labeled by hand, cheaply enough to run on every item in a dataset
rather than one at a time through a hosted API.

**What's being built:** TakeMeter, a classifier that sorts posts from a
chosen online community into labels its author defines. The training run
itself — fine-tuning a pre-trained language model on labeled examples —
is quick: in this repository, three seeds at 3 epochs each took about 3.6
minutes on a laptop CPU (README, "Run Log — Before"). The unit is built around the parts
that take longer: writing label definitions precise enough that two people
would apply them the same way, deciding the rule for the hardest boundary in
the taxonomy before hitting it 40 times, and collecting a dataset that
doesn't let one label dominate the rest.

This project runs across **two units that do different jobs**:

| Unit | Job |
|---|---|
| **Unit 5** (this document's current scope) | Design the label taxonomy, collect and label at least 200 real examples, run one training pass, and commit five acceptance criteria — written before any result exists. |
| **Unit 6** | Test the classifier against a zero-shot baseline and against labels a staff member wrote independently for the same posts. Same repository both units. |

In this unit **all five acceptance criteria are the author's own** — no
starting scaffolding is given.

**Why this matters (the brief's own framing, kept intact):** fine-tuning is
what makes a general model useful on a specialized task — legal review,
medical coding, content moderation. The skill that transfers is not running
the training pipeline; it's judgment about data — recognizing the difference
between a model that learned the intended distinction and one that learned
something merely correlated with it.

### Learning Objectives (Unit 5)

By the end of Unit 5, the goal is to be able to:

1. Design a **label taxonomy** with definitions two people would apply the
   same way.
2. Write a **decision rule** for the hardest boundary in the taxonomy before
   it's needed in practice.
3. Collect and label a real dataset, recording which cases were genuinely
   hard.
4. Recognize **class imbalance** and what it does to what a model actually
   learns.
5. Run a fine-tuning pipeline end to end and keep the resulting metrics.
6. Write acceptance criteria for a system that hasn't been trained yet.

---

## 2. Project Title, Problem, Goals, and Architecture

### Problem and Taxonomy as Built (Unit 5)

**The problem it solves:** online communities produce a mix of posts, and
regulars can usually sort them by feel within a sentence, without being able
to state the rule they're using. TakeMeter is an attempt to make that rule
explicit and automatable for one specific community.

**What it does, as actually built:** this project's taxonomy classifies
posts from **r/estimators** (a construction-cost-estimating community) by
subject matter and poster role — which trade, which software, which part of
the bidding process, or whether the post is unrelated to estimating at all.
This is a different flavor from the brief's own running examples (`analysis`
/ `hot_take` / `reaction` — a quality-of-argument taxonomy for a sports or
music community). That's a valid reading of "you choose the labels," not a
deviation from the assignment — but it's worth stating plainly, since the
taxonomy actually built sorts posts by *topic*, not by *argument quality*.

**A significant, measurable departure from the Features spec:** the brief
calls for **two to four labels**. This project has **21**, organized into six
thematic groupings (trade scope and division boundaries; software stack and
tooling; project sector and delivery model; operational intent; poster role
and expertise; and a stand-alone `unrelated` label). This is named here
plainly because it is the single fact that most shapes everything that
follows — see Architecture and Findings below for its direct, visible
consequences on the data and the first training result.

### Goals (Unit 5)

By the end of Unit 5, the project demonstrates the ability to:

- Write label definitions two people would apply the same way.
- Label real text consistently, and record the cases that were hard.
- Run a fine-tuning pipeline and get a model and metrics out of it.
- Write acceptance criteria for something that has never been trained yet.

### Architecture

TakeMeter has two distinct phases, run by different tools, at different
times, and this split matters: **Phase A builds the dataset; Phase B trains on
it.** Phase A is not described in the course brief at all — it is this
repository's own, substantially automated solution to Milestone 4. The
diagram below was generated by a script (box widths are computed, not typed)
and every behavioural claim in it was matched to a line of
`build_dataset.py` or `takemeter.ipynb` as they exist now: 46 claims, 46
matched. Two things in it are deliberately sourced elsewhere: the 281 → 204
row counts (from the README, since the code does not record them) and the
Unit 5 training settings (see the paragraph under the diagram).

```
+----------------------------------------------------------------------------------+
| PHASE A  build_dataset.py -- collect posts, then label them                      |
+----------------------------------------------------------------------------------+
| main() starts: load_dotenv(override=True) reads GEMINI_API_KEY from .env;        |
| read_rows() loads data/labels.csv (empty list if the file is missing).           |
|                                                                                  |
| Is any row's label (column 2) empty?                                             |
|   YES -> 'Resuming': skip all scraping, go straight to the dedupe step.          |
|   NO  -> Phase 1 (below).                                                        |
+----------------------------------------------------------------------------------+
                                          |
                                          | no empty label
                                          v
+----------------------------------------------------------------------------------+
| A1  Phase 1: scrape r/estimators (Reddit Atom search feed, no login)             |
+----------------------------------------------------------------------------------+
| 3 searches in order: Top + keywords, Hot + keywords, Top + 'self:true'.          |
| Each wants 100 NEW unique posts. URL: .../search.rss?...&limit=100&after=<id>    |
| At least 60 s between requests; on HTTP 429 wait x-ratelimit-reset + 2 s         |
| (4 tries). Any other non-200 status stops the scrape with a message.             |
|                                                                                  |
| Skipped, not counted: an id already seen, a body of [removed]/[deleted],         |
| text of 30 characters or fewer, text already in column 1 (whitespace-            |
| normalised). A search also ends early if Reddit has no more results, a page      |
| has nothing unseen, or a page is shorter than 100 entries.                       |
|                                                                                  |
| Each new post is appended at once to data/labels.csv as [text, '', ''].          |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| A2  remove_duplicate_rows()  (always runs, also on a resume)                     |
+----------------------------------------------------------------------------------+
| Keeps the first row for each whitespace-normalised column 1.                     |
| If any were removed, rewrites the file (temp file + os.replace).                 |
| Recorded in the README: 281 rows collected -> 204 unique.                        |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| A3  Phase 2: label each unlabelled row with Gemini, one row at a time            |
+----------------------------------------------------------------------------------+
| discover_models(): models this key can call, preferred list first.               |
| ModelPool.current() = first model not yet marked dead.                           |
|                                                                                  |
| For each row, in file order:                                                     |
|  CHECK 1  label already filled?  -> skip, never overwrite.                       |
|  label_post_with_gemini(text, pool): JSON reply -> must name two DIFFERENT       |
|    labels from the 21 allowed -> note 'Highest semantic match of X% for          |
|    L1 (with 2nd highest of Y% as L2) due to reason'.                             |
|    unusable reply (bad JSON / invalid label): retry same model, 3 tries,         |
|      then give up on this row (stays unlabelled, counted as skipped).            |
|    permanent error (daily quota, 401/403/404): mark model dead, next model.      |
|    transient error (429, 5xx, network): wait, retry same model 3 times,          |
|      then mark it dead and rotate.                                               |
|    no model left: AllModelsExhausted -> run stops; a rerun resumes.              |
|  CHECK 2  re-read the file: row must still hold the text that was sent           |
|    and still have an empty label, else nothing is written.                       |
|  write_rows() via temp file + os.replace; then pause 4 s.                        |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| RESULT  data/labels.csv   (header: text,label,note)                              |
+----------------------------------------------------------------------------------+
| 204 rows, every row labelled; 16 of the 21 labels occur (5 have no post).        |
+----------------------------------------------------------------------------------+

====================================================================================

+----------------------------------------------------------------------------------+
| PHASE B  takemeter.ipynb sections 1-5 -- the Unit 5 training run                 |
+----------------------------------------------------------------------------------+
| Reads data/labels.csv. Nothing in this phase calls an API.                       |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| Section 1  pick_device()                                                         |
+----------------------------------------------------------------------------------+
| cuda if available, else mps, else cpu. Recorded later in results.json.           |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| Section 2  LABELS, settings, load data, check_dataset()                          |
+----------------------------------------------------------------------------------+
| Settings cell: BASE_MODEL, EPOCHS, LEARNING_RATE, BATCH_SIZE, MAX_LENGTH, SEED.  |
| check_dataset() returns (problems, warnings). Problems: missing text/label       |
| column, blank text, label not in LABELS, a label above 70% of rows.              |
| Warnings: label in LABELS but absent from the CSV, a label above 60%, fewer      |
| than 150 rows (or 200), duplicate posts, any label with under 20 examples.       |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| Section 3  split_dataset() + describe_splits()                                   |
+----------------------------------------------------------------------------------+
| 70% train / 15% validation / 15% test, stratified by label. If any label         |
| has fewer than 2 rows the split falls back to plain random. 204 rows ->          |
| 142 / 31 / 31. Labels with under 8 test rows are flagged as thin.                |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| Section 4  train_model()  (full fine-tune, all weights trainable)                |
+----------------------------------------------------------------------------------+
| AutoModelForSequenceClassification on distilbert-base-uncased, 21 labels,        |
| Hugging Face Trainer, set_seed(seed), evaluation once per epoch.                 |
| Unit 5 run: 3 epochs, lr 2e-5, batch 16, 128 tokens, seed 42. No LoRA.           |
+----------------------------------------------------------------------------------+
                                          |
                                          v
+----------------------------------------------------------------------------------+
| Section 5  evaluate() -> files written                                           |
+----------------------------------------------------------------------------------+
| results.json: seed, n_test, accuracy, f1_macro, per_label, confusion,            |
| confidence, plus device, torch version and max_length.                           |
| test_split.csv: the 31 held-out posts.                                           |
| Unit 5 result: accuracy 0.258, macro F1 0.061 (31 test posts, cpu).              |
+----------------------------------------------------------------------------------+
```

**On sourcing this diagram's Phase B numbers**: the live `takemeter.ipynb`
shows `EPOCHS = 15`, not `3`, because Unit 6's improvement milestone edited
that same notebook cell in place. The settings in Section 4 of the diagram
(3 epochs) come from the commit that recorded the Unit 5 run (`552f709`) and
from `README.md`, not from the current file, which would attribute Unit 6's
configuration to Unit 5's result. The `check_dataset()` and `evaluate()`
code shown is as it exists now; `evaluate()` also gained a `high_confidence`
block in `7c15780` (end of Unit 5), which the committed `results.json` from
`552f709` therefore does not contain.

**Not pictured, and why:** `baseline.py`, `agreement.py`, `kappa.py`,
`macro_f1_ci.py`, and the three-seed training loop (`takemeter.ipynb`
section 6 onward) all exist in this repository already, but are Unit 6
tooling — left out of this diagram because this document doesn't cover Unit
6 yet.

---

## 3. Glossary

| Term | What it means | Where it shows up in TakeMeter |
|---|---|---|
| **Label** | The category assigned to one post. | One of 21 values in `data/labels.csv`'s `label` column, e.g. `general_construction_non_mechanical`. |
| **Taxonomy** | The whole set of labels, plus the rules for choosing between them. | `README.md`'s "Label Taxonomy" section — 21 labels across 6 groupings, mirrored verbatim in `label_definitions.txt`. |
| **Annotation** | Assigning labels by hand; also called labeling. | In this repo, almost none of the 204 rows were annotated by hand — see Milestone 4. |
| **Class imbalance** | When one label covers most of the data, so a model learns to guess that label. | The largest label (`industry_venting_operational_burnout`) holds 26.96% of the 204 rows — under the 70% cap, but see Findings for a different imbalance shape this check doesn't catch. |
| **Fine-tuning** | Taking a model that already knows language and teaching it a specific distinction. | `distilbert-base-uncased`, fully fine-tuned (all weights updated) on the 204-row dataset — not LoRA, despite early documentation saying otherwise. |
| **Epoch** | One full pass of the training data through the model. | This run: 3 epochs. |
| **Held-out set** | Examples the model never trained on — the only honest place to measure it. | The 31-post test split, written to `test_split.csv`. |
| **Leakage** | Held-out examples sneaking into training, making scores look good and mean nothing. | Guarded against by writing one unsplit CSV and letting the notebook do the 70/15/15 split exactly once. |
| **Decision rule** | A written rule for the single hardest boundary in a taxonomy, decided before labeling at scale. | `README.md`'s "The 1st Hardest Boundary" and "The 2nd Hardest Boundary" sections. Introduced in Milestone 2. |
| **Model pool** | A fallback list of model names tried in preference order, advancing only on a permanent failure. | `build_dataset.py`'s `ModelPool` class — rotates Gemini models on 404/401/403/daily-quota errors, retries the same model on transient ones. Introduced in Milestone 4. |

---

## 4. How to Run This Project

Full environment setup — venv, `test.py`, the notebook kernel, and the
Gemini-key step this repo's data pipeline needs — is in the companion file
**[`ENVIRONMENT_SETUP.md`](ENVIRONMENT_SETUP.md)**. This section is the
minimum path, with output captured from this repository's `.venv` on
2026-10-31.

**1. Create the environment and check it:**
```bash
python -m venv .venv
# activate it (see ENVIRONMENT_SETUP.md section 2 for the exact command per OS), then:
pip install -r requirements.txt
python test.py
```
Captured output (this working copy, `.venv` active):
```
AI201 environment check
------------------------------------------------------------
[PASS] Python version
         3.13.7 on Windows
[PASS] Virtual environment
         C:\Users\Angelica M\Documents\CP Prework\ai201-project3-takemeter-starter-v2026\.venv
[PASS] Pinned packages
         all 6 import cleanly
[PASS] Free disk space
         286.4 GB
[PASS] Memory
         15.3 GB
[PASS] PyTorch
         2.14.1+cpu, CPU only — fine, a three-seed run is minutes not hours
[PASS] Training packages
         datasets and accelerate both import
[PASS] Transformers
         imports cleanly
[PASS] Baseline model
         already downloaded
[PASS] Project files
         notebook, scripts and practice data all present
------------------------------------------------------------
10 passed, 0 failed, 0 to look at, 0 skipped
```

**2. Open the notebook, confirm the kernel, run section 1's device cell.**
Open `takemeter.ipynb` in VS Code, pick this project's `.venv` as the
kernel, run the first code cell. Captured output (the cell's source run
with the `.venv` interpreter; its closing explanatory paragraph is omitted):
```
device: cpu  (CPU)
torch:  2.14.1+cpu
```
The same device and torch version are recorded in `results.json`.

**3. Run sections 2-5 for the Unit 5 training run.** In this repository
that was done by executing the notebook's code cells from a script rather
than in the editor window (README, "The Training Run"). It writes
`results.json` and `test_split.csv`.

**4. (Optional) Rebuild the dataset.**
```bash
pip install google-generativeai python-dotenv requests
cp env.txt .env   # then set GEMINI_API_KEY
python build_dataset.py
```
> **Caution:** run this only in a copy of the repository with `data/labels.csv`
> removed. On the real checkout it appends new posts to the committed
> dataset; the full explanation is in `ENVIRONMENT_SETUP.md` section 4.

---

## 5. Findings and Lessons Learned

Eight findings, each stated with the evidence behind it.

**1. A quantity constraint is a design discipline, not a formality, and
exceeding it by 5x has a direct, traceable consequence.** The brief specifies
2-4 labels. This project has 21. That single choice is the direct cause of
the data-sparsity problem visible in the first training result: 204 examples
spread across 21 labels averages under 10 per label, and 10 of the 21 labels
have 0 or 1 example in the entire dataset (5 have none, 5 have exactly one;
12 have 3 or fewer). A classifier cannot learn a distinction it has one
example of.

**2. A check that catches one shape of imbalance does not catch a different
shape of the same underlying problem.** `check_dataset()`'s 70%-single-label
cap passed cleanly — the largest label holds only 26.96% of the data. But
"no label dominates" and "every label has enough examples to learn from" are
different properties, and this dataset fails the second while passing the
first. The brief's imbalance check was written with a 2-4-label taxonomy in
mind; it doesn't generalize to a 21-label one.

**3. "Pre-label a batch, then read and correct every one" and "label 100%
of the dataset automatically, with no correction pass claimed" are different
workflows, and the gap between them is large enough to require explicit
disclosure, not a passing mention.** The brief's Milestone 4 describes the
former. This repo's `README.md` states the latter directly: *"This section
does not claim a manual re-check of the labels."* That disclosure, stated
plainly rather than hedged, is itself worth noting as the right way to
report a process that diverged from the assignment.

**4. A repository's own documentation can contradict itself, and the honest
response is to name the contradiction, not silently pick a side.** `README.md`'s
"The Dataset" section ("How I labelled them", at line 337 as of this audit)
states the first 20 ("cold") rows' labels "were also assigned by Gemini."
The "How I Used AI" section's pre-labelling disclosure (line 465) states the
same 20 rows "were not assigned by Gemini but were assigned by human
opinion." These cannot both be true. The circumstantial
evidence favors the first claim — the actual note text on those rows (e.g.
*"Highest semantic match of 85% for software_benchmarking_technical_workflow
... due to..."*) matches Gemini's generated note format exactly, which a
human annotator following the brief's "no help, no tools" instruction would
have no reason to produce. This isn't resolved here with certainty; it's
reported as an open discrepancy for the author to confirm.

**5. Disclosing that a target was adjusted after results were already visible
is a different, more defensible act than adjusting it silently.** `README.md`
states directly: *"Criterion 2 was reworded... after the first single-seed
training run had been executed, when its overall accuracy was already
visible."* This is a real deviation from "write criteria before you see any
results." It is also self-reported, in the open, rather than discovered by
an outside audit — which is the more useful of the two ways this kind of
thing tends to come to light.

**6. A file's current content and its content at the time something ran are
different sources, and citing the wrong one misattributes a result.** The
live `takemeter.ipynb` shows `EPOCHS = 15` — Unit 6's later value, written
into the same cell Unit 5 used. The actual Unit 5 configuration
(`EPOCHS = 3`) only survives in git history (`552f709`) and in `README.md`'s
prose. Describing "the notebook's defaults" by reading the current file,
instead of the commit, would report Unit 6's training configuration as if it
were Unit 5's.

**7. A documentation error caught later is corrected where it appears and recorded in a commit, except for a dated prediction, which is annotated and not edited.** `criteria.md` and parts of `README.md` described this project as using LoRA (a partial-weight fine-tuning technique). It does not — it fully fine-tunes `distilbert-base-uncased`, confirmed directly against the training code (`AutoModelForSequenceClassification.from_pretrained`, no `peft` import, no `peft` in `requirements.txt`). The correction was made in Unit 6 in three ways: descriptive sentences were rewritten in place (`criteria.md` in `ab26ff5`; the README's "What This Does" paragraph in `6e2a454`), the README's Baseline vs. Trained prediction kept its original "LoRA model" wording with a note beneath it, because a prediction must not change after the fact, and the file `DistilBERT_LoRA_and_full_fine_tuning_explained.md` records the whole distinction. The original wrong wording therefore survives only in git history and in the annotated prediction, not as a footnote beside every instance.

**8. "No hosted model, no API key" can be true of one part of a pipeline and
false of another, and the boundary between them is exactly where a
reproducibility problem hides.** The classifier itself — training, baseline,
`test.py` — is genuinely local-only, confirmed by `requirements.txt` having
no hosted-model dependency. The dataset it trains on was built by a script
requiring a Gemini API key, using three packages (`google-generativeai`,
`python-dotenv`, `requests`) that aren't in `requirements.txt` at all. Both
halves of that sentence are true at once; reporting only the first half (as
the brief's setup text does) is incomplete, not wrong.

---

## 6. Milestones

Documented retroactively from this repository's real git history (66
commits in total at the time of the audit, from `git log --oneline | wc -l`;
the Unit 5 window runs from `8bd54ab` on 2026-10-18 through `7c15780` on
2026-10-19, 28 commits inclusive). Every per-file insertion/deletion count
below was re-derived with `git show --numstat <hash>`, not `--stat`, whose
per-file digit is a total-changed count easily misread as an insertion count.

### Milestone 1: Pick a Community and Learn Its Nomenclature

**The idea.** A taxonomy built without first reading how a community
actually talks produces labels that don't match how its members would sort
their own posts. This step has no code output by design — it is reading, so
there is nothing to commit.

**Where the answers went.** No commit is uniquely attributable to this
step — expected, since it's read-only research. The community chosen,
**r/estimators** (construction cost estimating), first appears in the commit
that builds the "What This Does" section (`8bd54ab`, 2026-10-18).

**Milestone summary.** True now: a community is chosen and its posts have
been read closely enough to inform the next milestone's taxonomy design.
Still a placeholder: no taxonomy, no labels, no data yet.

**What this milestone taught:** choosing a professional/trade community
(estimators discussing bids and takeoffs) rather than a hobbyist
opinion-community (the brief's own NBA/music examples) changes what "sorting
posts" even means — by topic and role, rather than by argument quality. That
choice, made here, shapes everything built afterward.

**Before moving on:** not independently verifiable from repository contents
beyond the community name itself appearing in Milestone 2's commits.

---

### Milestone 2: Design the Labels

**The idea.** A taxonomy where two people would label the same post
differently produces a model that learned an inconsistent signal. Writing
the hardest boundary's decision rule before labeling at scale is what keeps
that inconsistency from compounding across 200 examples.

**Where the answers went.** `README.md`'s Label Taxonomy and hardest-boundary
sections. Real commits (all 2026-10-18): `8bd54ab` ("What This Does",
README +11/−2), `83c23c5` ("Label Taxonomy", README +214/−13 — this is where
the real taxonomy was written), `015cbfb` ("The hardest boundary", README
+9/−3), and a second boundary added the next day in `aa1642c` (2026-10-19,
README +22/−2, which also adds the 21st label, `unrelated`). `00a249f` — whose
message also mentions the second boundary — changes no README line:
`--numstat` shows only `build_dataset.py` (+153/−0), `env.txt` (+5/−0) and
`.gitignore`.

**Step: write labels, each with a one-sentence definition and two example
posts.**
- What was written: **21 labels**, not the 2-4 the brief specifies, grouped
  into 6 thematic distinctions (Trade Scope & Division Boundaries; Software
  Stack & Tooling Architecture; Project Sector & Delivery Model; Operational
  Intent & Post Purpose; Poster Role & Expertise Level; and a stand-alone
  `unrelated`). Example, verbatim: `division_23_full_mechanical_hvac` —
  *"Posts covering entire building mechanical and environmental control
  systems under a single prime mechanical quote, combining sheet metal,
  equipment schedules, and hydronic piping."*
- Decision made (implicit, worth stating explicitly since it's a real
  departure):

  | Option | Chosen / consequence |
  |---|---|
  | 2-4 labels, as specified | Not chosen |
  | 21 labels across 6 groupings, for finer-grained real-world coverage of a professional community's topics | **Chosen** — richer taxonomy, but see Findings #1 for the direct cost: most labels end up with too few examples to be learnable from 204 total posts |

**Step: find one post between two labels, write the decision rule.**
- What was written, verbatim (first boundary): between
  `general_pdf_cad_digitizers` and `specialized_mechanical_estimating_databases`
  — *"Classify it as `specialized_mechanical_estimating_databases`, if the
  post describes trade-specific features that automatically translate
  measurements into mechanical components... classify it as
  `general_pdf_cad_digitizers` regardless of the trade context mentioned."*
- A second boundary was added the next day, between
  `general_construction_non_mechanical` and `unrelated` — *"Classify a post
  as `unrelated` only if it contains no construction estimation, bidding,
  takeoff, or precon content whatsoever."* Writing a second rule beyond the
  one required is itself notable — it suggests the first rule alone didn't
  cover every ambiguous case that showed up once labeling started.

**Step: stress-test the labels with an AI tool.**
- **No distinct artifact for this step was found.** The only AI involvement
  documented anywhere in this repository is the Gemini auto-labeling
  pipeline built in Milestone 4, which is a different activity (labeling the
  real dataset) from this step's intent (generating adversarial boundary
  posts to pressure-test the definitions before labeling begins). This may
  have happened without being recorded, or may not have happened as a
  distinct step — the repository doesn't show which.

**What this means for the next milestone.** The 21-label taxonomy and its
two decision rules are exactly what `build_dataset.py` sends to Gemini for
every row in Milestone 4 — the taxonomy's size is what Milestone 4's
automated pipeline has to scale to.

**Terms from this milestone:** Taxonomy, Decision rule (see Section 3).

**Milestone summary.** True now: a taxonomy exists with definitions and
decision rules for two boundaries. Still notably different from spec: 21
labels against a 2-4 target, and no recorded AI stress-test step.

**What this milestone taught:**
1. A label count isn't a neutral design choice — it directly determines how
   much data per label a fixed-size dataset can provide.
2. Writing a second boundary rule beyond the required one is a sign the
   first ambiguous case found wasn't the only one.
3. A step without a committed artifact can't be verified as having happened,
   which is itself worth knowing before assuming it did.

---

### Milestone 3: Write the Acceptance Criteria

**The idea.** An acceptance criterion names a number specific enough that
someone else could check it. All five are the author's own — no criteria
were given to build from.

**Where the answers went.** `criteria.md`. Real commits: `8b5db01` (message
"Create and define Criteria.md", +67/−28, 2026-10-18 — the file already
existed as the course template since the initial commit `9772abf`, which is
why there are deletions) and `395868b` (fixes, +9/−12, same day).
Two later edits, `75df603` and `7c15780` (both 2026-10-19), revise Criterion
2 and add final checks — both dated *after* the training commit (`552f709`,
same day) — see Findings #5.

**The five criteria, verbatim targets and reasoning:**

| # | Criterion (area) | Target | Why this target (as written) |
|---|---|---|---|
| 1 | Overall Classification Accuracy (overall accuracy) | ≥0.65 across all 3 seeds | "Random guessing across 21 distinct labels... yields an expected baseline accuracy of only about 0.048 (4.8%)." |
| 2 | Core Technical Label Minimum Performance (per-label) | Macro F1 ≥0.60 every seed, stdev <0.05, over 5 named labels with ≥10 posts each | Per-label F1 on a ~31-post test split is too noisy across 21 separate numbers; 5 labels fixed in advance, chosen by post count, deliberately excluding the two largest ("not technical estimating topics"). |
| 3 | Class Imbalance and Noise-Filter Precision (balance) | Precision of `general_construction_non_mechanical` ≥0.75, all seeds | So "actual mechanical scope discussions are not accidentally filtered out." |
| 4 | Inter-Annotator Agreement Consistency (consistency) | Cohen's kappa ≥0.70 on 30 held-out posts, two annotators | "Fine-grained taxonomy distinctions... rely on subtle context; human annotation must be demonstrably consistent." |
| 5 | Confidence Calibration on High-Certainty Predictions (confidence) | Precision ≥0.85 on predictions with confidence ≥0.80 | "Estimators using TakeMeter to sort bid opportunities need to trust high-confidence model tags without double-checking." |

The criteria are labelled as covering the brief's five suggested areas —
the brief only required three — with one caveat on the match: Criterion 3 is
titled "Class Imbalance" but measures the precision of a single label; no
criterion caps how lopsided the label counts may get (that cap is the 70%
rule in the notebook's `check_dataset()`), so the "Balance" area is covered
by title more than by content. Criteria 3, 4, and 5 each already carry a "Revised in unit
6" blockquote beneath the original (target unchanged, measurement instrument
reworded) — not documented further here, since that's Unit 6 content.

**How checked.** Each criterion names a number and a measurement method
specific enough to be checked by someone who didn't write it.

**A note on timing, named rather than hidden.** `criteria.md`'s edits at
`75df603` and `7c15780` land the same day as, and after, the training commit
`552f709`. The brief's framing — *"you're writing them before you train
anything... against a standard you set when you couldn't know the
answer"* — is only fully true of the four criteria untouched after training.
Criterion 2 was reworded with the first run's accuracy already visible,
per the repository's own disclosure.

**What this means for the next milestone.** Criterion 2's final form is what
Milestone 5's single training run is measured against — except that, per the
timing note above, part of that measurement standard was finalized after
seeing this unit's own result, not before.

**Terms from this milestone:** Held-out set (see Section 3).

**Milestone summary.** True now: five numbered criteria exist, covering all
five suggested areas, each with a stated target and reasoning. Still worth
flagging: one criterion's final wording postdates the only result it's
measured against so far.

**What this milestone taught:**
1. Five criteria for three required areas shows thoroughness, but a title
   is not a measurement: Criterion 3's "Class Imbalance" heading sits over a
   precision target, not a balance target.
2. "Before you see any results" is a claim about process, and git commit
   timestamps are the one record that can confirm or contradict it precisely.
3. Excluding the two largest labels from the per-label-performance criterion
   on purpose, because they're not the technically interesting ones, is a
   deliberate scoping decision worth stating as such.

---

### Milestone 4: Collect and Label 200 Examples

**The idea.** A model can only learn what the labels actually captured.
This is the longest, most consequential milestone in the unit, and this
repository's real implementation of it diverges from the brief's assumed
process more than any other milestone.

**Where the answers went.** `data/labels.csv`; `build_dataset.py`. A long
sequence of same-day commits (all 2026-10-19): `aa1642c` (create labels.csv,
add `unrelated` label) → `671bbf9` (Gemini JSON parsing, 21st label,
+172/−73) → `42b57cb` (**switch to RSS feed**, run started in background)
→ `9a16d38` (reworked into two phases: scrape, then label) → `a85dd10`
(first run, 104 rows) → `924d025` (switch second search to Hot sort,
append-not-replace) → `0eabd13` (second run, 77 rows — all duplicates of the
first) → `6fc4b7e` (merge) → `241c10c` (built a third search) → `9df0b4d`
(~100 more rows from the third search) → `79ef1b8` (**dedup run: 204 unique
rows, every row labeled**) → `02d4d46` (wrote the README "Dataset" section).

**Step: collect at least 200 public posts into one unsplit CSV.**
- What actually happened: **fully automated**, not manual. `build_dataset.py`
  scrapes r/estimators via Reddit's public Atom search feed
  (`search.rss?...`, no login) — built this way specifically because
  Reddit's `.json` endpoints now return HTTP 403 to logged-out callers. Three
  searches (Top/keywords, Hot/keywords, Top/`self:true`) were needed, not the
  two the brief implies, because the keyword query alone maxes out around
  105 posts regardless of sort or time window.
- Real yield: 281 raw rows collected, 77 removed as duplicates, leaving
  **204 unique posts** — above the 150-post floor, short of a clean 200,
  and only reached because of the third, broader search.
- Decision made:

  | Option | Chosen / why |
  |---|---|
  | Manual collection, as the brief describes | Not chosen |
  | Automated scraping via Reddit's public RSS/Atom feed | **Chosen** — Reddit's JSON API blocks logged-out requests; RSS needed no login or key |

**Step: label 20 posts cold, no help, no tools.**
- **Unresolved contradiction in this repository's own documentation**, named
  rather than silently resolved: `README.md` states in one place that the
  first 20 ("cold") rows' labels "were also assigned by Gemini," and in
  another place that those same labels "were not assigned by Gemini but were
  assigned by human opinion." The actual note text on those rows follows
  Gemini's generated format exactly (`"Highest semantic match of X% for
  [label]... due to [reason]"`), which weighs toward the first claim. This
  walkthrough reports both claims and the evidence, rather than picking one
  with false confidence — see Findings #4.

**Step: label the rest, optionally AI-pre-labeled and corrected.**
- What actually happened: **100% of the 204 labels** were produced by the
  Gemini API (`gemini-3.1-flash-lite`), via a `ModelPool` that rotates to a
  fallback model only on a permanent failure and retries transient ones.
  `README.md` states directly: *"This section does not claim a manual
  re-check of the labels."* This is a substantially larger AI role than the
  brief's "pre-label a batch, then read and correct every one" — see
  Findings #3.

**Step: keep a list of at least three hard cases.**
- Found, in `README.md`'s Dataset section, three cases verbatim (condensed):
  a career-mentorship post scored 90% `career_pathing_compensation_hiring`
  vs. 85% `general_construction_non_mechanical`; a bidding-software
  comparison scored 90% `enterprise_erp_bidding_platforms` vs. 85%
  `general_contractor_precon_coordinator`; an app review scored 95%
  `enterprise_erp_bidding_platforms` vs. 90%
  `software_benchmarking_technical_workflow`. All three were selected as
  "hardest" by **Gemini's own top-two score gap** (5 points each; the next
  closest gap in the dataset is 7), not by independent human judgment
  calling them ambiguous — a narrower meaning of "hard case" than the
  brief's instruction implies.

**Step: count labels, keep the largest under 70%.**
- Real, verified counts (programmatically confirmed against the actual CSV
  on 2026-10-31): largest label `industry_venting_operational_burnout` at
  55/204 (26.96%) — comfortably under the cap. But 10 of the 21 labels have 0
  or 1 example each (5 have none, 5 have exactly one). The 70% check, designed around a small taxonomy, doesn't surface
  this — see Findings #2.

**What this means for the next milestone.** `data/labels.csv`'s real shape
(204 rows, 21 sparse labels, near-100% machine-labeled) is exactly what
Milestone 5's `check_dataset()` and `describe_splits()` run against — and
what the training result in Milestone 5 has to be read in light of.

**Terms from this milestone:** Model pool (see Section 3).

**Milestone summary.** True now: 204 unique, labeled posts exist in one
CSV, under the 70% single-label cap, with three documented hard cases and a
disclosed (if internally inconsistent on one point) labeling process. Still
a real gap from the brief: collection was automated rather than manual, and
labeling was near-total Gemini output rather than a human-corrected batch.

**What this milestone taught:**
1. An API's access restrictions (Reddit blocking logged-out JSON requests)
   can force an entire method change (to RSS/Atom) that has nothing to do
   with the actual labeling task.
2. A keyword search has a ceiling independent of how it's sorted — widening
   the query (to `self:true` alone) was what actually closed the gap to 200,
   not a different sort order.
3. Disclosing "this is 100% machine-labeled, not spot-corrected" plainly is
   more useful to a reader than a vague "AI-assisted" phrase would have been.
4. A repository can disagree with itself about what actually happened in one
   of its own steps — and that's information worth preserving, not erasing
   by picking whichever account reads better.

**If stuck:** no equivalent note found for this milestone; the pipeline
was iterated on heavily (13 commits touch `build_dataset.py` in the Unit 5
window; 23 commits in total are dated 2026-10-19) rather than blocked and
abandoned.

---

### Milestone 5: Run the Training

**The idea.** Run the notebook, get a model and real metrics back. Most of
the work here is in having a dataset and criteria ready beforehand, not in
the run itself.

**Where the answers went.** `takemeter.ipynb` (sections 1-5), `results.json`,
`test_split.csv`. Real commit: `552f709` ("Run notebook sections 1-5 (unit 5
training)", 2026-10-19) — README +24/−9, `results.json` created (+630 lines),
`test_split.csv` created (+32 lines), notebook +4/−81. Followed by `97a3c14`
("How I Used AI" section, +10/−11).

**Step: commit `labels.csv` before touching anything else.** Confirmed —
`data/labels.csv` reaches its final 204-row state (`79ef1b8`) before the
training commit (`552f709`) that follows it.

**Step: open the notebook, confirm the kernel, run sections 1-4.**
- Real configuration used, sourced from the `552f709` commit and
  `README.md` — **not** from the current notebook file, whose own internal
  Section 4 cell was overwritten in Unit 6 (see Architecture and Findings
  #6): `BASE_MODEL = "distilbert-base-uncased"`, `EPOCHS = 3`,
  `LEARNING_RATE = 2e-5`, `BATCH_SIZE = 16`, `MAX_LENGTH = 128`, `SEED = 42`.
  Device: `cpu` (`torch 2.14.1+cpu`, no GPU) — confirmed directly in
  `results.json`.
- The notebook's own Section 2 (validation, `check_dataset()`) and Section 3
  (split reporting, `describe_splits()`) both exist and match the brief's
  described checks
  exactly: duplicate-post detection, label-spelling mismatches, the 70% cap,
  a size floor, and a <8-example "thin" warning on the test split.
- Split: 204 posts → 142 train / 31 val / 31 test, by plain random split —
  stratification wasn't possible, since several labels have exactly one
  example.

**Step: run section 5, commit `results.json` and `test_split.csv`.**
- Real result, this run, seed 42: **accuracy = 0.258, f1_macro = 0.061**,
  `n_test = 31`. Per-label scores show nonzero performance on only two
  labels (`career_pathing_compensation_hiring`,
  `industry_venting_operational_burnout` — not coincidentally, the two
  largest). The brief's own instruction for this milestone is explicit:
  *"Don't read the numbers closely yet; that's the next unit's work."* This
  walkthrough reports the numbers as committed, without diagnosing them —
  that diagnosis belongs to Unit 6, once its brief and milestones are
  supplied.

**What this means for the next milestone.** `results.json`'s single-seed
numbers are the "before" this project's eventual Unit 6 improvement is
measured against — confirmed by the existing (not yet documented)
`results_three_seeds_before.json` file already present in the repository.

**Terms from this milestone:** Epoch, Leakage (see Section 3).

**Milestone summary.** True now: one complete training run exists, with
metrics committed and the device/configuration recorded. Still a
placeholder, by the brief's own design: no interpretation of *why* the
numbers look the way they do — that's explicitly Unit 6's task.

**What this milestone taught:**
1. Recording the device a model trained on is itself part of the result —
   two machines can produce different numbers from the same seed, and that's
   a property of the training process, not an error.
2. A notebook's current cell contents are not a reliable record of what ran
   in an earlier unit, once a later unit edits the same cell in place.
3. Following the brief's instruction not to interpret a number yet is itself
   a disciplined choice, distinct from not noticing the number is low.

**If stuck:** no equivalent note found; the run appears to have completed on
the first attempt (one commit, no visible retry pattern in the git history).

---

### Milestone 6: Finish Documentation in the README

**The idea.** Close out the unit's write-up — the parts of `README.md` that
only make sense once the taxonomy, dataset, and a trained model all exist.

**Where the answers went.** `README.md`'s "How I Used AI" section. Real
commit: `97a3c14` (2026-10-19, +10/−11). Two further same-day commits,
`75df603` and `7c15780`, land after this one and revise `criteria.md` and
build `macro_f1_ci.py`/`kappa.py` — tooling this repository will need for
Unit 6, prepared at the tail end of the Unit 5 session rather than left for
later.

**Step: document how AI was used, with specific moments.** Two moments are
genuinely Unit 5 scope: (1) building `build_dataset.py` itself, discovering
Reddit's `.json` endpoint returns 403 to logged-out requests and switching to
the RSS/Atom feed, then discovering the keyword search's ~105-post ceiling
and adding a third, broader search to close the gap to 204; (2) the Gemini
labeling pipeline — the first preferred model was retired (404), its
replacement had a 20-request/day free-tier cap, so the script was rewritten
to rotate through a pool of fallback models instead of hardcoding one. A
third moment in the same section is explicitly headed "unit 6" in the
file — not reported in detail here, per this document's current scope.

**What this means for the next milestone.** The `macro_f1_ci.py` and
`kappa.py` scripts built here, at the close of Unit 5, are exactly the
tooling Criteria 2 and 4's "Revised in unit 6" measurement instruments
depend on — prepared before Unit 6 technically began.

**Terms from this milestone:** none new.

**Milestone summary.** True now: `README.md` documents two real, specific
Unit-5 AI-usage moments with concrete before/after detail (a blocked API
endpoint, a retired model). Still a placeholder: the full "How I Used AI"
section also contains Unit 6 content, not reported here.

**What this milestone taught:** building the next unit's measurement tooling
(`kappa.py`, `macro_f1_ci.py`) at the close of this unit, rather than at the
start of the next one, is a real, visible instance of preparing ahead —
worth noting as a pattern, not just an isolated commit.

---

## 7. Features

### Unit 5 Required Features

| Feature | What it requires | Current state |
|---|---|---|
| Two to four labels, each with a one-sentence definition and two example posts | A post should fit exactly one label almost every time | **Diverges from spec**: 21 labels exist, not 2-4 (Milestone 2) |
| A written decision rule for the hardest boundary | Found before labeling 200 things | **Done** — two decision rules written, for two separate boundaries (Milestone 2) |
| At least 200 labeled examples, one CSV, `text`/`label`/`note` columns | From one public community | **Done**, narrowly: 204 unique posts, `data/labels.csv`, correct columns (Milestone 4) |
| No label above 70% of the data | Prevents the model from learning to guess one label | **Passes the stated check** (max 26.96%) — but see Findings #2 for a different imbalance shape this check doesn't catch |
| A completed training run with saved metrics | — | **Done** — `results.json`, `test_split.csv`, seed 42, `cpu` (Milestone 5) |
| Five acceptance criteria, written before any result | All five are the author's own | **Done**, with one caveat: Criterion 2 was reworded after the first run's accuracy was already visible (Milestone 3) |

---

## 8. Setup

Full setup — venv, `test.py`, the notebook kernel, and the Gemini-key step
this repository's actual data pipeline needs but the course brief doesn't
mention — lives in the companion file
**[`ENVIRONMENT_SETUP.md`](ENVIRONMENT_SETUP.md)**. This section is a
pointer, not a duplicate.

In brief: fork and clone this repository, create a `.venv`, run
`pip install -r requirements.txt` and `python test.py`, open
`takemeter.ipynb` with this project's `.venv` kernel selected. No API key or
account is required for any of that. An API key *is* required only if
reproducing `build_dataset.py`'s data-collection step specifically — see
`ENVIRONMENT_SETUP.md` for exactly why and how.
