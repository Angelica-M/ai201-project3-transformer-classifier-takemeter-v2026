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
> **Scope:** this file covers **Unit 5** (build the dataset, train once, write
> the criteria) and **Unit 6** (baseline, three seeds, staff agreement,
> verdicts, one improvement). Unit 5 entries were written first; Unit 6 entries
> and sections were appended afterwards and earlier Unit 5 entries were not
> rewritten, except to replace "Unit 6 is not documented yet" pointers with
> pointers to the Unit 6 entries.

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
| **Unit 5** | Design the label taxonomy, collect and label at least 200 real examples, run one training pass, and commit five acceptance criteria — written before any result exists. |
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

### Unit 6 — Testing a Fine-Tuned Model

Unit 5 ended with a trained model and five acceptance criteria written
before any result existed. Unit 6 builds nothing new: it **tests** that model,
and for the first time something other than the author can say the work is
wrong. Three things test it:

1. **A baseline** — a general model doing the same job with no training at
   all, which shows whether fine-tuning bought anything.
2. **A confusion matrix** — a grid of which labels the model mixes up, which
   usually points at a boundary the taxonomy never settled.
3. **A staff reference annotation** — the same 30 posts, labeled
   independently by staff under a taxonomy staff wrote.

The unit also retrains on three different random splits instead of one,
because a single split is one draw, not a measurement. Disagreeing with staff
costs nothing and neither does missing one's own criteria; what is graded is
adjudication — arguing which label is right on a disputed post, and naming the
cause of every missed criterion.

**What is delivered in this unit:** a baseline run; three trials at three
seeds; a confusion matrix typed as a table; an agreement report against the
staff set; a verdict and diagnosis for each of the five criteria; one
improvement measured before and after; and a closing write-up of what is
still broken.

**Why this matters (the brief's framing):** anyone can report an accuracy
number. The transferable skill is saying what it means — where the model
fails, whether the failure is in the model or the labels, and whether it
could be trusted on unseen data.

### Learning Objectives (Unit 6)

By the end of Unit 6, the goal is to be able to:

1. Compare a fine-tuned model against a zero-shot baseline and interpret the
   gap.
2. Read a confusion matrix and name the distinction the model failed to
   learn.
3. Measure agreement between two annotators and adjudicate each
   disagreement.
4. Test with repeated trials across different random splits rather than one.
5. Trace a failure back to a property of the labels or the data.

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

**Unit 6 tooling** (`baseline.py`, `agreement.py`, `kappa.py`, `macro_f1_ci.py` and the
three-seed loop in `takemeter.ipynb` section 6 onward) is pictured separately,
under "Architecture, Unit 6 Additions" below.

### Goals (Unit 6)

By the end of Unit 6, the project demonstrates the ability to:

- Compare a trained model against an untrained one and say what the gap means.
- Read a confusion matrix and name the boundary the model did not learn.
- Compare its own labels against someone else's and argue a disagreement.
- Test a model with repeated trials rather than one split.

### Architecture, Unit 6 Additions

Unit 6 adds a testing layer on top of Phase B. Everything below reads files
Phases A and B already wrote (`data/labels.csv`, `test_split.csv`,
`results.json`) or the notebook's new three-seed output. The diagram was
generated by a script (widths computed, not typed) and 34 behavioural claims
in it were matched to lines of `baseline.py`, `takemeter.ipynb`,
`agreement.py`, `kappa.py` and `macro_f1_ci.py` as they exist now; 34 matched,
0 did not.

```
+--------------------------------------------------------------------------------------+
| PHASE C1  baseline.py -- zero-shot comparison on the Unit 5 test split               |
+--------------------------------------------------------------------------------------+
| Inputs: data/labels.csv (label set = the sorted unique labels IN the file, so        |
| 16, not 21), label_definitions.txt (missing -> bare label names, with a note),       |
| test_split.csv (missing -> random sample of labels.csv, with a warning).             |
| Model: facebook/bart-large-mnli via pipeline('zero-shot-classification'),            |
| hypothesis 'This post is {}.', candidate text = each label's definition.             |
| Highest-scoring candidate wins; no training, no API key.                             |
| score(): accuracy, per-label precision/recall/F1, macro F1 over the label list.      |
| --trained results.json adds a Trained column. Writes baseline_results.json.          |
+--------------------------------------------------------------------------------------+
                                            |
                                            v
+--------------------------------------------------------------------------------------+
| PHASE C2  takemeter.ipynb section 6 -- three seeds, one cell                         |
+--------------------------------------------------------------------------------------+
| SEEDS = [42, 7, 2024]. For each seed: split_dataset(df, seed) -> train_model()       |
| -> evaluate(). RUN_LABEL picks the output name: results_three_seeds_<label>.json.    |
| Each entry also records device, torch version and max_length (the 'after' run        |
| adds epochs, learning rate, batch size).                                             |
| spread_table() prints every measure per seed; stability_note(): accuracy spread      |
| above 0.10 = 'too small or too lopsided', above 0.05 = 'within the band'.            |
+--------------------------------------------------------------------------------------+
                                            |
                                            v
+--------------------------------------------------------------------------------------+
| PHASE C3  takemeter.ipynb section 7 -- confusion matrix                              |
+--------------------------------------------------------------------------------------+
| matrix_markdown() prints the FIRST seed's matrix (seed 42) as a markdown table;      |
| biggest_confusion() names the largest off-diagonal cell and the reverse count.       |
+--------------------------------------------------------------------------------------+

========================================================================================

+--------------------------------------------------------------------------------------+
| PHASE C4  agreement.py -- me versus staff on the same 30 posts                       |
+--------------------------------------------------------------------------------------+
| Staff set: data/staff_posts.csv joined to data/staff_labels.csv on `id`              |
| (or one combined text+label file with --staff). Mine: my_staff_labels.csv            |
| (text + label). Posts are paired on whitespace-normalised text.                      |
| Prints agreed / disagreed / rate and a disagreement table; the adjudication          |
| column is left empty on purpose. Writes agreement_results.json.                      |
+--------------------------------------------------------------------------------------+
                                            |
                                            v
+--------------------------------------------------------------------------------------+
| PHASE C5  kappa.py -- Criterion 4 (imports agreement.py's loader)                    |
+--------------------------------------------------------------------------------------+
| Same flags and same pairing as agreement.py. Cohen's kappa = (observed -             |
| chance) / (1 - chance); 2,000-resample bootstrap, 95% interval; target 0.70.         |
+--------------------------------------------------------------------------------------+

========================================================================================

+--------------------------------------------------------------------------------------+
| PHASE C6  macro_f1_ci.py -- Criterion 2, from the saved confusion matrices           |
+--------------------------------------------------------------------------------------+
| Reads a results file (default results_three_seeds_before.json), no retraining.       |
| Macro F1 over 5 fixed core labels (each with >= 10 posts in the dataset);            |
| 2,000-resample bootstrap, 95% interval. Target: every seed >= 0.60 AND               |
| standard deviation across seeds < 0.05.                                              |
+--------------------------------------------------------------------------------------+
                                            |
                                            v
+--------------------------------------------------------------------------------------+
| IMPROVEMENT LOOP  (Milestone 5)                                                      |
+--------------------------------------------------------------------------------------+
| Change one setting in the notebook's settings cell, set RUN_LABEL = 'after',         |
| rerun section 6 -> results_three_seeds_after.json, rerun macro_f1_ci.py with         |
| --results results_three_seeds_after.json, compare to the 'before' file.              |
+--------------------------------------------------------------------------------------+
```

**Two scripts were built at the end of Unit 5, not in Unit 6:** `macro_f1_ci.py`
and `kappa.py` (commits `75df603` and `7c15780`, see Unit 5, Milestone 6).
They belong to this layer because Criteria 2 and 4 are measured with them.
`baseline.py`, `agreement.py`, `test.py` and `label_definitions.txt` shipped in
the starter (initial commit `9772abf`), and the staff posts and taxonomy in
`66c0b04`; `data/staff_labels.csv` does not ship, since staff send it after the
student's own 30 labels are done. The author's contribution to the shipped
scripts is the content of `label_definitions.txt` and the inputs they were run
on.

**Phase C1's label set differs from the notebook's.** `baseline.py` scores over
the labels that occur in `data/labels.csv` (16), while the notebook's macro F1
is computed by scikit-learn over the labels present in the gold labels or the
predictions of a split. The two macro F1 values are therefore not directly
comparable; the README's "Baseline vs. Trained" section prints both (0.061 as
the notebook computes it, 0.042 over the same 16 labels).

