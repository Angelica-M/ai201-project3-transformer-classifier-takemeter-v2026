# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does
<!-- Your community, and what your classifier sorts posts into. 3-4 sentences. -->

## What This Does: Your community, and what your classifier sorts posts into.

TakeMeter targets **r/estimators**, a specialized online Reddit community where construction estimators, preconstruction managers, trade contractors, and software vendors discuss plan takeoffs, bidding strategies, software tools, and industry practices. The posts in this subreddit range from highly technical discussions about sheet metal weight calculations and complex mechanical scope gaps to general career questions, software sales pitches, and bid-day frustration posts. To analyze and categorize (label) these posts effectively, TakeMeter sorts incoming community content into six taxonomy distinctions (five groups of four labels, plus one stand-alone `unrelated` label, 21 labels in all):

* All 6 Taxonomy Distinctions are
* **Trade Scope & Division Boundaries** (with a total of 4 categories that are: Division 23 (Full Mechanical/HVAC), Sub-Trade Specialty (Sheet Metal vs. Hydronics/Piping), MEP Package (Mechanical/Plumbing/Electrical), and General Construction / Non-Mechanical)
* **Software Stack & Tooling Architecture** (with a total of 4 categories that are: General PDF & CAD Digitizers, Specialized Mechanical Estimating Databases, Enterprise ERP & Bidding Platforms, and Emerging AI & Automation Tools)
* **Project Sector & Delivery Model** (with a total of 4 categories that are: Commercial / Institutional / Industrial (Mid–Large B2B), Multi-Family Residential / Tract Housing, Service, Replacement & Retrofit Bidding, and Public Works / Prevailing Wage Bidding)
* **Operational Intent & Post Purpose** (with a total of 4 categories that are: Software Benchmarking & Technical Workflow, Bid Strategy, Scope Gaps & GC Relations, Career Pathing, Compensation & Hiring, and Industry Venting & Operational Burnout)
* **Poster Role & Expertise Level** (with a total of 4 categories that are: Senior B2B Mechanical Estimator / Chief Estimator, General Contractor (GC) Precon Coordinator, Field Tech / Residential Installer / Apprentice, and Software Vendor / Sales Representative)
* **Unrelated** (a single stand-alone category to catch any input posts that do not meet or closely align with any of the other Taxonomy Distinctions and their categories)

By establishing these distinct boundaries, the classifier (a fully fine-tuned `distilbert-base-uncased`; this project does not use LoRA) is meant to evaluate both the technical scope and user intent behind every post, allowing high-value peer expertise and technical queries to be sorted cleanly from marketing noise, career advice, and entry-level discussions.

---

## Label Taxonomy
<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.
     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `division_23_full_mechanical_hvac` 

**Definition:** Posts covering entire building mechanical and environmental control systems under a single prime mechanical quote, combining sheet metal, equipment schedules, and hydronic piping.
* **Label 1:** *Division 23 (Full Mechanical/HVAC)* — **Taxonomy Distinction 1:** *Trade Scope & Division Boundaries*

**Example 1:**
> We are bidding a complete Div 23 package on a 4-story medical office building, including the chillers, AHUs, and ductwork installation.

**Example 2:**
> Looking for feedback on how to structure our prime HVAC package bid form when RTU and VRF options are both listed as add-alternates.

### `sub_trade_specialty_sheet_metal_vs_hydronics`

**Definition:** Discussions strictly focused on granular trade-specific takeoffs such as ductwork weight/gauge calculations, fitting counts, or specialized piping materials.
* **Label 2:** *Sub-Trade Specialty (Sheet Metal vs. Hydronics/Piping)* — **Taxonomy Distinction 1:** *Trade Scope & Division Boundaries*

**Example 1:**
> What labor factor do you apply per pound of heavy-gauge welded ductwork versus standard SMACNA spiral duct?

**Example 2:**
> Need advice on calculating fitting counts and linear footage for a large chilled water PEX supply line takeoff in PlanSwift.

### `mep_package_mechanical_plumbing_electrical`

**Definition:** Posts resolving scope overlaps, omissions, or package coordination boundaries between mechanical, plumbing, and electrical trades.
* **Label 3:** *MEP Package (Combination Mechanical/Plumbing/Electrical)* — **Taxonomy Distinction 1:** *Trade Scope & Division Boundaries*

**Example 1:**
> Who typically covers the line voltage controls and motor starters on your jobs—the Div 23 mechanical sub or Div 26 electrical?

**Example 2:**
> How do you catch scope gaps between Div 22 plumbing condensate drains and Div 23 equipment hookups during preconstruction review?

### `general_construction_non_mechanical`

**Definition:** Queries focused strictly on non-HVAC trade takeoffs, acting as a negative filter for structural, civil, or architectural trade discussions.
* **Label 4:** *General Construction / Non-Mechanical* — **Taxonomy Distinction 1:** *Trade Scope & Division Boundaries*

**Example 1:**
> Estimating concrete slab yardage and rebar quantities for a tilt-wall warehouse expansion using Bluebeam Revu.

**Example 2:**
> What is the average square-foot unit price you use for Div 9 drywall and metal framing on commercial interior buildouts?

### `general_pdf_cad_digitizers`

**Definition:** Discussions centered on manual 2D quantity extraction, scale calibration, drawing markups, and basic PDF takeoff software without trade-specific database assemblies.
* **Label 5:** *General PDF & CAD Digitizers* — **Taxonomy Distinction 2:** *Software Stack & Tooling Architecture*

**Example 1:**
> Having trouble getting custom columns and tool chest symbol counts to export cleanly from Bluebeam Revu to Excel.

**Example 2:**
> What is the best way to calibrate page scales across overlay drawings when vector PDFs have distorted dimensions?

### `specialized_mechanical_estimating_databases`

**Definition:** Technical queries evaluating database software that automatically calculates fitting counts, material pricing feeds, and MCAA labor unit hours per unit.
* **Label 6:** *Specialized Mechanical Estimating Databases* — **Taxonomy Distinction 2:** *Software Stack & Tooling Architecture*

**Example 1:**
> How does Trimble AutoBid Mechanical compare to FastPIPE when setting up custom auto-fitting assembly rules and labor multipliers?

**Example 2:**
> Is anyone integrating Harrison Publishing pricing feeds directly into QuoteSoft to keep material costs updated automatically?

### `enterprise_erp_bidding_platforms`

**Definition:** High-level workflow discussions regarding invitation to bid (ITB) management, subcontractor bid matrix creation, and ERP accounting synchronization.
* **Label 7:** *Enterprise ERP & Bidding Platforms* — **Taxonomy Distinction 2:** *Software Stack & Tooling Architecture*

**Example 1:**
> How are you handling bid coverage tracking and ITB distribution through BuildingConnected vs Procore Bid Board?

**Example 2:**
> Looking for best practices when transferring awarded estimate job costs from ConstructConnect into Spectrum ERP.

### `emerging_ai_automation_tools`

**Definition:** Experimental or analytical posts evaluating AI, machine learning, OCR, or scripts for automated plan indexing and symbol counting.
* **Label 8:** *Emerging AI & Automation Tools* — **Taxonomy Distinction 2:** *Software Stack & Tooling Architecture*

**Example 1:**
> Has anyone tested Togal.ai or TakeoffBuilder for auto-detecting equipment symbols on vector mechanical sheets?

**Example 2:**
> I wrote a Python script using OCR to parse drawing spec books and auto-link equipment schedules to plan callouts.

### `commercial_institutional_industrial`

**Definition:** High-value B2B plan-and-spec or design-build project estimates requiring formal specification reviews, unit pricing breakdowns, and structured bid forms.
* **Label 9:** *Commercial / Institutional / Industrial (Mid–Large B2B)* — **Taxonomy Distinction 3:** *Project Sector & Delivery Model*

**Example 1:**
> Bidding a $4M plan-and-spec central plant overhaul for a university campus with strict long-lead equipment requirements.

**Example 2:**
> How do you handle submittal addendums and RFI clarifications when pricing a commercial hospital expansion under a design-build model?

### `multi_family_residential_tract_housing`

**Definition:** Repetitive wood-frame residential or apartment estimating based on price-per-square-foot, per-unit, or per-door metrics.
* **Label 10:** *Multi-Family Residential / Tract Housing* — **Taxonomy Distinction 3:** *Project Sector & Delivery Model*

**Example 1:**
> What is your typical price-per-door or per-unit estimate for DX split system installs on a 200-unit multi-family apartment complex?

**Example 2:**
> Estimating wood-frame tract housing HVAC layouts using piece-rate labor pricing instead of detailed commercial sheet metal takeoffs.

### `service_replacement_retrofit_bidding`

**Definition:** Quick-turnaround commercial service bids, equipment changeouts, or time-and-material quotes with short diagnostic-to-quote cycles.
* **Label 11:** *Service, Replacement & Retrofit Bidding* — **Taxonomy Distinction 3:** *Project Sector & Delivery Model*

