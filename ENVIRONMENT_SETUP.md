# TakeMeter — Environment Setup

> No course-wide, multi-unit setup guide was supplied for this project pair.
> This file covers only this project's own setup instructions, reformatted and
> checked against the actual repository rather than transcribed as given. Every "Reality check"
> callout below reflects something verified directly — a file actually read,
> a command actually run — not an assumption carried over from the brief.

---

## 1. Fork and Clone

```bash
git clone https://github.com/YOUR-USERNAME/ai201-project3-takemeter-starter-v2026.git
cd ai201-project3-takemeter-starter-v2026
```

This fork is the repository submitted for both Unit 5 and Unit 6. Everything
in the project runs inside this folder — labeling, the training notebook, the
baseline — and everything the notebook writes is committed from here, the
same as any other file.

> **Reality check:** confirmed this repository exists locally at
> `ai201-project3-takemeter-starter-v2026`, with its own `.git` history (66
> commits when this file was audited, from `git log --oneline | wc -l`) and
> its own `.venv` (`.venv/Scripts/python.exe` present, and `.venv/` is listed
> in `.gitignore`).

---

## 2. Virtual Environment and the Environment Check

**macOS / Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
python test.py
```

**Windows (PowerShell)**
```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
python test.py
```

No API key and no accounts are needed for **this specific set of steps** —
`test.py`, the baseline, and the training notebook all run locally and have
no hosted-model dependency. `requirements.txt` confirms this directly:
`pandas`, `transformers`, `datasets`, `accelerate`, `torch`,
`scikit-learn` — nothing that calls an external API.

> **⚠️ Reality check — this claim does NOT extend to the data-collection
> step.** See section 4 below. The classifier training pipeline is
> genuinely key-free; the *dataset this project actually trains on* was
> built with a script that requires a hosted-model API key. The brief's
> "no accounts, nothing hosted" framing is accurate for Milestone 5 (training)
> but incomplete for Milestone 4 (data collection) as this repo actually
> implements it.

`test.py` checks disk and memory before anything else — about 9 GB free (the
baseline model is ~1.6 GB; PyTorch itself is not small) and warns under 6 GB
of RAM (the training run peaks around 2.1 GB). Verified directly against the
source: `MIN_DISK_GB = 9`, `MIN_RAM_GB = 6`, matching exactly.

`test.py` also reports which device PyTorch will train on — `cuda`, `mps`,
or `cpu` — all three supported. This is worth writing down, not just reading:
`results.json` confirms this repo's actual Unit 5 training run used `cpu`
(`torch 2.14.1+cpu`, no GPU).

> **Reality check — real output, captured 2026-10-31 from this working copy
> with `.venv` active** (not copied from documentation):
> ```
> AI201 environment check
> ------------------------------------------------------------
> [PASS] Python version
>          3.13.7 on Windows
> [PASS] Virtual environment
>          C:\Users\Angelica M\Documents\CP Prework\ai201-project3-takemeter-starter-v2026\.venv
> [PASS] Pinned packages
>          all 6 import cleanly
> [PASS] Free disk space
>          286.4 GB
> [PASS] Memory
>          15.3 GB
> [PASS] PyTorch
>          2.14.1+cpu, CPU only — fine, a three-seed run is minutes not hours
> [PASS] Training packages
>          datasets and accelerate both import
> [PASS] Transformers
>          imports cleanly
> [PASS] Baseline model
>          already downloaded
> [PASS] Project files
>          notebook, scripts and practice data all present
> ------------------------------------------------------------
> 10 passed, 0 failed, 0 to look at, 0 skipped
> ```
> Two things to read precisely. First, the `[PASS] Virtual environment` line
> is the check that fails when the shell is not running inside `.venv`; this
> working copy has one, and in this shell `python` resolves to it. Second,
> the `.venv`'s PyTorch is `2.14.1+cpu`, the same version recorded in
> `results.json` for the Unit 5 training run, so a re-run here uses the same
> torch as the committed results. (A different Python on the same machine
> can report a different torch; that is a different interpreter, not a
> changed `.venv`.)

You'll see one `[WARN]` (or, as verified here, a `[PASS]`) for the baseline
model — ~1.6 GB, downloaded via `python baseline.py`. In this repo it's
already downloaded, which matches the brief's own advice to clear this
*before* the next unit's session rather than during it.

---

## 3. Open the Notebook

Open `takemeter.ipynb` in VS Code and select the kernel: this project's
`.venv` Python, via the kernel selector in the top right. Run section 1's
first cell — it should print the detected device and the installed
`torch` version. Getting the kernel wrong (pointing at a different Python
than this project's `.venv`) is the most common reason this first cell
fails with a "package not installed" error for a package that is, in fact,
installed — just not where the kernel is looking.

---

## 4. Reality Check: The Data-Collection Pipeline Needs Undocumented Setup

The brief's own setup text states: *"No API key and no accounts for this
project... Nothing here calls a hosted model."* That's true of `test.py` and
the training notebook. It is **not** true of how this repository's dataset
was actually built.

`build_dataset.py` — the script that produced every row in `data/labels.csv`
— imports `python-dotenv` and `google.generativeai`, reads a `GEMINI_API_KEY`
from a `.env` file, and calls the hosted Gemini API (`gemini-3.1-flash-lite`,
with a fallback pool of other Gemini models) to assign a label, a runner-up
label, two confidence scores, and a one-sentence reason to every post it
scrapes. **None of `python-dotenv`, `google-generativeai`, or `requests` is
listed in `requirements.txt`** — confirmed directly by reading the file.
Following only the documented `pip install -r requirements.txt` step leaves
`build_dataset.py` unrunnable.

To actually reproduce the data-collection step as this repo performed it:

```bash
pip install google-generativeai python-dotenv requests
cp env.txt .env            # Windows: copy env.txt .env
# then edit .env and set GEMINI_API_KEY=<a real key from aistudio.google.com>
```

`env.txt` is the committed template for this; the real `.env` (325 bytes)
exists on disk in this repo and is correctly listed in `.gitignore` — the key
itself was never committed. `build_dataset.py` also talks to Reddit's public
search feeds (`search.rss` / Atom), which needs no login or key — only the
*labeling* half of the pipeline needs Gemini.

> **Caution — do not run this on the real checkout.** `build_dataset.py` is
> append-only on `data/labels.csv`. Because every row there already has a
> label, the script does not resume: it scrapes up to 100 *new* posts per
> search from Reddit (about one request per minute), appends them,
> deduplicates, and sends the new unlabelled rows to Gemini. That extends the
> committed 204-row dataset and invalidates every number computed from it. To
> reproduce the dataset from scratch, run it in a copy of the repository with
> `data/labels.csv` removed.

**This step is not required to satisfy the brief's Milestone 4 as written**
(which describes a human collecting and mostly-hand-labeling 200 posts). It
*is* required to understand or re-run how this specific repository's dataset
was actually produced — see the Milestones section of
`PROJECT_WALKTHROUGH.md` for the full account of that divergence.

---

## 5. Command Reference and Commit Workflow

`RUNNING.md` (331 lines) is this repo's own command reference — every
command, which step goes with which milestone, and a troubleshooting table.
It is read, not edited here.

One machine, one repository, for both units — nothing is pushed
automatically. Anything the notebook writes (`results.json`, `test_split.csv`,
and later the three-seed and baseline files) is committed deliberately, the
same as any other file in this repo.