---

## 3. Glossary

| Term | What it means | Where it shows up in TakeMeter |
|---|---|---|
| **Baseline** | A general model doing the task with no training; what the trained model has to beat. | `baseline.py`, `baseline_results.json`. Introduced in Unit 6, Milestone 1. |
| **Zero-shot** | Doing a task from instructions alone, with no training examples. | `pipeline("zero-shot-classification", model="facebook/bart-large-mnli")` in `baseline.py`; the label definitions are the only input. |
| **Precision** | Of the posts the model called label X, how many really were X. | `per_label[...]["precision"]` in `results.json`; Criteria 3 and 5. A label never predicted has precision 0 of 0, stored as 0.0. |
| **Recall** | Of the posts that really were X, how many the model found. | `per_label[...]["recall"]` in `results.json`. |
| **F1** | One number combining precision and recall for a label. | `per_label[...]["f1"]`; printed per label in the notebook's spread table. |
| **Macro F1** | The unweighted average of per-label F1 scores, so rare labels count as much as common ones. | `f1_macro` in each results file; Criterion 2 averages it over five fixed labels with `macro_f1_ci.py`. |
| **Confusion matrix** | A grid: rows are true labels, columns are the model's guesses, the diagonal is right answers. | Notebook section 7 prints it as a markdown table; stored as `confusion` in each results file. |
| **Seed** | The number that fixes which posts land in train, validation and test. | `SEEDS = [42, 7, 2024]` in notebook section 6; `SEED = 42` in section 2. |
| **Agreement** | How often two annotators give the same label to the same post. | `agreement.py`; `agreement_results.json`. |
| **Cohen's kappa** | Agreement corrected for the agreement two annotators would reach by chance. | `kappa.py`; Criterion 4. Chance agreement was 0.333 for 3 labels used evenly. |
| **Bootstrap confidence interval** | The middle 95% of a statistic recomputed on 2,000 resamples of the test posts, showing how much of the number is luck of the split. | `macro_f1_ci.py` and `kappa.py`; `N_BOOTSTRAP = 2000`. |
| **Adjudication** | Arguing, from the written rules, which of two disputed labels is right and why. | The README's Agreement Report; it has no entries because there were no disagreements. |
| **Majority-class guess** | The accuracy of always naming the most common label in the test split. | 0.258 on the seed-42 test split (8 of 31). Trained model seed 42 also scored 0.258. |
| **Constant guesser** | A model whose predictions collapse onto one or two labels regardless of the post. | Seed 2024, before improvement: `career_pathing_compensation_hiring` for all 31 posts. |
| **RUN_LABEL** | A string in notebook section 6 that names the output file so a re-run does not overwrite earlier evidence. | `"before"` produced `results_three_seeds_before.json`; `"after"` produced `results_three_seeds_after.json`. |
| **Staff set** | The 30 posts and the staff's labels, written under a three-label taxonomy unrelated to the 21 HVAC-estimating labels. | `data/staff_posts.csv`, `data/staff_labels.csv`, `data/staff_taxonomy.md`. |
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


**Unit 6 commands (testing the trained model).** Run these after the steps
above. Captured output for each is in the milestone named beside it.

```bash
python baseline.py --data data/labels.csv --trained results.json   # Unit 6, Milestone 1
# notebook: section 1, then section 6 with RUN_LABEL set, then section 7       # Milestone 2
python agreement.py                                                # Milestone 3
python kappa.py                                                    # Milestone 3
python macro_f1_ci.py                                              # Milestone 4
python macro_f1_ci.py --results results_three_seeds_after.json     # Milestone 5
```
- `baseline.py` takes a few seconds per post on CPU and is slow by design; the
  zero-shot model is about 1.6 GB on first use and is already downloaded in
  this working copy (`test.py` reports "Baseline model: already downloaded").
- Section 6 of the notebook trains three seeds in one cell: about 3.6 minutes
  at 3 epochs and about 15.6 minutes at 15 epochs on this CPU.
- Run `agreement.py` only after `my_staff_labels.csv` is saved and
  `data/staff_labels.csv` has been received; see `ENVIRONMENT_SETUP.md`
  section 6.

---

## 5. Findings and Lessons Learned

Fifteen findings, numbered 1-15 (1-8 from Unit 5, 9-15 from Unit 6), each stated with the evidence behind it.

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

**Unit 6 findings (9-15).** Appended after the Unit 5 findings; numbering continues.

**9. A spread check can pass while the model is a constant guesser, so read
the predictions, not only the spread.** Before the improvement, accuracy
across seeds was 0.258, 0.226, 0.290, a spread of 0.065, which the notebook's
`stability_note()` calls "within the band" (above 0.05, not above 0.10).
The confusion matrices showed why that number was misleading: seed 2024
predicted `career_pathing_compensation_hiring` for all 31 test posts, seed 7
for 28, and seed 42 used only two labels. Accuracy moved with which big label
a seed favoured and how many such posts its split held. The model's average
winning probability was 0.09 to 0.12, whether right or wrong, against 0.048
for a uniform guess over 21 labels.

**10. A zero-shot baseline can do worse with long label definitions than with
bare label names.** On the same 31 test posts `facebook/bart-large-mnli`
scored accuracy 0.129 (4 of 31) with the author's definitions as the
candidate text and 0.355 (11 of 31) with bare label names; the two runs agreed
on 4 of 31 posts. With definitions it predicted
`software_vendor_sales_representative` for 19 of 31 posts. The README gives a
plausible, untested reason (long definitions let the model score on shared
vocabulary). The consequence for the comparison: the 3-epoch trained model
(0.258) beat the definitions baseline but lost to the bare-names baseline.

**11. A precision criterion has no verdict when nothing is predicted, and the
generous reading hides that.** For Criterion 3 (precision of
`general_construction_non_mechanical`) and Criterion 5 (precision at
confidence 0.80 or more) the before-run had zero predictions on every seed, so
precision was 0 of 0. Reading that as "nothing it predicted was wrong" would
pass a model that predicts nothing. Both criteria were revised (commit
`44805d1`) so that no predictions counts as MISSED; neither target number
moved and both verdicts are MISSED under the original and revised wording.

