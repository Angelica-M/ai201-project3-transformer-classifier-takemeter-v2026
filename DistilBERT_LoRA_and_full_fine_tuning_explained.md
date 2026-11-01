# DistilBERT, LoRA and Full Fine-Tuning Explained: What This Project Actually Uses

> **Short answer for distinction of concepts:** TakeMeter fine-tunes **DistilBERT** with
> **full fine-tuning**. **LoRA is not used anywhere in this repository.**
> DistilBERT is a *model*; LoRA is a *training method* you could apply to it.
> They are not alternatives to each other, and one is not a kind of the other.

---

## Table of contents

1. [The one idea to hold on to](#1-the-one-idea-to-hold-on-to)
2. [DistilBERT: the model](#2-distilbert-the-model)
3. [Full fine-tuning: how this project trains](#3-full-fine-tuning-how-this-project-trains)
4. [LoRA: a different way to train the same model](#4-lora-a-different-way-to-train-the-same-model)
5. [Side-by-side comparison](#5-side-by-side-comparison)
6. [What this repo actually uses, with evidence](#6-what-this-repo-actually-uses-with-evidence)
7. [The other models in the project (none use LoRA)](#7-the-other-models-in-the-project-none-use-lora)
8. [How to tell which method a piece of code uses](#8-how-to-tell-which-method-a-piece-of-code-uses)
9. [Would LoRA help your project?](#9-would-lora-help-your-project)
10. [If you wanted to try LoRA anyway](#10-if-you-wanted-to-try-lora-anyway)
11. [Common misunderstandings](#11-common-misunderstandings)
12. [Glossary](#12-glossary)
13. [Where "LoRA" wrongly appears in this repo](#13-where-lora-wrongly-appears-in-this-repo)

---

## 1. The one idea to hold on to

There are two separate questions, and people often merge them:

| Question | Possible answers |
|---|---|
| **Which model** do I start from? | DistilBERT, BERT, RoBERTa, LLaMA, GPT, ... |
| **How** do I train it on my data? | Full fine-tuning, LoRA (and other parameter-efficient methods), prompting only (no training), ... |

- **DistilBERT** answers the first question.
- **LoRA** and **full fine-tuning** answer the second.

You can combine them freely:

| | Full fine-tuning | LoRA |
|---|---|---|
| **DistilBERT** | **← this project** | possible, not used here |
| **LLaMA, RoBERTa, ...** | possible | possible |

LoRA cannot run on its own. It has no language layers of its own; it is a small
add-on that is always attached to some base model.

---

## 2. DistilBERT: the model

DistilBERT is a pretrained language model released by Hugging Face. It is a
smaller, faster version of BERT made by **knowledge distillation**: a small
"student" network was trained to imitate the larger BERT "teacher".

Approximate facts (from the DistilBERT paper and the Hugging Face model card):

| Property | Value |
|---|---|
| Layers (transformer blocks) | 6 (BERT-base has 12) |
| Hidden size | 768 |
| Attention heads per layer | 12 |
| Parameters | about 66 million |
| Size vs. BERT-base | about 40% smaller |
| Speed vs. BERT-base | about 60% faster |
| Quality vs. BERT-base | keeps roughly 97% of BERT's score on the GLUE benchmark |
| File size (fp32 weights) | roughly 250 to 270 MB |

The checkpoint used here is **`distilbert-base-uncased`**. "Uncased" means it
lowercases everything, so "HVAC" and "hvac" look the same to it.

### What "pretrained" gives you

DistilBERT already learned general English (word meaning, grammar, which words
go together) from a large text corpus. It has **not** learned your task. It does
not know what `analysis` or `hvac_takeoff_and_estimating_scope` means. The
fine-tuning step teaches it that.

### What the classifier looks like

For classification, Hugging Face's `AutoModelForSequenceClassification` wraps
the pretrained DistilBERT with a small new **classification head**. For
DistilBERT the head is not a single layer. It is:

1. `pre_classifier`: a linear layer, 768 to 768
2. a ReLU and dropout
3. `classifier`: a linear layer, 768 to the number of labels (21 in this project)

The head starts **randomly initialised**. It is the only part of the model that
has never seen any text before you start training.

---

## 3. Full fine-tuning: how this project trains

In **full fine-tuning**, every weight in the network is trainable. Backpropagation
sends error gradients through the whole model, and the optimizer adjusts all
~67 million parameters (the pretrained layers and the new head) to reduce the
loss on your labelled posts.

```text
 [ Input text ]
       │
       ▼
┌──────────────────────────────────┐
│  DistilBERT base layers          │
│  (6 transformer blocks)          │   ALL weights are trainable and are
│  every pretrained weight updates │   changed by training
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  Classification head (new)       │   trained from random start
│  pre_classifier + classifier     │
└───────────────┬──────────────────┘
                │
                ▼
         [ 21 label scores ]
```

**Cost:** the optimizer (AdamW) keeps extra numbers for every trainable
parameter, so memory use is roughly several times the model's size. For a
66M-parameter model this is small enough to train on a laptop CPU, which is
exactly what happened here (about 3.6 minutes for three seeds).

**Output:** a complete separate copy of the model per task.

---

## 4. LoRA: a different way to train the same model

**LoRA** stands for **Low-Rank Adaptation** (Hu et al., 2021). Its idea: when you
adapt a big pretrained model to a new task, the *change* in each weight matrix
tends to be simple, meaning it can be described by far fewer numbers than the
matrix itself.

### The mechanism

Take one pretrained weight matrix `W` (frozen, never changed). LoRA learns a
correction made of two small matrices:

```text
output = W·x  +  (α / r) · B·(A·x)
          │                 │
          │                 └─ the LoRA path: trainable, tiny
          └─ the original path: frozen
```

- `A` shrinks the input from 768 dimensions down to `r` dimensions.
- `B` expands it back from `r` to 768.
- `r` is the **rank** (common values: 4, 8, 16).
- `α` (lora_alpha) is a scaling constant. The correction is scaled by `α / r`.
- `B` starts at **zero**, so at step 0 the model behaves exactly like the
  original. Training only moves it away as needed.

```text
          [ Input ]
        ┌─────┴─────┐
        ▼           ▼
 ┌────────────┐ ┌──────────┐
 │  Original  │ │ Matrix A │   (768 → r)   trainable
 │  weight W  │ └────┬─────┘
 │  FROZEN    │      ▼
 │            │ ┌──────────┐
 │            │ │ Matrix B │   (r → 768)   trainable
 └─────┬──────┘ └────┬─────┘
       └──────┬──────┘
              ▼
          [ added ]
```

### Where it is attached

LoRA is not applied to the whole network. You choose which layers get an
adapter, usually the attention projections. In DistilBERT these layers are
named `q_lin`, `k_lin`, `v_lin` and `out_lin`; a common choice is
`target_modules=["q_lin", "v_lin"]`.

### A worked parameter count for this project

If you applied LoRA with `r = 8` to `q_lin` and `v_lin` of DistilBERT:

| Part | Calculation | Trainable parameters |
|---|---|---|
| One adapter (A + B) on one 768×768 layer | 768×8 + 8×768 | 12,288 |
| 12 adapters (6 blocks × 2 layers) | 12 × 12,288 | 147,456 |
| Classification head (kept trainable) | 768×768+768 and 768×21+21 | about 606,700 |
| **Total trainable** | | **about 754,000** |
| Total model size | | about 67 million |
| **Trainable share** | | **about 1.1%** |

Note that in a classifier, **the head is trained fully either way**, so it
makes up most of LoRA's trainable parameters here.

### Practical details

- **Saving:** only the adapter and head are saved, a few MB. The base model is
  not copied per task.
- **Speed at inference:** the extra path adds a little work, but you can
  **merge** the adapter into `W` afterwards (`merge_and_unload()` in the PEFT
  library). A merged model runs at exactly the speed of the original.
- **Learning rate:** LoRA is usually trained with a higher learning rate
  (often around 1e-4 to 1e-3) than full fine-tuning (about 2e-5 to 5e-5),
  because the adapter starts at zero and has fewer parameters to move.
- **Library:** Hugging Face PEFT (`pip install peft`).

---

## 5. Side-by-side comparison

| Property | Full fine-tuning of DistilBERT | DistilBERT + LoRA |
|---|---|---|
| Pretrained weights | Changed by training | Frozen, unchanged |
| Trainable parameters | about 100% (about 67M) | about 1% to 2% |
| What is new at the end | A whole modified model | A small adapter + head (a few MB) |
| Training memory | Higher (optimizer state for all weights) | Lower |
| Training speed on small models | Fine | Not noticeably faster on tiny models |
| Inference speed | Baseline | Same after merging; slightly slower if not merged |
| Several tasks | One full model per task | One base model plus one small adapter per task |
| Accuracy | Strongest option when you have enough data | Often close; can trail on hard tasks or with a low rank |
| Extra tooling | None beyond Transformers | PEFT library, more settings (`r`, `α`, target layers) |
| Suits | Small models, enough data, simple setup | Large models, limited GPU memory, many tasks |

---

## 6. What this repo actually uses, with evidence

**Conclusion: full fine-tuning of DistilBERT. No LoRA.**

| Evidence | What it shows |
|---|---|
| `takemeter.ipynb` loads the model with `AutoModelForSequenceClassification.from_pretrained(base_model)` | A plain Hugging Face classifier with all weights trainable |
| The notebook trains it with the Hugging Face `Trainer` | Standard full-parameter training |
| No `peft`, `LoraConfig`, or `get_peft_model` anywhere in the code | LoRA's required ingredients are absent |
| `requirements.txt` does not list `peft` | The library LoRA needs is not even installed by the project |
| Settings used: 3 epochs, learning rate 2e-5, batch size 16, 128 tokens | A typical *full* fine-tune learning rate; LoRA would normally use a higher one |
| Run time: about 3.6 minutes for three seeds on a CPU | A full fine-tune of a 66M-parameter model is cheap enough that LoRA is not needed |

You can confirm this yourself in one line, after the model is built in the
notebook:

```python
trainable = sum(p.numel() for p in model.parameters() if p.requires_grad)
total = sum(p.numel() for p in model.parameters())
print(f"trainable {trainable:,} of {total:,} = {trainable/total:.1%}")
```

A full fine-tune prints about **100%**. A LoRA setup prints about **1%**.
(Do not add this to the graded notebook unless you want to; it is only a check.)

---

## 7. The other models in the project (none use LoRA)

This project involves **three** models, which is part of why it is easy to
mix things up:

| Model | Role | Trained by you? | LoRA? |
|---|---|---|---|
| **Gemini** (`gemini-3.1-flash-lite`) | Wrote the first draft of the 204 labels in `build_dataset.py` | No (called through an API) | No |
| **DistilBERT** (`distilbert-base-uncased`) | The classifier that is trained in `takemeter.ipynb` and judged by the five criteria | **Yes, fully fine-tuned** | **No** |
| **BART-large-MNLI** (`facebook/bart-large-mnli`) | The zero-shot baseline in `baseline.py`; it is never trained on your data | No | No |

---

## 8. How to tell which method a piece of code uses

**Full fine-tuning** (this project):

```python
from transformers import AutoModelForSequenceClassification, Trainer

model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased", num_labels=21
)
trainer = Trainer(model=model, ...)   # trains every weight
```

**LoRA** (not in this project):

```python
from transformers import AutoModelForSequenceClassification, Trainer
from peft import LoraConfig, get_peft_model, TaskType

model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased", num_labels=21
)
config = LoraConfig(
    task_type=TaskType.SEQ_CLS,        # keeps the classification head trainable
    r=8,
    lora_alpha=16,
    lora_dropout=0.1,
    target_modules=["q_lin", "v_lin"],  # DistilBERT's attention layers
)
model = get_peft_model(model, config)   # freezes the base, adds adapters
model.print_trainable_parameters()      # prints roughly 1% trainable
trainer = Trainer(model=model, ...)
```

The tell is simple: **LoRA code imports `peft` and wraps the model.** If you
don't see that, it is not LoRA.

---

## 9. Would LoRA help your project?

**Almost certainly not, because LoRA solves a different problem from yours.**

LoRA reduces **memory and storage cost**. Your results show a **data and
learning** problem:

- 204 posts over 21 labels (about 10 per label), with 5 labels having no posts
  and 3 having none in training.
- Across three seeds the model behaved like a near-constant guesser
  (seed 2024 predicted `career_pathing_compensation_hiring` for all 31 test
  posts), and no prediction reached 0.80 confidence.

A model that is not learning cannot be fixed by training *fewer* parameters.
If anything, LoRA with only 204 posts could learn even less in 3 epochs.

The likelier fixes sit in the data and the training setup:

| Idea | Why it targets your actual problem |
|---|---|
| More posts for labels with few or none | The model cannot learn a label it never sees |
| Merge or drop labels that cannot be supported by the data | Fewer, better-supported labels |
| More epochs and/or class weighting | The model currently settles on the most common labels |
| Stratified split that keeps every label in training | Removes the zero-train labels |
| Clearer label definitions | Reduces label noise that no training method can undo |

LoRA would make sense if you were adapting a model with billions of parameters
on a small GPU, or serving many different classifiers from one base model.
Neither is true here.

---

## 10. If you wanted to try LoRA anyway

It would be a legitimate experiment, but treat it carefully:

1. **It changes the method, not the diagnosis.** Milestone 5 asks for **one**
   change tied to a diagnosed miss. "Switch to LoRA" is only defensible if your
   diagnosis says memory or overfitting was the cause, and the evidence so far
   does not.
2. **Change nothing else.** Keep the same data, split, seeds and epochs, so any
   difference is due to LoRA. Because LoRA wants a higher learning rate, a
   fair comparison needs that noted and justified.
3. **Setup needed:** `pip install peft`, add it to `requirements.txt`, and wrap
   the model as in section 8.
4. **Expect:** about 1% trainable parameters, and results that are similar or
   worse than full fine-tuning on this small dataset.
5. **Report honestly:** a LoRA run that scores no better is a valid finding.

If you do this, the README and `criteria.md` could then truthfully say
"LoRA"; until then they should not.

---

## 11. Common misunderstandings

| Claim | Reality |
|---|---|
| "DistilBERT is a kind of LoRA." | No. DistilBERT is a model; LoRA is a training method. |
| "I have to choose DistilBERT or LoRA." | No. You choose a base model *and* a training method. |
| "Fine-tuning a classifier means LoRA." | No. Fine-tuning is the general idea; LoRA is one way to do it. The default in Hugging Face is full fine-tuning. |
| "LoRA is a model I can download and run." | A LoRA adapter file is useless without its base model. |
| "LoRA always matches full fine-tuning." | Often close, not guaranteed, and it can lag on small encoders or with a low rank. |
| "A smaller trainable share means a smaller model." | The base model is the same size; LoRA only trains less of it. |
| "LoRA fixes a small dataset." | It cuts cost, not data requirements. |
| "My notebook output says 'fine-tuning', so it's LoRA." | "Fine-tuning" covers both. Check for `peft` in the code. |

---

## 12. Glossary

| Term | Meaning |
|---|---|
| **Pretrained model** | A model already trained on large general text |
| **Fine-tuning** | Further training a pretrained model on your task data |
| **Full fine-tuning** | Fine-tuning where every weight can change |
| **PEFT** | Parameter-Efficient Fine-Tuning: methods that train only a small part of a model. LoRA is the most popular one. Also the name of the Hugging Face library |
| **LoRA** | Low-Rank Adaptation: frozen base weights plus small trainable matrices A and B |
| **Adapter** | The small trainable add-on (LoRA's A and B matrices) saved separately from the base model |
| **Rank (`r`)** | The small inner dimension of A and B; bigger means more capacity and more parameters |
| **`lora_alpha`** | Scaling constant; the adapter's output is multiplied by `α / r` |
| **Classification head** | The new layer(s) on top of the base model that output label scores |
| **Distillation** | Training a small "student" model to imitate a larger "teacher"; how DistilBERT was made |
| **Zero-shot** | Using a model on a task with no training on that task, as `baseline.py` does |
| **Frozen** | Weights that training is not allowed to change |
| **Merging** | Folding a LoRA adapter into the base weights to remove its runtime cost |

---

## 13. Where "LoRA" wrongly appears in this repo

These files call the classifier a "LoRA classifier" or "LoRA adapter", which
does not match the code. They should say "fine-tuned DistilBERT classifier":

- `criteria.md`: the intro to the five criteria, the "Why this target" text for
  Criterion 1 and Criterion 4, the "too hard" reasoning for Criterion 5, and
  the section titled "Low-Rank Adaptation (LoRA) Concepts' Explanations".
- Your earlier README prediction used the phrase "LoRA adapter"; the
  explanation file already notes that this was inaccurate.

Calling the project a "LoRA assignment" is understandable, since these words
were in your own files, but the course materials and the code describe a
fine-tuned DistilBERT classifier.

---

*Figures such as parameter counts and file sizes are approximate; exact values
depend on the checkpoint, the number of labels, and the LoRA settings chosen.*