**Example 1:**
> How do you factor crane rigging and roof curb adaptor costs when quoting a rapid weekend Carrier RTU replacement?

**Example 2:**
> Looking for standard T&M labor rates and markup percentages on commercial HVAC preventive maintenance service agreements.

### `public_works_prevailing_wage_bidding`

**Definition:** Government or public sector bidding involving certified payroll, Davis-Bacon prevailing wage determinations, and strict sealed bid procedures.
* **Label 12:** *Public Works / Prevailing Wage Bidding* — **Taxonomy Distinction 3:** *Project Sector & Delivery Model*

**Example 1:**
> How do you adjust your labor pricing models when bidding public school projects subject to local prevailing wage rates?

**Example 2:**
> Dealing with MWBE/DBE subcontracting goals and bonding limits on a sealed bid public works municipal building.

### `software_benchmarking_technical_workflow`

**Definition:** Posts evaluating features, pricing tiers, system stability, and speed efficiency across competing estimating software platforms.
* **Label 13:** *Software Benchmarking & Technical Workflow* — **Taxonomy Distinction 4:** *Operational Intent & Post Purpose*

**Example 1:**
> Comparing PlanSwift vs Trimble AutoBid: is the database setup time worth the upgrade for a 10-person estimating team?

**Example 2:**
> Our digitizer software keeps crashing on 200MB CAD vector sheets—looking for benchmark recommendations for smoother workflow performance.

### `bid_strategy_scope_gaps_gc_relations`

**Definition:** Operational posts on protecting profit margins, managing incomplete drawings, writing exclusions, and handling general contractor bid deadlines.
* **Label 14:** *Bid Strategy, Scope Gaps & GC Relations* — **Taxonomy Distinction 4:** *Operational Intent & Post Purpose*

**Example 1:**
> How do you protect your quote against scope creep when GC drawings are only 60% CDs on bid day?

**Example 2:**
> What standard exclusions list do you attach to your mechanical proposals to avoid eating unpriced GC scope gaps?

### `career_pathing_compensation_hiring`

**Definition:** Discussions regarding estimator compensation packages, remote work options, workload expectations, and career advancement paths.
* **Label 15:** *Career Pathing, Compensation & Hiring* — **Taxonomy Distinction 4:** *Operational Intent & Post Purpose*

**Example 1:**
> What is the market compensation rate and bonus structure for a Senior Mechanical Chief Estimator in a major metro area?

**Example 2:**
> Is 5 full commercial plan takeoffs per week a reasonable workload expectation for a junior estimator working remotely?

### `industry_venting_operational_burnout`

**Definition:** Frustration-driven posts highlighting bid-day stress, last-minute addendums, unrealistic GC deadlines, and industry friction points.
* **Label 16:** *Industry Venting & Operational Burnout* — **Taxonomy Distinction 4:** *Operational Intent & Post Purpose*

**Example 1:**
> GC sent out a 150-page addendum 2 hours before the bid due date and still expects full itemized breakdowns.

**Example 2:**
> Tired of working weekends doing takeoffs only for GCs to ghost us on bid results and re-bid the project three months later.

### `senior_b2b_mechanical_estimator_chief_estimator`

**Definition:** Posts authored by experienced professionals demonstrating deep industry terminology, calibrated labor multipliers, and strategic risk management.
* **Label 17:** *Senior B2B Mechanical Estimator / Chief Estimator* — **Taxonomy Distinction 5:** *Poster Role & Expertise Level*

**Example 1:**
> After 20 years as Chief Estimator, here is how we calibrate MCAA labor units against historical shop performance data to maintain a 25% win rate.

**Example 2:**
> My rule of thumb for overhead recovery on $5M+ mechanical packages is to isolate equipment quotes from field labor units before applying burden.

### `general_contractor_precon_coordinator`

**Definition:** Posts written from the buyer perspective evaluating how subcontractor quotes are reviewed, leveled, and combined into a prime bid.
* **Label 18:** *General Contractor (GC) Precon Coordinator* — **Taxonomy Distinction 5:** *Poster Role & Expertise Level*

**Example 1:**
> From the GC precon side: why do mechanical subs wait until 10 minutes before bid closing to submit their scope sheets?

**Example 2:**
> What is your process for leveling three wildly different HVAC subcontractor quotes when scope inclusions are missing?

### `field_tech_residential_installer_apprentice`

**Definition:** Entry-level or field-focused posts seeking guidance on transitioning into the office, reading blueprints, or beginner tools.
* **Label 19:** *Field Tech / Residential Installer / Apprentice* — **Taxonomy Distinction 5:** *Poster Role & Expertise Level*

**Example 1:**
> Just transitioned from being a field HVAC tech to junior estimator—what is the easiest way to learn blueprint reading and scale takeoffs?

**Example 2:**
> Field installer moving into office precon next month—looking for recommendations on software tutorials for someone starting from scratch.

### `software_vendor_sales_representative`

**Definition:** Posts or replies containing self-promotional pitches, demo offers, or vendor-driven product marketing.
* **Label 20:** *Software Vendor / Sales Representative* — **Taxonomy Distinction 5:** *Poster Role & Expertise Level*

**Example 1:**
> DM me if you'd like to schedule a quick demo of our platform—we auto-detect mechanical symbols and export straight to Excel.

**Example 2:**
> We solved this exact problem with our automated AI takeoff software—check out our website for a 14-day free trial.

### `unrelated` 

**Definition:** Catch-all category for posts that fall entirely outside construction precon, bidding, and estimating software—such as generic financial advice, unrelated software programming queries, random spam, or general sub trades asking non-estimating jobsite questions.
* **Label 21:** *Unrelated* — **Taxonomy Distinction 6:** *Unrelated*

**Example 1:**
> What are the best Tax-Free Savings Account (TFSA) index funds for long-term passive investing if I want to retire in my 50s?

**Example 2:**
> Does anyone have a good python script to automatically download attached PDF invoices from Outlook emails and save them to a local Google Drive folder?

### The 1st Hardest Boundary
<!-- e.g. "If the post names a specific checkable fact, it's `analysis`, even
     if the tone is heated." -->

**Which two labels:**
`general_pdf_cad_digitizers` vs. `specialized_mechanical_estimating_databases`
Distinction is difficult because both categories share the same surface‑level functions:
both measure lengths and areas, allow markup, import PDFs or CAD files, and export quantities.  This overlap means that a post or product description may look mechanical, even when the underlying tool is just a general digitizer. The main sources of confusion are: 
* **Shared core functionality**: Every digitizer measures drawings, so measurement alone is not a reliable indicator.
* **Trade context does not guarantee specialization**: A general digitizer used by an HVAC estimator is still general‑purpose; the trade context is misleading.
* **Marketing language is vague**: Many tools advertise “HVAC takeoff” or “MEP workflows” without offering true mechanical automation.
* **Partial feature sets**: Some tools include one mechanical feature (e.g., duct symbols) but lack assemblies, labor databases, or pricing—making them hybrid and hard to classify.
* **User workflows mask tool capability**: Estimators often build their own assemblies manually inside general tools, making the tool appear specialized when it isn’t.

**The decision rule I used every time:**
1. Specialized MEP/HVAC digitizers add domain‑specific automation that ties measurements to mechanical systems; these features transform raw geometry into mechanical assemblies, not just measurements. Classify it as `specialized_mechanical_estimating_databases`, if the post describes trade-specific features that automatically translate measurements into mechanical components (e.g., MCAA/PHCC labor units, fitting/elbow auto-assembly generation, pressure‑class logic, or live material pricing feeds). More specifically, these are the tasks automatic fitting and elbow generation based on duct/pipe routing, MCAA or PHCC labor units tied to each component, material pricing feeds from suppliers, system‑aware assemblies (e.g., duct sections with hangers, insulation, transitions), trade‑specific rules (e.g., pressure class, gauge, joint type), and automatic BOM creation for mechanical systems.
2. General CAD/PDF digitizers focus purely on universal measurement and markup tasks; these tools are intentionally trade‑agnostic. They work for architecture, civil, landscaping, electrical, plumbing, HVAC, or any discipline because they only measure geometry. If the post focuses solely on manual spatial measurement (these are universal measurement tasks like length, area, and count takeoffs), drawing scaling, basic annotation and markup (general markup tools), or tools related to generic PDF/CAD navigation tools or exporting quantitites to spreadsheets functions (without any automated mechanical assemblies or trade‑specific databases), classify it as `general_pdf_cad_digitizers` regardless of the trade context mentioned (even if the user is performing HVAC or piping takeoff).

### The 2nd Hardest Boundary 

**Which two labels:**
`general_construction_non_mechanical` vs. `unrelated`