**12. A perfect agreement score from a copied answer key measures nothing.**
The first `data/staff_labels.csv` committed (`ba4b184`) was byte-for-byte the
author's own `my_staff_labels.csv` at `d3a3178` (checked with
`git show <hash>:<file>` and `cmp`; identical file size and modification time
too). `agreement.py` reported 30 of 30 and kappa 1.000 on it. That result was
discarded, the file replaced by the real key (`8932fa1`, columns `id`,
`label`, `borderline`, `why`), and the real run also gave 30 of 30. Both facts
are recorded in the README's Agreement Report; the copied file remains in git
history.

**13. A number estimated for an explanation needs the same verification as a
number reported as a result.** The diagnosis said training was about 33
optimizer steps, computed from 173 training posts. The split is
142 train / 31 validation / 31 test, which is 9 steps per epoch, about 27 for
3 epochs and about 135 for 15. The error was found in the final README audit
and corrected in `6e2a454`; three earlier commit messages (`3900c0e`,
`182c835`, `bca35b4`) keep the old figure. The conclusion did not change.

**14. Training for longer helped, and changed no verdict.** With `EPOCHS`
raised from 3 to 15 (nothing else changed), accuracy moved from
0.258 / 0.226 / 0.290 to 0.419 / 0.419 / 0.548 and the model predicted four
labels on every seed instead of one or two. Criteria 1, 2, 3 and 5 stayed
MISSED, and no prediction reached 0.80 confidence. The accuracy spread across
seeds widened from 0.065 to 0.129, which the notebook flags as more than ten
points. The README concludes that training length was part of the cause and
that thin data is the remaining limit.

**15. Agreement on a three-label task says nothing about the consistency of a
21-label dataset labeled by a different annotator.** The 30 of 30 result is
against staff's `analysis` / `hot_take` / `reaction` taxonomy. The 204
training labels were written by Gemini, with no second human annotator, so the
agreement report cannot separate a labeling problem from a model problem for
the HVAC-estimating labels. The README states this and names the one check
that would (hand-labeling a random 30 posts blind to Gemini and computing
kappa), which was not done.

---

## 6. Milestones

Documented retroactively from this repository's real git history (67
commits in total at the time of the audit, from `git log --oneline | wc -l`,
the last being `f16eeab`, which committed these two files;
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
reworded) — documented under Unit 6, Milestone 4.

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
  that diagnosis is in Unit 6, Milestone 4.

**What this means for the next milestone.** `results.json`'s single-seed
numbers are the "before" this project's eventual Unit 6 improvement is
measured against — confirmed by `results_three_seeds_before.json`, documented under
Unit 6, Milestone 2.

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
third moment in the same section is headed "unit 6" in the file and is
covered under Unit 6, Milestone 6.

**What this means for the next milestone.** The `macro_f1_ci.py` and
`kappa.py` scripts built here, at the close of Unit 5, are exactly the
tooling Criteria 2 and 4's "Revised in unit 6" measurement instruments
depend on — prepared before Unit 6 technically began.

**Terms from this milestone:** none new.

**Milestone summary.** True now: `README.md` documents two real, specific
Unit-5 AI-usage moments with concrete before/after detail (a blocked API
endpoint, a retired model). Not covered here: the full "How I Used AI"
section also contains Unit 6 content, reported under Unit 6, Milestone 6.

**What this milestone taught:** building the next unit's measurement tooling
(`kappa.py`, `macro_f1_ci.py`) at the close of this unit, rather than at the
start of the next one, is a real, visible instance of preparing ahead —
worth noting as a pattern, not just an isolated commit.

---

**Unit 6 milestones.** Unit 5 entries above are headed "Milestone N"; Unit 6 entries are headed
"Unit 6, Milestone N". All Unit 6 commits were made on 2026-10-27 (first
`044a0a4` at 14:55, last `6e2a454` at 21:26): 31 commits inclusive, from
`git rev-list --count 044a0a4^..6e2a454`. Milestone 6 is documented from the
milestone text the author supplied earlier in the working session
("Finish the write-up and make the call on what you didn't fix"), since it was
not part of the Unit 6 brief pasted for this file. Every per-file count below
comes from `git show --numstat <hash>`.

### Unit 6, Milestone 1: Run the Baseline

**The idea.** A trained model's accuracy means nothing without a reference
for how hard the task is. A general model reading the label definitions and
nothing else is a fair opponent; if it matches the trained model, the training
added nothing, which is a finding and not a failure. The step avoids crediting
fine-tuning for accuracy a free model would have reached, and avoids comparing
two models on different posts.

**Where the answers went.** The commits do not map one per step.

| Commit | What it holds | Numstat |
|---|---|---|
| `044a0a4` | The 21 definitions and the first baseline run, together | `baseline_results.json` +267/−0; `label_definitions.txt` +26/−7 |
| `8083f71` | The written prediction | `README.md` +2/−0 |
| `1b8a166` | The Baseline vs. Trained table | `README.md` +20/−4 |
| `19207eb` | A plain-language guide to `baseline.py` (not a step in the brief) | `baseline-py_output_explanation.md` +207/−0 |
| `558f6bd` | Correction to that guide's AI-use statements | `baseline-py_output_explanation.md` +2/−2 |
| `9bf31e4` | The bare-names baseline run | `README.md` +10/−0; guide +26/−2; `baseline_results_bare_names.json` +267/−0 |
| `f4c8ebb` | A revert of `9bf31e4` | `README.md` +0/−10; guide +2/−26; `baseline_results_bare_names.json` +0/−267 |
| `a9a7963` | A revert of that revert, restoring the run | `README.md` +10/−0; guide +26/−2; `baseline_results_bare_names.json` +267/−0 |

The definitions file and the first run share one commit, and the fourth step
took three commits (do, undo, redo), described below.

**Step: put the label definitions in `label_definitions.txt`.**
- What changed: the shipped file held three example definitions; the author's
  21 were copied in from the README's Label Taxonomy. Checked now: all 21
  definitions are identical, character for character, to the README's
  `**Definition:**` lines. First lines of the file:
```
division_23_full_mechanical_hvac: Posts covering entire building mechanical and environmental control sys...
sub_trade_specialty_sheet_metal_vs_hydronics: Discussions strictly focused on granular trade-specific tak...
mep_package_mechanical_plumbing_electrical: Posts resolving scope overlaps, omissions, or package coordin...
(21 lines in all; each line cut here at 105 characters for width)
```
- Why: the definitions are the baseline's entire input, so the baseline is
  only as fair as they are.

  | Option | Chosen / why |
  |---|---|
  | Copy the README definitions verbatim | **Chosen** — the baseline is judged on exactly what was written |
  | Write shorter, baseline-friendly definitions | Not chosen — it would measure a different taxonomy from the one the model was trained on |
  | Skip the file (bare label names) | Run on purpose as a second experiment, see below |

**Step: run `python baseline.py`.**
- What it does (Architecture, Phase C1): scores the Unit 5 test split with
  `facebook/bart-large-mnli`. It chooses among the **16 labels present in the
  data**, not 21, because `baseline.py` builds its label list from
  `data/labels.csv`. Output captured when this file was audited (the command
  is the one used for the committed `baseline_results.json`, with `--out`
  pointing elsewhere so the committed file was not overwritten; the re-run's
  scores and all 31 predictions are identical to the committed
  `baseline_results.json`, checked by comparing the two files):
```
Using your definitions from label_definitions.txt.
Reading the held-out posts from test_split.csv — the same ones your
trained model was scored on.

Classifying 31 posts across 16 labels.
A few seconds each — this is what no fine-tuning costs at runtime.

Loading facebook/bart-large-mnli (first run downloads ~1.6 GB)…
Device set to use cpu

accuracy  0.129
macro F1  0.060

  bid_strategy_scope_gaps_gc_relations F1 0.000   (n=2)
  career_pathing_compensation_hiring F1 0.000   (n=7)
  commercial_institutional_industrial F1 0.000   (n=1)
  emerging_ai_automation_tools F1 0.000   (n=2)
  enterprise_erp_bidding_platforms F1 0.000   (n=1)
  general_construction_non_mechanical F1 0.000   (n=4)
  general_contractor_precon_coordinator F1 0.500   (n=2)
  general_pdf_cad_digitizers F1 0.000   (n=0)
  industry_venting_operational_burnout F1 0.364   (n=8)
  multi_family_residential_tract_housing F1 0.000   (n=0)
  public_works_prevailing_wage_bidding F1 0.000   (n=2)
  senior_b2b_mechanical_estimator_chief_estimator F1 0.000   (n=0)
  software_benchmarking_technical_workflow F1 0.000   (n=1)
  software_vendor_sales_representative F1 0.100   (n=1)
  specialized_mechanical_estimating_databases F1 0.000   (n=0)
  unrelated        F1 0.000   (n=0)

Paste this into your README under **Baseline vs. Trained**:

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.129 | 0.258 | +0.129 |
| Macro F1 | 0.060 | 0.061 | +0.001 |
| F1 — `bid_strategy_scope_gaps_gc_relations` | 0.000 | 0.000 | +0.000 |
| F1 — `career_pathing_compensation_hiring` | 0.000 | 0.308 | +0.308 |
| F1 — `commercial_institutional_industrial` | 0.000 | 0.000 | +0.000 |
| F1 — `emerging_ai_automation_tools` | 0.000 | 0.000 | +0.000 |
| F1 — `enterprise_erp_bidding_platforms` | 0.000 | 0.000 | +0.000 |
| F1 — `general_construction_non_mechanical` | 0.000 | 0.000 | +0.000 |
| F1 — `general_contractor_precon_coordinator` | 0.500 | 0.000 | -0.500 |
| F1 — `general_pdf_cad_digitizers` | 0.000 | 0.000 | +0.000 |
| F1 — `industry_venting_operational_burnout` | 0.364 | 0.364 | +0.000 |
| F1 — `multi_family_residential_tract_housing` | 0.000 | 0.000 | +0.000 |
| F1 — `public_works_prevailing_wage_bidding` | 0.000 | 0.000 | +0.000 |
| F1 — `senior_b2b_mechanical_estimator_chief_estimator` | 0.000 | 0.000 | +0.000 |
| F1 — `software_benchmarking_technical_workflow` | 0.000 | 0.000 | +0.000 |
| F1 — `software_vendor_sales_representative` | 0.100 | 0.000 | -0.100 |
| F1 — `specialized_mechanical_estimating_databases` | 0.000 | 0.000 | +0.000 |
| F1 — `unrelated` | 0.000 | 0.000 | +0.000 |

Wrote <scratch dir>/baseline_rerun.json. Commit it.

Before you look at your trained model's numbers: write one sentence
predicting where it will beat this. Milestone 1 asks for it, and a
prediction made after the fact isn't one.
```

**Step: write a prediction, then produce the comparison table.**
- The prediction is in the README (commit `8083f71`); it predicted the
  trained model would reach 0.35 to 0.45 accuracy. The table (`1b8a166`) put
  the trained model at 0.258 against the baseline's 0.129.