**The decision rule I used every time:**
If a post discusses preconstruction workflows, takeoff measurements, bluebeam markups, or material/labor pricing for non-HVAC construction trades (such as concrete, framing, or electrical), classify it under its respective trade category or `general_construction_non_mechanical`. Classify a post as `unrelated` **only** if it contains no construction estimation, bidding, takeoff, or precon content whatsoever (e.g., general personal finance, non-estimating coding scripts, jobsite safety memes, or spam).

---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and 3 hard cases. -->

**Where the posts came from:**
All 204 posts are text (self) posts from **r/estimators**, collected in October 2026 by `build_dataset.py` through Reddit's public Atom search feeds (`search.rss`, no login). Three searches were run, each asked for 100 posts not already in `labels.csv`:
1. Top (All Time): `self:true (guide OR megathread OR analysis OR "deep dive")`
2. Hot: the same keyword query
3. Top (All Time): `self:true` alone, to supply posts the keyword searches could not (those two together only contain about 105 posts)

281 rows were collected in total; 77 repeated posts (identical column 1) were then removed, keeping the first copy of each, which leaves **204 unique posts**. Each post is its title plus its full body text.

**How I labelled them:**
Every label and note was produced by the Gemini API (`gemini-3.1-flash-lite`), not by hand. For each post the script sent the full text, the role "expert HVAC estimator / MEP construction expert", the 21 label definitions from the Label Taxonomy above and the two hardest-boundary decision rules, and asked for the single best-fit label plus a runner-up, each with a 0-100 match score and a one-sentence reason. Column 3 stores that ranking in the form "Highest semantic match of X% for [label] (with 2nd highest of Y% as [label]) due to [reason]". The first 20 rows carry a `cold` tag at the start of their note, as the project spec asks; those labels were also assigned by Gemini. This section does not claim a manual re-check of the labels.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `division_23_full_mechanical_hvac` | 0 | 0.00% |
| `sub_trade_specialty_sheet_metal_vs_hydronics` | 0 | 0.00% |
| `mep_package_mechanical_plumbing_electrical` | 0 | 0.00% |
| `general_construction_non_mechanical` | 23 | 11.28% |
| `general_pdf_cad_digitizers` | 5 | 2.451% |
| `specialized_mechanical_estimating_databases` | 1 | 0.49% |
| `enterprise_erp_bidding_platforms` | 10 | 4.90% |
| `emerging_ai_automation_tools` | 16 | 7.84% |
| `commercial_institutional_industrial` | 1 | 0.49% |
| `multi_family_residential_tract_housing` | 1 | 0.49% |
| `service_replacement_retrofit_bidding` | 0 | 0.00% |
| `public_works_prevailing_wage_bidding` | 2 | 0.98% |
| `software_benchmarking_technical_workflow` | 16 | 7.84% |
| `bid_strategy_scope_gaps_gc_relations` | 12 | 5.88% |
| `career_pathing_compensation_hiring` | 49 | 24.02% |
| `industry_venting_operational_burnout` | 55 | 26.96% |
| `senior_b2b_mechanical_estimator_chief_estimator` | 1 | 0.49% |
| `general_contractor_precon_coordinator` | 8 | 3.92% |
| `field_tech_residential_installer_apprentice` | 0 | 0.00% |
| `software_vendor_sales_representative` | 1 | 0.49% |
| `unrelated` | 3 | 1.47% |
| **Total** | **204** | 100% |

Five labels have no posts at all (`division_23_full_mechanical_hvac`, `sub_trade_specialty_sheet_metal_vs_hydronics`, `mep_package_mechanical_plumbing_electrical`, `service_replacement_retrofit_bidding`, `field_tech_residential_installer_apprentice`), and the two largest labels hold 51% of the data together. The guide / analysis style of search pulls in long discussion posts, which skew towards career and venting topics.

**Three hard cases**

<!-- Any post that made you pause: what it was, which 2 labels it could have been, and 
     what you chose. These are worth more than the easy 190. -->

These are the three posts where Gemini's top two scores were closest (a 5-point gap each; the next closest gap in the dataset is 7 points). Post text is shortened with "…"; the full text is in `data/labels.csv`.

**1.**
> *The post:* "New Siding Estimator Wanting Mentor — Hey everyone, I had to join a cat subreddit to get enough karma to post. I am 4 months in with a skins company that brought me in to start bidding siding on Multifamily, comm, mixed…bigger projects. I had no estimating experience. …" (about 3,900 characters)
>
> *Could have been:* `career_pathing_compensation_hiring` (90%) or `general_construction_non_mechanical` (85%)
>
> *Labelled (by Gemini) as `career_pathing_compensation_hiring`, because:* the post is mainly a request for career mentorship from a newcomer to estimating, even though the trade it discusses (siding and exterior wall systems) is non-mechanical.

**2.**
> *The post:* "Better sub-network vs existing (Bluebook, Procore, Planhub) — Hey all, Some of you probably know me as the author of a few AI applications in precon posts … I'm working to build a better subcontractor network for the industry. Think of it as an updated Bluebook (better data, more vetted profiles, GCs kno…" (about 2,000 characters)
>
> *Could have been:* `enterprise_erp_bidding_platforms` (90%) or `general_contractor_precon_coordinator` (85%)
>
> *Labelled (by Gemini) as `enterprise_erp_bidding_platforms`, because:* the post is mostly about improving bidding platforms and subcontractor networks (enterprise-level tools), although it is written from the GC precon coordinator's point of view.

**3.**
> *The post:* "Downtobid app review from a GC estimator — What is Downtobid? In their own language, Downtobid is an AI construction bidding platform for GCs and subs. It's supposed to: Streamline bid invites / ITBs and deadlines. Analyze your drawings and plans, identify scopes, draft invite emails … " (about 10,200 characters)
>
> *Could have been:* `enterprise_erp_bidding_platforms` (95%) or `software_benchmarking_technical_workflow` (90%)
>
> *Labelled (by Gemini) as `enterprise_erp_bidding_platforms`, because:* it is a detailed technical review, but what it evaluates is a bidding / ITB management platform, which falls squarely under the enterprise bidding category.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`, fine-tuned for 21-way text classification (the 21 labels in the Label Taxonomy).

**Settings (the unit 5 run and the before run):** 3 epochs, learning rate 2e-5, batch size 16, 128 tokens per post, seed 42. The Milestone 5 improvement changed only the epochs, to 15 (see The Improvement). **Trained on: `cpu`** (device name "CPU", torch 2.14.1+cpu, Windows 11 machine, no GPU). The device is also recorded in `results.json`, because different hardware can give slightly different numbers for the same seed.

**Anything I changed from the defaults, and why:** For the unit 5 run the training settings are all the notebook's defaults (the one later change, EPOCHS 3 to 15, is in The Improvement). The only edits to `takemeter.ipynb` were setup: my name and email, `LABELS` set to my 21 labels, and `CSV` pointed at `data/labels.csv` (where `build_dataset.py` writes my data) instead of `labels.csv` in the repo root. The notebook's code cells were run from a script with the project's `.venv` Python rather than inside the VS Code notebook window.

**Split sizes:** 204 posts split 142 train / 31 validation / 31 test. The notebook could not stratify the split, because several labels have only one post, so it fell back to a plain random split (that is a warning about the dataset, not the split). Test split per label:

| Label | Test posts |
|---|---|
| `industry_venting_operational_burnout` | 8 |
| `career_pathing_compensation_hiring` | 7 |
| `general_construction_non_mechanical` | 4 |
| `emerging_ai_automation_tools` | 2 |
| `public_works_prevailing_wage_bidding` | 2 |
| `bid_strategy_scope_gaps_gc_relations` | 2 |
| `general_contractor_precon_coordinator` | 2 |
| `enterprise_erp_bidding_platforms` | 1 |
| `software_benchmarking_technical_workflow` | 1 |
| `commercial_institutional_industrial` | 1 |
| `software_vendor_sales_representative` | 1 |
| All other 10 labels (including the 5 with no posts at all) | 0 |

Only one label reaches the 8-post mark in test, so every other label's F1 will swing a lot between seeds, and 10 labels cannot be measured at all on this split. That will explain much of next unit's variance.

**How Criterion 2 is measured, and why it has a bootstrap confidence interval:** With a test split of about 31 posts, a per-label F1 is mostly luck, and 21 separate ones would be 21 untrustworthy numbers. So `criteria.md` Criterion 2 asks for one number per seed: the macro F1 (the average of the per-label F1 scores) over five core technical estimating labels that each have at least 10 posts in the whole dataset, which must reach 0.60 on every seed with a standard deviation under 0.05 across the seeds. `macro_f1_ci.py` computes it from the confusion matrix that the notebook already saves in its results file, so nothing needs retraining.

The script also attaches a 95% bootstrap confidence interval to each seed's number. It takes the test posts, resamples them with replacement 2,000 times at the same size, recomputes the macro F1 on every resample, and keeps the middle 95% of those values. Its value is that it shows how much of the number is only the luck of which 31 posts ended up in the test split. If the interval contains 0.60, that seed can't honestly be called a pass or a fail; if it sits clearly above or below 0.60, the verdict is real. It does not capture the randomness of training itself, which is what the three seeds are for, and with this little data the intervals will be wide. That width is itself a finding about the dataset.

One disclosure about timing: Criterion 2 was reworded (which labels it covers and how it is computed) after the first single-seed training run had been executed, when its overall accuracy was already visible. The 0.60 target and the 0.05 spread limit were not changed, the five labels were chosen from their post counts in the dataset, not from any per-label score, and no three-seed run existed yet.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the problem. -->

**Moment 1: collecting the posts (Claude Code)**

- *What I asked for:* A scraper, `build_dataset.py`, that pulls 200 posts from r/estimators using the exact search `self:true (guide OR megathread OR analysis OR "deep dive")`, sorted Top / All Time, and writes them into `labels.csv`.
- *What came back:* Reddit's `.json` endpoints now return HTTP 403 to anyone who isn't logged in, so the first version fetched nothing. Claude Code tested the alternatives and found that the same search still works as a public Atom feed (`search.rss`, no login), which returns each post's full text. It then measured the limits instead of guessing: the keyword query only matches about 105 posts in total, no matter the sort or time window, so 200 was impossible with that query alone.
- *What I changed:* I added a Hot-sorted version of the keyword search and a third, broader `self:true` search (Top sort) to supply enough different posts. I set each search to collect 100 posts that are not already in `labels.csv`, kept new rows appending to the end of the file, and had the script remove repeated posts (keeping the first copy) before any labelling. Even so, only 204 unique posts were reachable, which is above the 150 floor but short of 200.

**Moment 2: labelling the posts (Gemini, called from my script)**

- *What I asked for:* For each post, one best-fit label out of my 21, with a runner-up and a 0-100 match score for each, in a fixed note format. The prompt gave Gemini the role "expert HVAC estimator for bids and takeoffs and expert in the MEP construction industry", my label definitions from the Label Taxonomy, and my two hardest-boundary decision rules.
- *What came back:* A JSON answer per post that the script validated (it had to be one of the 21 labels, and the top two had to differ). The first run produced no labels at all because the model I had picked was retired for new users (HTTP 404), and the next model allowed only 20 requests per day on the free tier. Retries and a fallback label would have written invented labels into the data, so the script now skips and retries instead, and rotates through the other Gemini models my key can see. All 204 final labels came from `gemini-3.1-flash-lite`.
- *What I changed:* I removed a hard-coded "pre-labelled, checked" note so every row carries Gemini's own ranked scores, made the script label only rows whose label is empty, and made it check that the text it sent still matches the row it writes to. I also removed the "cold" prefix from the code (the first 20 rows still carry it from an earlier run).

**Moment 3: finding the patterns in my errors, and checking my own work (Claude Code, unit 6)**

- *What I asked for:* Run the baseline, the three-seed training, the agreement check against staff and the improvement, then work out why the criteria were missed.
- *What came back:* Claude Code read the result files and turned them into patterns I could check. The per-seed prediction counts showed the model was close to a constant guesser (seed 2024 called all 31 test posts `career_pathing_compensation_hiring`). The confidence numbers showed its probabilities were almost flat (0.09 to 0.12). A count of the labels showed 12 of 21 had 3 or fewer posts. A zero-shot baseline that used only bare label names beat my trained model, which told me my definitions were hurting that baseline. It also caught two problems in my own setup: my criteria and README called the model "LoRA", but the notebook fully fine-tunes DistilBERT with no LoRA, and the first copy of the staff key was byte-for-byte my own file, so its 100% agreement meant nothing.
- *What I changed:* I corrected the "LoRA" wording in `criteria.md`; I replaced the copied key with the real staff file, and the README says the first attempt was discarded; and I chose the one improvement (more epochs) from the diagnosis. In the final check of this README, Claude Code also found an error of its own: it had used 173 training posts to estimate optimizer steps, but my split has 142 training posts (204 minus 31 validation and 31 test), so the correct figures are about 27 steps for 3 epochs and about 135 for 15. I corrected those numbers in the README and the notebook comment. The wording of some earlier commit messages still shows the old figure.

**What was and was not AI-written.** I labelled the 30 staff posts by hand, and my Baseline vs. Trained prediction was my own writing, not drafted by AI. The verdicts, diagnoses, run-log text, Agreement Report, "What's Still Broken" and the gap sentences were drafted by Claude Code from my result files. Every number in them comes from `results_three_seeds_before.json`, `results_three_seeds_after.json`, `agreement_results.json` and `baseline_results*.json`, and can be re-checked with the commands in `criteria.md`. Judgements in those sections, such as calling Criteria 3 and 5 MISSED when no predictions existed, are written up with their reasoning in the README so they can be disputed.

**Pre-labelling disclosure:** Every one, except for the first 20, of the 204 labels and notes in `labels.csv` was produced by the Gemini API (`gemini-3.1-flash-lite`), not by me. The notes in column 3 are Gemini's own explanations of its top two picks. The first 20 rows are tagged `cold` as the project spec asks, to denote that those labels were not assigned by Gemini but were assigned by human opinion; Gemini did help with the "notes" explanation section of those 20 rows. The label quality and the boundary behavior in the 3 hard cases above are Gemini's judgement applied to my definitions.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

Both models scored the same 31 held-out posts in `test_split.csv` (seed 42). The baseline is zero-shot `facebook/bart-large-mnli` using my 21 label definitions and chose among the 16 labels present in my data; the trained model is the fine-tuned `distilbert-base-uncased` from the notebook. Output of `python baseline.py --data data/labels.csv --trained results.json`:

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

**What my definitions were worth (the baseline run twice, on purpose):** the baseline above is the run that uses my 21 definitions from `label_definitions.txt`, and it is the baseline I report. I also ran it once with no definitions file, so it fell back to bare label names (`python baseline.py --data data/labels.csv --definitions no_definitions_on_purpose.txt --out baseline_results_bare_names.json`), on the same 31 posts:

| Baseline run | Overall accuracy | Macro F1 (over the 16 labels in my data) |
|---|---|---|
| With my definitions (reported above) | 0.129 (4/31) | 0.060 |
| Bare label names | 0.355 (11/31) | 0.215 |
| My trained model (for reference) | 0.258 (8/31) | 0.061 as printed (0.042 over the same 16 labels) |

So my definitions did not help the baseline, they cost it 0.226 of accuracy (11 posts versus 4 right). With definitions it predicted `software_vendor_sales_representative` for 19 of 31 posts; with bare names its guesses were spread across `general_contractor_precon_coordinator` (9), `general_construction_non_mechanical` (5), `bid_strategy_scope_gaps_gc_relations` (5) and others, and it gave `enterprise_erp_bidding_platforms` an F1 of 1.0 (on a single test post). The two runs agreed on only 4 of the 31 posts. A plausible reason, which I have not tested, is that my long definitions make the zero-shot model score on shared vocabulary instead of on the label's meaning. With 31 posts, 11 versus 4 correct is a large gap but still a small sample. Because the bare-names baseline is stronger than my trained model on both accuracy and macro F1, the trained model beats only the definitions-based baseline, not the bare-names one. The full results of the bare-names run are in `baseline_results_bare_names.json`.

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

I predict the fine-tuned LoRA model will achieve an overall test accuracy between 0.35 and 0.45—comfortably outperforming the baseline score of 0.129 and the majority-class guess of 0.258, yet capped by severe data skew. While high-frequency categories like `industry_venting_operational_burnout` (36 train) and `career_pathing_compensation_hiring` (34 train) will be learned robustly by the adapter, the model will completely fail on the three zero-train labels (`public_works_prevailing_wage_bidding`, `software_vendor_sales_representative`, and `commercial_institutional_industrial`) that hold 4 test instances. Additionally, the model's 128-token truncation limit will cut off longer posts (where the 90th percentile spans ~1,880 characters), exacerbating classification errors on fuzzy category boundaries like operational venting versus bid strategy scope gaps where semantic score gaps are already narrow.

*Note on the prediction above: it says "LoRA model" and "adapter". That wording was wrong, because the notebook fully fine-tunes DistilBERT and uses no LoRA. I have left the prediction text as I wrote it, since a prediction should not be edited after the fact.*

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->

At 3 epochs, my fine-tuning added almost nothing. Against the baseline that used my 21 definitions the trained model was better on accuracy (0.258 against 0.129) but identical on macro F1 (0.061 against 0.060), and both scored zero on nearly every label, so neither learned the rare ones. Against the bare-label-names baseline (0.355 accuracy, 0.215 macro F1) the trained model was worse on both, so a free zero-shot model with no training on my data beat it. That means at 3 epochs the fine-tuning had not yet earned its cost.

After the 15-epoch improvement, the trained model's accuracy on the same seed-42 split is 0.419 (13 of 31), against the bare-names baseline's 0.355 (11 of 31). That is a 2-post difference on 31 posts, too small to call, so the honest reading is that the fine-tuned model has caught up with the best zero-shot baseline and has not clearly passed it. The baseline was run once, on the seed-42 split only, so I cannot make this comparison for seeds 7 and 2024.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

**Where and how it ran:** device `cpu` (CPU, torch 2.14.1+cpu, Windows 11 machine, no GPU), the same for all three seeds, and recorded inside every seed's entry in `results_three_seeds_before.json`. Seeds 42, 7 and 2024 were trained in one run of section 6 (`RUN_LABEL = "before"`), about 3.6 minutes in total, with the unchanged settings: 3 epochs, learning rate 2e-5, batch size 16, 128 tokens. Seed 42 reproduced the unit 5 run exactly (accuracy 0.258, macro F1 0.061).

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | at least 0.65 on every seed | 0.258 | 0.226 | 0.290 | **MISSED** |
| 2. Macro F1 over the 5 core technical labels | at least 0.60 on every seed, standard deviation under 0.05 | 0.000 | 0.000 | 0.000 | **MISSED** |
| 3. Precision of `general_construction_non_mechanical` | at least 0.75 on every seed | 0.000 (never predicted; 4 test posts) | 0.000 (never predicted; 4 test posts) | 0.000 (never predicted; 5 test posts) | **MISSED** |
| 4. Cohen's kappa against staff labels (30 posts) | at least 0.70 | 1.000 (30 of 30 agreed with staff; not a model measure, so the same for all seeds) | n/a | n/a | **MET** (revised wording) |
| 5. Precision when confidence is at least 0.80 | at least 0.85 on every seed | no prediction reached 0.80 (n = 0) | no prediction reached 0.80 (n = 0) | no prediction reached 0.80 (n = 0) | **MISSED** |

**The spread across seeds.** Accuracy was 0.258, 0.226 and 0.290, a spread of 0.065 (6.5 points), which is under the notebook's "more than about ten points" warning line, so by that rule alone the measure is not flagged as unstable. It still hides a bigger problem. The model was close to a constant guesser on every seed:

- Seed 42 predicted only `industry_venting_operational_burnout` (25 posts) and `career_pathing_compensation_hiring` (6).
- Seed 7 predicted `career_pathing_compensation_hiring` for 28 posts and `industry_venting_operational_burnout` for 3.
- Seed 2024 predicted `career_pathing_compensation_hiring` for **all 31** posts, so its 0.290 is just the 9 career posts that happened to land in that test split.

So accuracy moved with which of the two largest labels a seed favoured and how many posts of it the split held, not with anything the model learned. Macro F1 barely moves (0.056 to 0.061) only because it is close to zero everywhere: only those two labels ever score, and `industry_venting_operational_burnout` scored 0.000 in seed 2024. The model's confidence tells the same story: the average winning probability was only about 0.09 to 0.12 whether it was right or wrong (uniform guessing over 21 labels would be 0.048), and no prediction on any seed reached 0.80, so Criterion 5 has nothing to measure. My reading, not yet tested, is that a 204-post dataset this lopsided, with only 3 epochs of training, taught the model label frequencies instead of post content. That is a diagnosis about the dataset and the training run, not a mistake in running it.

**Full spread table, as printed by the notebook (section 6):**

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.258 | 0.226 | 0.290 | 0.065 |
| Macro F1 | 0.061 | 0.059 | 0.056 | 0.005 |
| F1 — `division_23_full_mechanical_hvac` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `sub_trade_specialty_sheet_metal_vs_hydronics` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `mep_package_mechanical_plumbing_electrical` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `general_construction_non_mechanical` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `general_pdf_cad_digitizers` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `specialized_mechanical_estimating_databases` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `enterprise_erp_bidding_platforms` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `emerging_ai_automation_tools` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `commercial_institutional_industrial` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `multi_family_residential_tract_housing` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `service_replacement_retrofit_bidding` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `public_works_prevailing_wage_bidding` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `software_benchmarking_technical_workflow` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `bid_strategy_scope_gaps_gc_relations` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `career_pathing_compensation_hiring` | 0.308 | 0.250 | 0.450 | 0.200 |
| F1 — `industry_venting_operational_burnout` | 0.364 | 0.462 | 0.000 | 0.462 |
| F1 — `senior_b2b_mechanical_estimator_chief_estimator` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `general_contractor_precon_coordinator` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `field_tech_residential_installer_apprentice` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `software_vendor_sales_representative` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `unrelated` | 0.000 | 0.000 | 0.000 | 0.000 |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42 (the matrix the notebook's section 7 prints; rows are true labels, columns are the model's guesses). The matrices for seeds 7 and 2024 are in `results_three_seeds_before.json` under `confusion`.

| true \ predicted | division_23_full_mechanical_hvac | sub_trade_specialty_sheet_metal_vs_hydronics | mep_package_mechanical_plumbing_electrical | general_construction_non_mechanical | general_pdf_cad_digitizers | specialized_mechanical_estimating_databases | enterprise_erp_bidding_platforms | emerging_ai_automation_tools | commercial_institutional_industrial | multi_family_residential_tract_housing | service_replacement_retrofit_bidding | public_works_prevailing_wage_bidding | software_benchmarking_technical_workflow | bid_strategy_scope_gaps_gc_relations | career_pathing_compensation_hiring | industry_venting_operational_burnout | senior_b2b_mechanical_estimator_chief_estimator | general_contractor_precon_coordinator | field_tech_residential_installer_apprentice | software_vendor_sales_representative | unrelated |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **division_23_full_mechanical_hvac** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **sub_trade_specialty_sheet_metal_vs_hydronics** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **mep_package_mechanical_plumbing_electrical** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **general_construction_non_mechanical** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 4 | 0 | 0 | 0 | 0 | 0 |
| **general_pdf_cad_digitizers** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **specialized_mechanical_estimating_databases** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **enterprise_erp_bidding_platforms** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| **emerging_ai_automation_tools** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 |
| **commercial_institutional_industrial** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| **multi_family_residential_tract_housing** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **service_replacement_retrofit_bidding** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **public_works_prevailing_wage_bidding** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 |
| **software_benchmarking_technical_workflow** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| **bid_strategy_scope_gaps_gc_relations** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| **career_pathing_compensation_hiring** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 5 | 0 | 0 | 0 | 0 | 0 |
| **industry_venting_operational_burnout** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 6 | 0 | 0 | 0 | 0 | 0 |
| **senior_b2b_mechanical_estimator_chief_estimator** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **general_contractor_precon_coordinator** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| **field_tech_residential_installer_apprentice** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **software_vendor_sales_representative** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| **unrelated** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

In seed 42, 5 real `career_pathing_compensation_hiring` posts were called `industry_venting_operational_burnout` and only 2 went the other way, but because that seed called 25 of the 31 posts venting (and seeds 7 and 2024 called almost everything career instead), this direction reflects whichever of the two largest labels a seed happened to favour, not a learned career-versus-venting boundary.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | at least 0.65 on every seed | **MISSED** | The three seeds scored 0.258, 0.226 and 0.290. The best seed is 0.36 below the target, and no seed came close, so this is not a close call that the three-seed rule decides. Seed 42's 0.258 also equals the majority-class guess from the baseline section, so the model is not better than always naming the biggest label. |
| 2 | Macro F1 over the 5 core technical labels | at least 0.60 on every seed, standard deviation under 0.05 | **MISSED** | Macro F1 over the five fixed labels is 0.000 on all three seeds, and `python macro_f1_ci.py` gives a 95% bootstrap interval of [0.000, 0.000] each time, so the interval is nowhere near 0.60 and this is a real miss, not a split too small to judge. The "standard deviation under 0.05" half is technically met (0.000), but only because the model never scores on any of the five labels, so I do not count it as a pass. |
| 3 | Precision of `general_construction_non_mechanical` | at least 0.75 on every seed | **MISSED** | The model never predicted this label on any seed (it had 4, 4 and 5 test posts), so there were no predictions to be precise about: precision is 0 of 0, stored as 0.000. A target that says precision must be at least 0.75 is not shown to hold by an empty set, so I count it as missed. The generous reading, "nothing it predicted was wrong", would also pass a model that predicts nothing at all. |
| 4 | Cohen's kappa against staff labels | at least 0.70 (my revised wording: me versus staff on the 30 staff posts) | **MET** | `python agreement.py` and `python kappa.py` gave 30 of 30 agreed and kappa 1.000 against the real staff key. The original wording (two annotators on 30 held-out posts) could not be measured, so this verdict is against the revised wording in `criteria.md`. It shows I applied the three-label staff rules consistently, not that my 21 HVAC labels are consistent. |
| 5 | Precision when confidence is at least 0.80 | at least 0.85 on every seed | **MISSED** | No prediction reached 0.80 on any seed (n = 0), so there is no precision to compare with 0.85. The criterion exists so that confident tags can be trusted without checking each post, and a model that is never confident gives no tags to trust, so I count it as missed. The criterion's checking note says a seed with n = 0 "cannot be judged"; I am choosing MISSED over "cannot be judged" because the generous version of that phrase would leave an empty verdict where the target was not shown to hold. |

Tally: 1 MET, 4 MISSED. The one MET is the criterion that does not depend on the model.

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->


Criterion 4 was met, so it needs no diagnosis. Criteria 1, 2, 3 and 5 are missed, and they share one root cause with different symptoms, so I diagnose that cause once and then say what it did to each criterion.

**Root cause: a task the model cannot reach from this data and this training run, made worse by too few examples for most labels.** I am saying "cannot reach" and not "labels are bad" or "labels are hard", and here is how I know:

- **It is a constant guesser, not a model that learned wrong boundaries.** Seed 42 predicted only two labels (`industry_venting_operational_burnout` 25 times, `career_pathing_compensation_hiring` 6), seed 7 predicted career 28 times and venting 3, and seed 2024 predicted career for all 31 test posts. Accuracy moved only with which big label a seed favoured, which is why the three accuracies (0.258, 0.226, 0.290) track how many career or venting posts each split held.
- **Its probabilities are nearly flat.** The average winning probability was about 0.09 to 0.12 whether it was right or wrong, against 0.048 for a uniform guess over 21 labels. A model that had learned a wrong boundary would be confident and wrong. This one has not committed to anything.
- **The data are very lopsided.** Of the 21 labels, 5 have no posts at all and 7 more have 3 or fewer (5 labels with 1 post, `public_works_prevailing_wage_bidding` with 2, `unrelated` with 3), so 12 of 21 labels cannot be learned or tested. The two biggest labels hold 104 of 204 posts (venting 55, career 49).
- **The training run is very short.** About 142 posts in training (204 minus 31 validation and 31 test) at batch size 16 is 9 steps per epoch, so 3 epochs is about 27 optimizer steps at a learning rate of 2e-5. That count comes from the settings, not from a log. It is not many updates for a model whose 21-way classification head started random.
- **Too few examples is not the whole story.** The five core labels in Criterion 2 have 10 to 23 posts each, and `general_construction_non_mechanical` has 23 (the third largest label), yet none was ever predicted. If data volume alone explained it, a label with 23 posts should show some signal. That points at the short training run too, but I have not tested it. It is a hypothesis for Milestone 5, not a finding.

**Criterion 1 (accuracy 0.258 / 0.226 / 0.290, target 0.65).** Cause: the root cause above. Evidence: the accuracy equals what always guessing the biggest label scores (0.258, the majority-class figure from the baseline section), so the model learned no more than label frequency. Seed 7's 0.226 is below even that.

**Criterion 2 (macro F1 over the five core labels 0.000 on every seed, target 0.60).** Cause: too few examples for the technical labels (10 to 23 posts, 1 to 4 in each test split) together with the under-trained run. Evidence: all five F1s are exactly 0.000 on all three seeds because the model never predicted any of the five. In seed 42's confusion matrix the true posts for these labels all went to `industry_venting_operational_burnout` (for example 4 `general_construction_non_mechanical`, 2 `emerging_ai_automation_tools`), the biggest label, not to a related one, so this is frequency collapse and not a confusion between two neighbouring labels. I cannot call any pair "genuinely hard" because a model that never predicts a label shows no pair confusion.

**Criterion 3 (precision of `general_construction_non_mechanical`, 0.000, target 0.75).** Cause: the same under-training; this is not a "too few examples" case in the strict sense because the label has 23 posts. Evidence: it was never predicted (0 of 0), and in seed 42 all four of its test posts were called `industry_venting_operational_burnout`. The consequence for the use case is that the noise filter would pass every non-HVAC post through unflagged.

**Criterion 5 (no prediction at or above 0.80, target 0.85 precision).** Cause: the same under-training, which here shows up as under-confidence, the opposite of the overconfidence my "too hard" note for this criterion predicted. Evidence: the average winning probability of 0.09 to 0.12 means the model never got near 0.80, so n = 0 on every seed.

**What I have not shown.** I have not shown that more training would fix it, since I have not trained longer. I have not shown that my labels are consistent, which is what the next subsection addresses with the agreement report.

**Labelling problem or model problem? What the agreement report can and cannot tell me.**

The course's example diagnosis works by lining up two things: where the confusion matrix puts the misses, and where my own disagreements with staff fall. If both point at the same label pair, the cause is how I labelled. I did the same lining-up, and it gives a limited answer.

- **What it shows.** I matched staff on 30 of 30 posts, including all 7 that staff themselves marked as hard (see the Agreement Report). So when I apply a written rule to posts, I applied it consistently on this set. That is some evidence against "I apply boundaries loosely", but only for the three-label staff taxonomy.
- **What it cannot show.** My 204 HVAC labels were written by Gemini from my definitions, not applied by me post by post, and no second human has labelled them. So 30 of 30 on the staff set cannot tell me whether the HVAC labels are consistent, and I am not claiming it does.
- **No pair to line up.** With 0 disagreements there is no label pair on the staff side to match against the matrix. On the model side the misses are not concentrated on a pair either: the true posts of almost every label went to the same two big labels (`industry_venting_operational_burnout`, `career_pathing_compensation_hiring`). Inconsistent labelling would show up as confident confusion between specific neighbouring labels, which is not what the matrix shows.
- **Other evidence that the labels carry signal.** The zero-shot baseline that used only bare label names, with no training on my data, scored 0.355 accuracy on the same 31 test posts, which is above the 0.258 majority-class guess. A model that had never seen my labels could therefore agree with them well above chance, so the labels are not random noise.
- **Evidence that the boundaries are genuinely soft.** In Gemini's own notes, which give a top score and a second-highest score for each post, the median gap between the two is 20 percentage points, and 58 of the 204 posts (28%) have a second label within 10 points. Those scores are the labeller's own numbers and are not calibrated, so I treat this only as a sign that many posts could fairly carry two labels. That would cap how high accuracy can go. It would not by itself make a model predict one label for every post.

**My conclusion.** The best-supported cause of the four misses is the model and data (too little data per label and too short a training run), not my labelling. I hold that with moderate confidence, not certainty, because the one check that could separate them for the HVAC data has not been done: hand-labelling a random 30 of my own posts without looking at Gemini's labels and computing kappa against them. I have not done it, so I make no claim about the result.

**Did any criterion turn out to be broken, as opposed to missed?**

The rule I followed: a criterion may be revised only if it could not be measured, never because I missed it. Going through the five:

| # | Broken or just missed? | Decision |
|---|---|---|
| 1 | Missed. It was measurable and every seed fell far short. | No revision. Lowering 0.65 would be exactly the change that does not count. |
| 2 | Missed. Measurable, with a bootstrap interval nowhere near 0.60. | No revision. |
| 3 | Partly broken. The original wording gives no verdict when the label is never predicted, because precision is then 0 of 0. | Revise so that "no predictions" is stated to count as MISSED. The 0.75 target is unchanged and the verdict does not change. |
| 4 | Broken in unit 5's wording: "two independent annotators on 30 held-out posts" could not be measured. | Already revised earlier (kappa between me and staff on the 30 staff posts, target unchanged at 0.70). |
| 5 | Partly broken, for the same reason as 3: with no prediction at or above 0.80, precision is undefined. | Revise so that "no predictions at or above 0.80" is stated to count as MISSED. The 0.85 target is unchanged and the verdict does not change. |

The revisions for 3 and 5 do not move any target and do not change any verdict. Their only effect is that the original wording can no longer be read generously, as "nothing it predicted was wrong". The original lines stay in `criteria.md` and each revision goes underneath with its reason, as the course requires.




---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100% (Cohen's kappa 1.000, where 0.333 agreement was expected by chance; `agreement_results.json`, commit 7657aab).

How it was produced: I labelled the 30 posts in `data/staff_posts.csv` under `data/staff_taxonomy.md` and committed `my_staff_labels.csv` (d3a3178) before the staff key was in the repo. Then I ran `python agreement.py` and `python kappa.py`. One disclosure: my first attempt at the key was a copy of my own file, which scored a meaningless 100%. I discarded that result without committing it and re-ran with the real `data/staff_labels.csv` (8932fa1).

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

There were **none**: `agreement.py` listed 0 disagreements, so there is nothing to adjudicate and nothing to sort. I am not inventing any.

| Pile | Count |
|---|---|
| (a) The rule covered it and I applied it loosely | 0 |
| (b) The rule genuinely doesn't say | 0 |
| (c) The rule is ambiguous here and my reading is defensible | 0 |

**The closest thing to evidence: the staff's own "borderline" posts.** Staff marked 7 of the 30 posts as hard (`borderline = yes` in `data/staff_labels.csv`), and I matched staff on all 7. Each is a place where a disagreement was most likely, and each turns on one of the written rules:

| Post | Label (both of us) | What makes it hard (staff's note) | Rule |
|---|---|---|---|
| sp_005 | `analysis` | Opens like a shoe hot take, but the second sentence carries the argument with two comparable numbers | 1 |
| sp_010 | `analysis` | Hedged conclusion on a tiny sample, but the checkable observation is what the claim rests on | 1 |
| sp_014 | `hot_take` | The 3% decorates a conclusion already reached ("so obviously" does the work) | 1 |
| sp_019 | `hot_take` | Reads reflective, almost a reaction, but it is a general claim about everyone | 2 |
| sp_020 | `hot_take` | Sounds observational, but no count is given: asserting, not showing | 1 |
| sp_028 | `reaction` | "Third this year" is countable but nothing is argued from it, so no fact is load-bearing | 1, 3 |
| sp_029 | `reaction` | A wry shoe observation, but it reports the writer's own realisation in the moment | 2 |

**What the pattern in my disagreements tells me:**

There is no disagreement pattern, because there were no disagreements. What I can say is limited to this set. Matching on all 7 posts the staff themselves called hard suggests I applied rule 1 (the fact must carry the argument, not decorate it) and rule 2 (about the writer versus a general claim) the same way staff did, and I found no gap in the definitions that forced a guess. That fits a rule that is tight on this set, not a set that was easy. I would not push it further:

- **Thirty posts is small.** A perfect score on 30 can't rule out an agreement rate of 85% to 90% on a larger set, and the kappa interval of [1.000, 1.000] only reflects that there was nothing to resample.
- **It does not transfer to my 21 HVAC labels.** This checked how I apply the staff taxonomy of three labels. My 204 training labels were written by Gemini, and no second human has labelled them. So the agreement report cannot yet tell a labelling problem from a model problem for the HVAC data, which is what Milestone 4 needs it to do. I will say that in the diagnoses rather than lean on this result.



---

## The Improvement

**What I changed:** one training setting: `EPOCHS` from 3 to 15 in the notebook. Nothing else changes: the same 204 posts and labels, the same split code, the same three seeds (42, 7, 2024), learning rate 2e-5, batch size 16, 128 tokens, and the same model (`distilbert-base-uncased`, full fine-tune). At 142 training posts and batch size 16 (9 steps per epoch), that moves training from about 27 optimizer steps to about 135.

**Which diagnosis pointed at it:** the root cause in Verdicts and Diagnoses: a model that is a near-constant guesser with nearly flat probabilities (average winning probability 0.09 to 0.12 against 0.048 for a uniform guess), which looks like a model that has barely started learning, and a short run of about 27 optimizer steps. The strongest single clue was that `general_construction_non_mechanical` has 23 posts, the third most of any label, and was never predicted on any seed, so data volume alone does not explain the misses.

**Why this change and not the other two the course lists:**

- *Relabel one boundary:* not applicable. My agreement check found no disagreements, so there is no boundary I applied two ways, and I will not invent one.
- *Add 30 posts to the thinnest label:* my diagnosis says 12 of 21 labels are too thin, and no single label's 30 posts would fix that. It also would not explain why a 23-post label was never predicted. That is a data change for a later round.
- *Change one training setting:* the course says to pick this only if the diagnosis actually pointed at training. Mine did, but only as a hypothesis, which I said I had not tested. This run tests it.

**My prediction, written before the run:** the model will stop being a pure constant guesser on at least two of the three seeds and will predict more than two labels. Overall accuracy will rise from 0.226 to 0.290 to somewhere around 0.30 to 0.45, but it will stay under 0.65, so Criterion 1 stays MISSED. Macro F1 over the five core labels will move off 0.000 on at least one seed but stay well under 0.60, so Criterion 2 stays MISSED. I expect some predictions to reach 0.80 confidence, which would let Criterion 5 be measured for the first time. If accuracy does not move and the model still collapses onto one or two labels, the short run was not the main cause. I would then say that and point at the data (labels with 3 or fewer posts) instead.

### Run Log — After

**Where and how it ran:** the same device (`cpu`, torch 2.14.1+cpu), the same three seeds (42, 7, 2024) and the same 204 posts as the before run. The one difference is `EPOCHS = 15` instead of 3, with learning rate 2e-5, batch size 16 and 128 tokens unchanged. Written to `results_three_seeds_after.json` (`RUN_LABEL = "after"`). The three seeds took about 934 seconds (15.6 minutes), against about 3.6 minutes before.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | at least 0.65 on every seed | 0.419 | 0.419 | 0.548 | **MISSED** |
| 2. Macro F1 over the 5 core technical labels | at least 0.60 on every seed, standard deviation under 0.05 | 0.080 (95% interval 0.000 to 0.222) | 0.213 (0.000 to 0.433) | 0.222 (0.057 to 0.444); standard deviation across seeds 0.080 | **MISSED** |
| 3. Precision of `general_construction_non_mechanical` | at least 0.75 on every seed | 1.000 (1 of 1 predicted right; 4 test posts) | 1.000 (2 of 2; 4 test posts) | 0.500 (2 of 4; 5 test posts) | **MISSED** |
| 4. Cohen's kappa against staff labels (30 posts) | at least 0.70 | 1.000 (30 of 30 agreed with staff; not a model measure, so unchanged by retraining) | n/a | n/a | **MET** (revised wording) |
| 5. Precision when confidence is at least 0.80 | at least 0.85 on every seed | no prediction reached 0.80 (n = 0) | no prediction reached 0.80 (n = 0) | no prediction reached 0.80 (n = 0) | **MISSED** |

**Before and after, side by side** (the before numbers are from `results_three_seeds_before.json`, the after numbers from `results_three_seeds_after.json`):

| Measure | Before: seed 42 / 7 / 2024 | After: seed 42 / 7 / 2024 |
|---|---|---|
| Overall accuracy | 0.258 / 0.226 / 0.290 | 0.419 / 0.419 / 0.548 |
| Spread of accuracy across seeds | 0.065 | 0.129 |
| Macro F1 over all 21 labels (as the notebook prints it) | 0.061 / 0.059 / 0.056 | 0.144 / 0.178 / 0.299 |
| Macro F1 over the 5 core labels (Criterion 2) | 0.000 / 0.000 / 0.000 | 0.080 / 0.213 / 0.222 |
| Precision of `general_construction_non_mechanical` | 0.000 (never predicted) on all three | 1.000 / 1.000 / 0.500 |
| Number of different labels the model predicted | 2 / 2 / 1 | 4 / 4 / 4 |
| Predictions at confidence 0.80 or more | 0 / 0 / 0 | 0 / 0 / 0 |
| Average confidence when right | 0.088 / 0.100 / 0.124 | 0.264 / 0.367 / 0.369 |
| Average confidence when wrong | 0.087 / 0.101 / 0.120 | 0.183 / 0.253 / 0.282 |

**Full spread table for the after run, as printed by the notebook (section 6):**

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.419 | 0.419 | 0.548 | 0.129 |
| Macro F1 | 0.144 | 0.178 | 0.299 | 0.155 |
| F1 — `division_23_full_mechanical_hvac` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `sub_trade_specialty_sheet_metal_vs_hydronics` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `mep_package_mechanical_plumbing_electrical` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `general_construction_non_mechanical` | 0.400 | 0.667 | 0.444 | 0.267 |
| F1 — `general_pdf_cad_digitizers` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `specialized_mechanical_estimating_databases` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `enterprise_erp_bidding_platforms` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `emerging_ai_automation_tools` | 0.000 | 0.400 | 0.667 | 0.667 |
| F1 — `commercial_institutional_industrial` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `multi_family_residential_tract_housing` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `service_replacement_retrofit_bidding` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `public_works_prevailing_wage_bidding` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `software_benchmarking_technical_workflow` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `bid_strategy_scope_gaps_gc_relations` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `career_pathing_compensation_hiring` | 0.625 | 0.320 | 0.727 | 0.407 |
| F1 — `industry_venting_operational_burnout` | 0.560 | 0.750 | 0.556 | 0.194 |
| F1 — `senior_b2b_mechanical_estimator_chief_estimator` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `general_contractor_precon_coordinator` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `field_tech_residential_installer_apprentice` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `software_vendor_sales_representative` | 0.000 | 0.000 | 0.000 | 0.000 |
| F1 — `unrelated` | 0.000 | 0.000 | 0.000 | 0.000 |

**Confusion matrix, after run, seed 42** (the matrix the notebook's section 7 prints; rows are true labels, columns are the model's guesses). The matrices for seeds 7 and 2024 are in `results_three_seeds_after.json` under `confusion`.

| true \ predicted | division_23_full_mechanical_hvac | sub_trade_specialty_sheet_metal_vs_hydronics | mep_package_mechanical_plumbing_electrical | general_construction_non_mechanical | general_pdf_cad_digitizers | specialized_mechanical_estimating_databases | enterprise_erp_bidding_platforms | emerging_ai_automation_tools | commercial_institutional_industrial | multi_family_residential_tract_housing | service_replacement_retrofit_bidding | public_works_prevailing_wage_bidding | software_benchmarking_technical_workflow | bid_strategy_scope_gaps_gc_relations | career_pathing_compensation_hiring | industry_venting_operational_burnout | senior_b2b_mechanical_estimator_chief_estimator | general_contractor_precon_coordinator | field_tech_residential_installer_apprentice | software_vendor_sales_representative | unrelated |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **division_23_full_mechanical_hvac** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **sub_trade_specialty_sheet_metal_vs_hydronics** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **mep_package_mechanical_plumbing_electrical** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **general_construction_non_mechanical** | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| **general_pdf_cad_digitizers** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **specialized_mechanical_estimating_databases** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **enterprise_erp_bidding_platforms** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| **emerging_ai_automation_tools** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **commercial_institutional_industrial** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| **multi_family_residential_tract_housing** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **service_replacement_retrofit_bidding** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **public_works_prevailing_wage_bidding** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 |
| **software_benchmarking_technical_workflow** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| **bid_strategy_scope_gaps_gc_relations** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 |
| **career_pathing_compensation_hiring** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 5 | 2 | 0 | 0 | 0 | 0 | 0 |
| **industry_venting_operational_burnout** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 7 | 0 | 0 | 0 | 0 | 0 |
| **senior_b2b_mechanical_estimator_chief_estimator** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **general_contractor_precon_coordinator** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 | 0 |
| **field_tech_residential_installer_apprentice** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **software_vendor_sales_representative** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| **unrelated** | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->

**In one sentence: it helped, partly. Training for 15 epochs instead of 3 lifted accuracy on every seed (0.258, 0.226, 0.290 to 0.419, 0.419, 0.548) and made the model predict four labels instead of one or two, but it did not change a single verdict: Criteria 1, 2, 3 and 5 are still MISSED and Criterion 4 is still MET.**

**What the numbers say:**

- **Accuracy rose on all three seeds, by 0.16 to 0.26.** Before, the model was at or below the majority-class guess; now it is above it on every seed. That is the first evidence in this project that it learned something from post content, and it supports the Milestone 4 diagnosis that the short run (about 27 optimizer steps) was part of the cause.
- **It is not yet close to the target.** The best seed, 0.548, is still 0.10 below 0.65, and the other two are 0.23 below it.
- **The five core labels moved off zero, but not reliably.** Macro F1 went from 0.000 to 0.080, 0.213 and 0.222. The 95% bootstrap intervals for seeds 42 and 7 still start at 0.000, so for those two seeds I cannot say the core-label improvement is more than luck in which posts landed in the test split. Only seed 2024's interval (0.057 to 0.444) stays above zero, and every interval is far below 0.60.
- **`general_construction_non_mechanical` is now predicted, but on very few posts.** It was predicted 1, 2 and 4 times, giving precision 1.000, 1.000 and 0.500. The two 1.000 values rest on 1 and 2 predictions, so I do not read them as a pass. Seed 2024 misses the 0.75 target.
- **Confidence now means something, but it never reaches 0.80.** Before, the model was equally unsure when right and wrong (about 0.09 to 0.12 either way). Now the average confidence is higher when it is right (0.26 to 0.37) than when it is wrong (0.18 to 0.28), and the most confident third of predictions was right 70% to 80% of the time. Still, no prediction reached 0.80, so Criterion 5 remains unmeasurable and counts as MISSED.
- **The results got less stable.** Accuracy spread across seeds widened from 0.065 to 0.129, which the notebook flags as more than 10 points, a sign the data is too small or too lopsided for a steady measure.

**Against my prediction.** I said the model would predict more than two labels on at least two seeds (yes: 4, 4 and 4), accuracy would land around 0.30 to 0.45 (two seeds at 0.419 did; seed 2024 at 0.548 was higher than I expected), core-label macro F1 would leave 0.000 but stay far below 0.60 (yes), Criterion 1 would stay MISSED (yes), and some predictions would reach 0.80 (no, none did). I got the direction right and the confidence part wrong.

**What this does not show.** It was one change tested once per seed on a 31-post test split, so the size of the gain is uncertain even though all three seeds moved the same way. I did not test whether 15 epochs is the right number: I did not look at validation loss to see whether more epochs would help further or start to overfit, and I did not try a different learning rate. And the model still predicts only four of the 21 labels, mostly `career_pathing_compensation_hiring` and `industry_venting_operational_burnout`. That is what I expected if the data, with 12 of 21 labels having 3 or fewer posts, is the remaining limit. Training longer fixed part of the problem. It did not fix the part the data causes.


---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

Four criteria are still MISSED after the improvement (1, 2, 3 and 5); Criterion 4 is MET and needs nothing. The reason I stopped is the same for all four, so I give it once and then say what I would do for each.

**Why I stopped.** The course allows one change in this milestone, and I spent it on training length (EPOCHS 3 to 15), because that is what my diagnosis pointed at. The other fixes are bigger than one change: they need more data, and I have already reached the ceiling of what my scraper can collect. The three searches in `build_dataset.py` return about 204 unique r/estimators posts in total, and Reddit's public feed allows about one request per minute and caps a keyword search at roughly 105 posts. Getting more posts means a new source or a different scope for the dataset, which is a redesign and not a quick extra step. I am not claiming that I ran out of time. I stopped at the one-change limit and the data ceiling.

**What I would do for each missed criterion:**

| # | Still missed because | What I would do next | What I expect, and what could prove me wrong |
|---|---|---|---|
| 1. Accuracy (0.419 / 0.419 / 0.548, target 0.65) | The model still predicts only four of 21 labels, mostly `career_pathing_compensation_hiring` and `industry_venting_operational_burnout`. | First reduce the label set to the labels I can support: 12 of 21 labels have 3 or fewer posts, and 5 have none. Merge them into neighbours or drop them, and collect more posts for the ones that remain, aiming for at least 10 to 15 per label. Then retrain. | Accuracy should rise if the model's problem is thin labels. If it does not, the 15-epoch run was already limited by something else, such as the label definitions, and I would test that next. |
| 2. Macro F1 over the 5 core labels (0.080 / 0.213 / 0.222, target 0.60) | Each core label has 10 to 23 posts and only 1 to 5 land in a test split, so each score is very noisy. The intervals for seeds 42 and 7 still start at 0.000. | The same data work, plus a larger test split (or repeated splits) so one wrong post does not move a label's F1 by 0.2 or more. | The target is measurable but may be out of reach until the dataset is much bigger. I would not lower it, because a missed number gets diagnosed and not renegotiated. |
| 3. Precision of `general_construction_non_mechanical` (1.000 / 1.000 / 0.500, target 0.75) | Seed 2024 reached only 0.500, and the 1.000s rest on 1 and 2 predictions. | More posts of the non-HVAC trades, and a check of how many predictions the label gets per seed. Precision on fewer than about 5 predictions should be reported with its count and not as a pass. | This is the closest criterion to passing. If more data gives the label more predictions and precision stays at 0.75 or above on every seed, it would pass. |
| 5. Precision at confidence 0.80 or more (no prediction reached 0.80, target 0.85) | The average winning probability is only 0.26 to 0.37 even when the model is right, so nothing gets near 0.80. | Train longer with validation loss tracked (which I did not do), try a different learning rate, and consider fewer labels, since a 21-way softmax spreads probability thinly across many labels. | I do not know that any of these gets confident predictions on 204 posts. If none does, this criterion may need a bigger dataset, not a different setting. |

**The one check I would do first, before any of the above:** hand-label a random 30 of my own 204 posts without looking at Gemini's labels and compute kappa against them. That is the only way to find out whether my HVAC labels are consistent, and it decides whether more training data of the same kind would help or whether the label definitions need fixing first. I have not done it, so none of the rows above depends on its result.



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->

I meant the 21 labels to separate posts along five different axes (trade scope, software, project sector, post purpose and poster role), but in the after-run seed-42 matrix the model got 13 of 31 posts right and 12 of those 13 are `career_pathing_compensation_hiring` or `industry_venting_operational_burnout` posts, so what it actually learned is mostly one contrast between two post-purpose labels. It has no usable boundary for scope, sector or poster role, the axes an estimator would act on: 14 of its 18 mistakes in that matrix landed on those same two labels, and the other four went to `software_benchmarking_technical_workflow`, which is also a post-purpose label.



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**

🧭 **Full project diary, decisions, and architecture reference: [PROJECT_WALKTHROUGH.md](PROJECT_WALKTHROUGH.md)**

🛠️ **Full environment setup guide: [ENVIRONMENT_SETUP.md](ENVIRONMENT_SETUP.md)**

🤖 **DistilBERT fine-tuning guide (full adaptation, LoRA analysis, and implementation): [DistilBERT_LoRA_and_full_fine_tuning_explained.md](DistilBERT_LoRA_and_full_fine_tuning_explained.md)**