- **Timing caveat, stated plainly:** the brief asks for the prediction
  "before you look." The repo cannot show that. `results.json` with the
  trained accuracy of 0.258 was committed on 2026-10-19 (`552f709`); the
  baseline results were committed at 14:55 on 2026-10-27 and the prediction at
  15:16 the same day. Whether the author looked at the trained number before
  writing is not verifiable from the repository. The prediction's accuracy
  range turned out to be wrong (0.35 to 0.45 against 0.258). Its wording also
  called the model a "LoRA model" with an "adapter", which is inaccurate (see
  Findings #7); the text was left as written and annotated.

**Step: run it once on purpose without definitions.**
- `python baseline.py --data data/labels.csv --definitions no_definitions_on_purpose.txt --out baseline_results_bare_names.json`
  (a path that does not exist, so the script falls back to bare label names).
- The history of this step is untidy and is reported as it was: the run was
  committed (`9bf31e4`), then reverted (`f4c8ebb`) after a misreading of an
  instruction that only one baseline run was wanted, then restored
  (`a9a7963`) once it was clear that both runs were wanted. A revert-of-a-
  revert was used instead of rewriting history because earlier commits had
  already been pushed.

  | Option | Chosen / why |
  |---|---|
  | `git revert` then `git revert` the revert | **Chosen** — adds commits, rewrites nothing that was pushed |
  | `git reset --hard` / force-push | Not chosen — would rewrite pushed history |

**Comparison, same 31 held-out posts (seed-42 test split):**

| Run | Accuracy | Macro F1 | Note |
|---|---|---|---|
| Baseline, with the author's definitions | 0.129 (4/31) | 0.060 | the baseline that is reported |
| Baseline, bare label names | 0.355 (11/31) | 0.215 | the two baseline runs agreed on 4 of 31 posts |
| Trained model, 3 epochs | 0.258 (8/31) | 0.061 as the notebook prints it; 0.042 over the same 16 labels | different label sets, see Architecture, Unit 6 Additions |

**What this means for the next milestone.** The trained model is ahead of the
definitions baseline on accuracy only, tied on macro F1, and behind the
bare-names baseline on both. Whether the trained result is stable is what
Milestone 2 tests.

**Terms introduced:** Baseline, Zero-shot, Precision, Recall, F1, Macro F1,
Majority-class guess (see Section 3).

**Milestone summary.** True now: both baselines exist and are committed, the
README table compares them with the trained model on the same posts, and the
comparison is honest that the trained model did not beat the bare-names
baseline. Still open: the untested reason why definitions hurt the baseline.

**What this milestone taught:**
1. A baseline is only a fair opponent if it is scored on the same posts; the
   script reads `test_split.csv` for that reason.
2. More detailed instructions do not guarantee a better zero-shot result: the
   long definitions scored 0.129 against 0.355 for bare names.
3. A prediction is only evidence of foresight if the numbers were unseen when
   it was written, and a commit timeline can show when that was impossible.
4. After a bad commit is pushed, `git revert` fixes it without rewriting
   history; an untidy history is a better record than a rewritten one.

**Completion criteria.** No "before you move on" list was supplied for this
milestone; its own steps were checked instead.

| Step | Status |
|---|---|
| Definitions in `label_definitions.txt`, copied from the README | TRUE (21 of 21 identical) |
| `python baseline.py` run on `test_split.csv` | TRUE (`held_out: true`, `n: 31` in `baseline_results.json`) |
| One-sentence prediction written before looking | Not verifiable (see timing caveat) |
| Comparison table produced and pasted into the README | TRUE |
| `baseline_results.json` committed | TRUE (`044a0a4`) |
| Run with and without definitions | TRUE (`baseline_results_bare_names.json`, `a9a7963`) |

---

### Unit 6, Milestone 2: Retrain Three Times and Read the Matrix

**The idea.** A score depends on which posts landed in the test split, so one
split is one draw and not a measurement. Three seeds give three draws, and the
spread says whether the result is real. The confusion matrix then shows which
labels get mixed up and in which direction, which a single accuracy number
hides.

**Where the answers went.** One commit: `75899d2` (`README.md` +69/−11,
`results_three_seeds_before.json` +1910/−0). It holds the three-seed run, the
spread table, the confusion matrix and the README's run log together; the
steps did not get separate commits. The notebook was executed from a script
that ran the same code cells, so the committed notebook has no stored
outputs for this run.

**Step: run section 1, then section 6 with `RUN_LABEL = "before"`.**
- What changed: `results_three_seeds_before.json` (three entries: seeds 42, 7
  and 2024), each recording device `cpu`, torch version and `max_length`.
  Three seeds at 3 epochs took about 3.6 minutes in total (README, "Run Log —
  Before").
- Why: the cell runs all three seeds in one execution on purpose, so a
  comparable set exists or none does.

  | Option | Chosen / why |
  |---|---|
  | Three seeds in one cell, output named by `RUN_LABEL` | **Chosen** — the notebook's design; a later re-run cannot overwrite the "before" file |
  | Edit `SEED` by hand three times | Not chosen — easy to overwrite or mix results |
  | One split with a larger test set | Not possible: 204 posts total |

- Real values, read from `results_three_seeds_before.json`:
```
seed   42: accuracy 0.258  macro F1 0.061  n_test 31  device cpu
seed    7: accuracy 0.226  macro F1 0.059  n_test 31  device cpu
seed 2024: accuracy 0.290  macro F1 0.056  n_test 31  device cpu
accuracy spread (max - min): 0.065
```

**Step: look at the spread.**
- The notebook's `stability_note()` applied to those values (the function
  from notebook section 6, run on the committed file):
```
Accuracy moved 0.065 across seeds. Enough that a criterion landing within that band can't really be called either way.
```
- Reading it correctly needs the predictions, not only the spread: see
  Findings #9. Seed 2024 predicted one label for all 31 posts.

**Step: run section 7 and name the biggest off-diagonal number.**
- The seed-42 matrix is a 21 by 21 markdown table in the README ("Confusion
  matrix"); it is not repeated here. The notebook's `biggest_confusion()`
  applied to that matrix:
```
Biggest confusion: 5 real `career_pathing_compensation_hiring` posts called `industry_venting_operational_burnout`.
The other direction: 2 real `industry_venting_operational_burnout` posts called `career_pathing_compensation_hiring`.
```
- The README's one-sentence reading: the direction reflects whichever of the
  two largest labels a seed favoured, not a learned boundary.

**What this means for the next milestone.** A model that is close to a
constant guesser makes the labeling question urgent: is the data unlearnable
or is it inconsistent? Milestone 3's staff comparison is the only available
evidence on consistency.

**Terms introduced:** Seed, Confusion matrix, Constant guesser, RUN_LABEL
(see Section 3).

**Milestone summary.** True now: three comparable result sets exist for the
unchanged 3-epoch configuration, with a typed matrix and an honest reading of
the spread. Still open: why the model collapses onto two labels (diagnosed in
Milestone 4).

**What this milestone taught:**
1. A spread under the notebook's ten-point threshold does not make a measure
   trustworthy; a constant guesser can have a small spread.
2. A seed changes which posts land in the test split, so the three accuracies
   mostly track how many posts of the favoured label each split holds.
3. Running the three seeds from `RUN_LABEL`-named output keeps the "before"
   file safe for the later comparison.

**Completion criteria.** No "before you move on" list was supplied; its own
steps were checked.

| Step | Status |
|---|---|
| Section 1 run, device recorded | TRUE (`device: cpu` in all three entries) |
| Section 6 run with `RUN_LABEL = "before"`, file written | TRUE (`results_three_seeds_before.json`, committed in `75899d2`) |
| All three sets of numbers written down | TRUE (README run log and spread table) |
| Spread examined and noted | TRUE (0.065; README discusses it) |
| Matrix typed as a markdown table, biggest off-diagonal named | TRUE (README, "Confusion matrix") |

---

### Unit 6, Milestone 3: Check Yourself Against the Staff Labels

**The idea.** Staff labeled 30 posts under a taxonomy staff wrote
(`data/staff_taxonomy.md`: `analysis`, `hot_take`, `reaction`, and four
decision rules). The student labels the same posts under those definitions,
not their own 21 HVAC-estimating labels. That swap is the point: applying
someone else's written rule has a checkable answer, and it is the only evidence
about whether the student's own labeling was consistent. The step avoids
grading invented definitions that nobody can contradict.

**Where the answers went.** The commits do not map cleanly one per step, and
include one that was wrong.

| Commit | What it holds | Numstat |
|---|---|---|
| `e5556d9` | Message says "Labeled 30 staff labels"; the file has 30 posts and **no labels** | `my_staff_labels.csv` +31/−0 |
| `d3a3178` | The 30 labels actually filled in by hand | `my_staff_labels.csv` +30/−30 |
| `ba4b184` | A `data/staff_labels.csv` that was a **copy of the student's own labels** | `data/staff_labels.csv` +31/−0 |
| `8932fa1` | The real staff file replaces it | `data/staff_labels.csv` +31/−31 |
| `7657aab` | `python agreement.py` output | `agreement_results.json` +7/−0 |
| `ac6c835` | The README Agreement Report | `README.md` +27/−18 |

Three more commits fall in the same session but belong to a side question
about LoRA, not to a step: `02a6c19` and `f5016d6` (the LoRA/DistilBERT guide:
`LoRA_vs_BERT_guide.md` +89/−0, then replaced by
`DistilBERT_LoRA_and_full_fine_tuning_explained.md` +414/−0) and `ab26ff5`
(`criteria.md` +11/−5, which corrects "LoRA" wording and adds the Criterion 4
revision).

**Step: label all 30 posts by hand under the staff definitions.**
- What changed: `my_staff_labels.csv`, 30 rows, `text` and `label` columns;
  the text is identical to `data/staff_posts.csv` for all 30 rows (checked).
  The labels are 10 `analysis`, 10 `hot_take`, 10 `reaction`. They were typed
  by the author; the AI assistant created the empty scaffold with the post
  text and did not label.
- Why the scaffold: the two files are matched on text, so a retyped or
  trimmed post would not match.

**Step: obtain the staff key only after the labels are saved.**
- The ordering is recorded in git: `d3a3178` (20:16) holds the labels; the
  first `data/staff_labels.csv` arrived at `ba4b184` (20:21) and the real one at
  `8932fa1` (20:29).
- What actually happened, in order: the first file had columns `text`,`label`
  only, and `agreement.py`'s default mode (posts joined to labels on `id`)
  stopped with `has no 'id' column`. The alternative `--staff` mode (one file
  with text and label) ran and reported 30 of 30. A byte comparison showed the
  file was identical to the student's own labels, so that result was
  discarded, not committed, and not used. The file was replaced with the
  real key, which has columns `id`, `label`, `borderline`, `why`, and the
  default mode then ran.

  | Option | Chosen / why |
  |---|---|
  | Replace the file with the real key and re-run | **Chosen** |
  | Keep the first result because it "passed" | Rejected — a copy of one's own answers cannot disagree |
  | Rewrite `ba4b184` out of history | Not done — the record of the mistake stays |

**Step: run `python agreement.py` and `python kappa.py`.**
```
Compared 30 posts.

  agreed      30
  disagreed   0
  agreement   100%

You matched staff on every post. Say so in your README — and say
whether you think the set was easy or your rule is genuinely tight.

Wrote agreement_results.json. Commit it.

Then sort each disagreement into one of three piles:
  (a) the rule covered it and you applied it loosely — a consistency problem
  (b) the rule genuinely doesn't say — a gap in the definitions
  (c) the rule is ambiguous here and your reading is defensible — argue it

(b) and (c) are the interesting ones, and (c) is a legitimate win.
```
```
Compared 30 posts.

  observed agreement           1.000   (same as agreement.py's rate)
  agreement expected by chance 0.333
  Cohen's kappa                1.000   95% bootstrap interval [1.000, 1.000]
  Criterion 4 target           >= 0.7   ->   MET
```
- 30 compared, 30 agreed, 0 disagreed; kappa 1.000, chance agreement 0.333,
  bootstrap interval [1.000, 1.000] (nothing to resample).
- Staff marked 7 of the 30 posts as hard (`borderline = yes`); all 7 matched.

**Step: adjudicate each disagreement and sort into piles (a), (b), (c).**
- There were no disagreements, so there is nothing to adjudicate and every
  pile is empty (0 / 0 / 0). The README says so and does not invent entries.
  In place of disagreements it tabulates the 7 staff-flagged borderline posts
  with the rule each turns on.
- What the result can and cannot show is in Findings #15.

**What this means for the next milestone.** Criterion 4 can be judged, but
only against its revised wording; and the agreement report offers no label
pair to line up with the confusion matrix, so the diagnosis in Milestone 4
cannot lean on it for the HVAC labels.

**Terms introduced:** Agreement, Cohen's kappa, Adjudication, Staff set (see
Section 3).

**Milestone summary.** True now: the 30 labels are committed before the real
key, the agreement report is run and committed, and the README records it. Still
open: nothing in the repo can show that the 30 of 30 reflects an independent
match, beyond the ordering above.

**What this milestone taught:**
1. A commit message is a claim: `e5556d9` says "Labeled 30 staff labels" over
   a file with none.
2. An agreement score is only meaningful if the two sets of labels were
   written independently; comparing the files byte for byte takes seconds.
3. A script's default mode and its alternative mode can accept the same
   data but mean different things; `--staff` hid the problem the default
   mode had flagged.
4. A result from a three-label task does not carry over to a 21-label task.

**Completion criteria.** No "before you move on" list was supplied; the steps
were checked.

| Step | Status |
|---|---|
| Read the staff taxonomy; label all 30 posts; save `my_staff_labels.csv` with `text` and `label` | TRUE (`d3a3178`) |
| Get `data/staff_labels.csv` only after the 30 are saved | TRUE by commit order; whether it was unopened before then is not verifiable |
| Run `python agreement.py` | TRUE (30 of 30) |
| Commit `agreement_results.json` | TRUE (`7657aab`) |
| Adjudicate each disagreement | TRUE vacuously (0 disagreements) |
| Sort disagreements into piles | TRUE vacuously (0 / 0 / 0) |

---

### Unit 6, Milestone 4: Call Each Criterion and Diagnose Every Miss

**The idea.** Say met or missed against the previous unit's targets, then
work out why. The verdict looks easy and is where people go wrong, because
"met" is more comfortable and there is always a generous reading of three
numbers. The step avoids reading a target as met when it only held on some
seeds, or when nothing was measured.

**Where the answers went.** Five commits, one per step.

| Commit | Step | Numstat |
|---|---|---|
| `4c14295` | Run log verdicts | `README.md` +12/−10 |
| `3900c0e` | Diagnoses | `README.md` +22/−0 |
| `0103ab3` | Agreement report as evidence | `README.md` +12/−0 |
| `9493940` | Broken or just missed | `README.md` +14/−0 |
| `44805d1` | Revisions recorded under the originals | `criteria.md` +9/−1 |

**Step: fill in the run log and write MET or MISSED for each criterion.**
- The five rows (targets from `criteria.md`, before-run values):

| # | Criterion | Target | Seed 42 / 7 / 2024 | Verdict |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.65 on every seed | 0.258 / 0.226 / 0.290 | MISSED |
| 2 | Macro F1, 5 core labels | ≥ 0.60 every seed, standard deviation < 0.05 | 0.000 / 0.000 / 0.000 | MISSED |
| 3 | Precision, `general_construction_non_mechanical` | ≥ 0.75 on every seed | 0.000 each (never predicted) | MISSED |
| 4 | Kappa against staff | ≥ 0.70 (revised wording) | 1.000 (not a model measure) | MET |
| 5 | Precision at confidence ≥ 0.80 | ≥ 0.85 on every seed | no prediction reached 0.80 (n = 0) | MISSED |

- Captured output behind rows 2, 3 and 5 (`python macro_f1_ci.py`, then the
  one-line `python -c` checks documented in `criteria.md`):
```
Scoring 5 labels: general_construction_non_mechanical (23), software_benchmarking_technical_workflow (16), emerging_ai_automation_tools (16), bid_strategy_scope_gaps_gc_relations (12), enterprise_erp_bidding_platforms (10)
  seed | test posts | macro F1 | 95% bootstrap interval
--------------------------------------------------------
    42 |         31 |    0.000 | [0.000, 0.000]
     7 |         31 |    0.000 | [0.000, 0.000]
  2024 |         31 |    0.000 | [0.000, 0.000]
mean macro F1 across seeds: 0.000   standard deviation across seeds: 0.000
target: every seed >= 0.6 and standard deviation < 0.05
  every seed >= 0.6: NO   spread < 0.05: yes
How to read the interval: if it contains the target (0.60), this seed cannot honestly be called a pass
or a fail - the test split is too small to tell. Report that as a finding rather than rounding it up or down.
```
```
C1 accuracy        [0.258, 0.226, 0.29]
C3 precision       [0.0, 0.0, 0.0]
C5 (n, precision)  [(0, None), (0, None), (0, None)]
```
- Why these verdicts: where a generous reading existed it was not taken.
  Rows 3 and 5 have no predictions, which could be read as "nothing it
  predicted was wrong"; that reading would pass a model that predicts nothing.
  Row 2's standard-deviation clause is technically met at 0.000 only because
  the model scores zero everywhere. Row 4 is MET only against its revised
  wording.

  | Option | Chosen / why |
  |---|---|
  | MISSED when no predictions exist | **Chosen** — the target was not shown to hold |
  | "Cannot be judged" | Rejected — it leaves a blank where a verdict is owed |
  | MET (empty set vacuously true) | Rejected — rewards predicting nothing |

**Step: name the cause of each miss and how it is known.**
- One root cause for rows 1, 2, 3 and 5, in the README's Diagnoses: the model
  cannot reach the task from this data and this training run. Evidence: it is
  a constant guesser; its probabilities are nearly flat; 12 of 21 labels have
  3 or fewer posts; the run is short (about 27 optimizer steps, corrected in
  Milestone 6 from an earlier figure of 33); and the five core labels have
  10 to 23 posts each yet were never predicted, so data volume alone does not
  explain it.

**Step: use the agreement report as evidence.**
- The report separates a labeling problem from a model problem only if
  disagreements line up with the matrix's confusions. There were no
  disagreements and the confusions are not concentrated on a label pair. The
  README therefore concludes, with moderate confidence, that the cause is
  model and data, and names the check that would settle it for the HVAC
  labels (not done). See Findings #15.

**Step: decide which criteria were broken and which were just missed.**
- Rule applied: a criterion may be revised only if it could not be measured.

| # | Decision |
|---|---|
| 1, 2 | Missed and measurable; no revision (lowering a target is the change that earns nothing) |
| 3, 5 | Unmeasurable when nothing is predicted; revised: no predictions counts as MISSED |
| 4 | Unmeasurable as originally worded; revised in `ab26ff5` to me versus staff on the 30 staff posts |

- Revisions for 3 and 5 sit under the original lines in `criteria.md`
  (`44805d1`); neither target number moved and neither verdict changed.

**What this means for the next milestone.** The diagnosis names training
length as an untested hypothesis, which is what Milestone 5 tests.

**Terms introduced:** none new beyond Section 3.

**Milestone summary.** True now: 1 criterion MET (4, under its revised
wording) and 4 MISSED, each miss with a cause and its evidence. Still open:
whether the HVAC labels are consistent.

**What this milestone taught:**
1. Where a target has no data behind it, "met" and "cannot be judged" are both
   generous; MISSED is the only verdict that does not reward a silent model.
2. A criterion may be revised for measurability, never because it was missed.
3. An agreement score can only diagnose labeling where its labels are the
   ones being diagnosed.
4. One shared root cause is clearer than four separate stories.

**Completion criteria** (copied from the task text supplied for this
milestone).

| Criterion | Status |
|---|---|
| "Every criterion has a verdict and a sentence on how you decided." | TRUE (README, "Verdicts and Diagnoses" table, five rows) |
| "Every miss has a cause and evidence." | TRUE (four misses, one shared root cause with evidence) |
| "Commits." | TRUE (five commits, one per step) |

---

### Unit 6, Milestone 5: Fix One Thing and Re-Run

**The idea.** Pick one thing the diagnosis pointed at, change it, retrain on
the same three seeds, and report whether it helped. One change measured
properly beats four changes nobody can tell apart.

**Where the answers went.** Four commits.

| Commit | Step | Numstat |
|---|---|---|
| `182c835` | Pick the improvement and write the prediction first | `README.md` +10/−2 |
| `bca35b4` | Make the change and retrain | `results_three_seeds_after.json` +1919/−0; `takemeter.ipynb` +2/−44 |
| `f21f250` | Put both run logs side by side | `README.md` +75/−5 |
| `78cf2c0` | Say whether it helped | `README.md` +14/−0 |

The notebook's 44 deleted lines are not removed code: the two edited cells'
`source` was re-serialised from a list of lines to one string.

**Step: pick the improvement.**
- Chosen: `EPOCHS` from 3 to 15, everything else unchanged.

  | Option | Chosen / why |
  |---|---|
  | Change one training setting (epochs) | **Chosen** — the diagnosis pointed at a short run and a never-predicted 23-post label |
  | Relabel one boundary | Not applicable: no disagreements, so no boundary applied two ways |
  | Add 30 posts to the thinnest label | Not chosen — 12 of 21 labels are too thin for one label's posts to matter, and it would not explain a never-predicted 23-post label |

- The prediction was written before the run (`182c835`, 21:00; the run's
  commit `bca35b4` is 21:16): more than two labels predicted on at least two
  seeds, accuracy about 0.30 to 0.45 but under 0.65, some predictions at or
  above 0.80.

**Step: make the change, set `RUN_LABEL = "after"`, retrain the same seeds.**
- Took about 934 seconds (15.6 minutes) for the three seeds against about 3.6
  minutes before. The after entries also record `epochs`, `learning_rate` and
  `batch_size`.
```
Scoring 5 labels: general_construction_non_mechanical (23), software_benchmarking_technical_workflow (16), emerging_ai_automation_tools (16), bid_strategy_scope_gaps_gc_relations (12), enterprise_erp_bidding_platforms (10)
  seed | test posts | macro F1 | 95% bootstrap interval
--------------------------------------------------------
    42 |         31 |    0.080 | [0.000, 0.222]
     7 |         31 |    0.213 | [0.000, 0.433]
  2024 |         31 |    0.222 | [0.057, 0.444]
mean macro F1 across seeds: 0.172   standard deviation across seeds: 0.080
target: every seed >= 0.6 and standard deviation < 0.05
  every seed >= 0.6: NO   spread < 0.05: NO
How to read the interval: if it contains the target (0.60), this seed cannot honestly be called a pass
or a fail - the test split is too small to tell. Report that as a finding rather than rounding it up or down.
```
```
C1 accuracy        [0.419, 0.419, 0.548]
C3 precision       [1.0, 1.0, 0.5]
C5 (n, precision)  [(0, None), (0, None), (0, None)]
```

**Step: both logs in the README, side by side.** `f21f250`; the README's "Run
Log — After" and "Before and after, side by side" tables.

| Measure | Before: seed 42 / 7 / 2024 | After: seed 42 / 7 / 2024 |
|---|---|---|
| Overall accuracy | 0.258 / 0.226 / 0.290 | 0.419 / 0.419 / 0.548 |
| Spread of accuracy | 0.065 | 0.129 |
| Macro F1 over all labels (notebook) | 0.061 / 0.059 / 0.056 | 0.144 / 0.178 / 0.299 |
| Macro F1, 5 core labels | 0.000 / 0.000 / 0.000 | 0.080 / 0.213 / 0.222 |
| Precision, `general_construction_non_mechanical` | 0.000 / 0.000 / 0.000 | 1.000 / 1.000 / 0.500 |
| Different labels predicted | 2 / 2 / 1 | 4 / 4 / 4 |
| Predictions at confidence ≥ 0.80 | 0 / 0 / 0 | 0 / 0 / 0 |

**Step: say whether it helped.** README, `78cf2c0`: it helped, partly;
accuracy rose on all three seeds and no verdict changed (Criteria 1, 2, 3 and
5 still MISSED, Criterion 4 still MET). Against the prediction: direction
right (more labels predicted, accuracy mostly in the range, criteria 1 and 2
still missed); the claim that some predictions would reach 0.80 was wrong and
seed 2024's 0.548 exceeded the predicted range. Stated limits: one change, one
run per seed, a 31-post test split, validation loss not examined, no other
learning rate tried. See Findings #14.

**What this means for the next milestone.** The write-up must say what is
still broken, since nothing flipped to MET.

**Terms introduced:** none new beyond Section 3.

**Milestone summary.** True now: before and after result sets exist side by
side, the change helped on accuracy and not on any verdict, and the README says
so. Still open: the remaining limit is data, which was not changed.

**What this milestone taught:**
1. A prediction committed before a run, then checked against it, shows which
   parts of the diagnosis were right (the direction) and wrong (the
   confidence).
2. A change can improve a measure and leave every verdict unchanged.
3. Changing only one setting made the comparison readable; the cost was
   fifteen minutes of CPU.
4. Re-serialising a notebook's cell source shows up as many deleted lines
   that are not deleted code.

**Completion criteria** (copied from the task text supplied for this
milestone).

| Criterion | Status |
|---|---|
| "Both run logs are in your README." | TRUE ("Run Log — Before" and "Run Log — After") |
| "You can say in one sentence whether the change helped." | TRUE (README opens "it helped, partly" with the caveat that no verdict changed) |
| "Commits." | TRUE (four commits, one per step) |

---

### Unit 6, Milestone 6: Say What's Still Broken, and Submit

**The idea.** Finish the write-up and make the call on what was not fixed.
For each criterion still missed: what would be done and why work stopped; two
sentences on the gap between what the labels were meant to capture and what the
model learned; and an updated account of how AI was used, especially where a
model helped find patterns in errors. The step avoids ending with a table of
numbers and no judgement about them.

**Where the answers went.** Four commits, the first three one per step and
the fourth an audit.

| Commit | Step | Numstat |
|---|---|---|
| `e0d1259` | What's Still Broken | `README.md` +15/−0 |
| `e013f2b` | The gap sentences | `README.md` +2/−0 |
| `c8cde9e` | How I Used AI | `README.md` +8/−0 |
| `6e2a454` | README audit before submitting | `README.md` +14/−8; `takemeter.ipynb` +1/−1 |

**Step: write What's Still Broken.** For Criteria 1, 2, 3 and 5: why still
missed, what would be done next, and what could prove that wrong. The stated
reason for stopping is the course's one-change limit plus a data ceiling (the
scrapers reach about 204 unique posts), and the README says explicitly that
this is not a claim of having run out of time. The one check to do first, and
not done: hand-label a random 30 of the 204 posts blind to Gemini and compute
kappa.

**Step: two sentences on the gap.** In the after-run seed-42 matrix the model
got 13 of 31 right and 12 of those 13 are career or venting posts; 14 of its
18 mistakes landed on those two labels and the other four on
`software_benchmarking_technical_workflow`. The labels were meant to separate
five axes (trade scope, software, sector, purpose, poster role); the model
learned mostly one contrast between two post-purpose labels. Checked against
`results_three_seeds_after.json` before writing.

**Step: update How I Used AI.** A third moment was added and a statement of
what was and was not AI-written: the 30 staff labels and the Baseline vs.
Trained prediction are the author's; verdicts, diagnoses, run-log text and the
write-up sections were drafted by the AI assistant from the result files.

**Audit before submitting (`6e2a454`).** A full read of the README found and
fixed: the optimizer-step arithmetic (173 train posts used, 142 correct; see
Findings #13); a stray "LoRA" in the README's first paragraph; "five core
taxonomy distinctions" against the six listed; a Training Run section that
implied every run used 3 epochs; and a blank "What the gap actually means",
which was filled. One contradiction was found and not resolved: the README says
in "The Dataset" that the first 20 rows' labels were assigned by Gemini and in
"How I Used AI" that they were assigned by human opinion. The note text on
those rows is in Gemini's output format, which favors the first statement; it
is flagged for the author to settle.

  | Option | Chosen / why |
  |---|---|
  | Correct the numbers in place and say so in the AI-use section | **Chosen** |
  | Quietly fix the numbers | Rejected — hides an AI error from a graded disclosure |
  | Edit the dated prediction's wording | Rejected — a prediction must not change after the fact; a note was added instead |

**What this means for the next milestone.** There is none in this project;
this is the last entry. The unresolved README contradiction is the one open
item that affects what the README claims about the dataset.

**Terms introduced:** none new.

**Milestone summary.** True now: the README has both run logs, verdicts,
diagnoses, the matrix, the agreement report, the still-broken section, the gap
sentences and the AI-use update. Intentionally still open: the unresolved
first-20-labels contradiction and the HVAC-label consistency check.

**What this milestone taught:**
1. "I stopped because of X" should name the actual reason; the README names the
   one-change limit and the data ceiling, not time.
2. A final read-through of a graded document finds errors the writing pass
   did not (here, an arithmetic error and a stray term).
3. An error made by the AI assistant belongs in the AI-use disclosure.
4. A contradiction the author must settle is better flagged than silently
   resolved by the assistant.

**Completion criteria** (copied from the task text supplied for this
milestone).

| Criterion | Status |
|---|---|
| "Both run logs" | TRUE |
| "verdicts" | TRUE |
| "diagnoses" | TRUE |
| "your matrix" | TRUE (seed-42 before and after, typed as tables) |
| "and your agreement report" | TRUE (30 of 30, no disagreements) |
| "Same repository URL." | Not verifiable from repository contents (a submission action outside the repo; the configured remote is `https://github.com/Angelica-M/ai201-project3-takemeter-starter-v2026.git`) |
| "Commits." | TRUE (four commits, listed above) |


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

### Unit 6 Required Features

| Feature | What it requires | Current state |
|---|---|---|
| A baseline run | A general model classifies the held-out posts using only the label definitions; no training | **Done** — `baseline.py`, `baseline_results.json` (accuracy 0.129 with definitions), plus a bare-names run, `baseline_results_bare_names.json` (0.355). Unit 6, Milestone 1 |
| Three trials per criterion | Retrain on three random splits with three seeds | **Done** — seeds 42, 7, 2024; `results_three_seeds_before.json` and `results_three_seeds_after.json`. Unit 6, Milestones 2 and 5 |
| A confusion matrix typed as a markdown table | Not an image | **Done** — seed-42 matrices, before and after, in the README. Unit 6, Milestones 2 and 5 |
| An agreement check against the staff set, every disagreement adjudicated | 30 posts labeled by staff and by the author; report the rate; argue each disagreement | **Done, vacuously on adjudication** — 30 of 30 agreed, 0 disagreements, so nothing to adjudicate. Unit 6, Milestone 3 |
| A verdict and a diagnosis for each of the five criteria | Met or missed against Unit 5's targets; for every miss, which label and what cause | **Done** — 1 MET (Criterion 4, revised wording), 4 MISSED, one shared root cause. Unit 6, Milestone 4 |
| One improvement, made and measured | Change one thing, retrain, both result sets side by side | **Done** — `EPOCHS` 3 to 15; accuracy up on all seeds, no verdict changed. Unit 6, Milestone 5 |

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

Unit 6 adds no packages. The extra Unit 6 prerequisites (committed
`results.json` and `test_split.csv`, the baseline model download, the staff
files and the notebook's `RUN_LABEL`) are in `ENVIRONMENT_SETUP.md` section 6.
