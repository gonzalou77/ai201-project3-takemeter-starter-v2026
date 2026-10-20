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

r/stretched is a public Reddit community for people stretching their ear
lobes, septums and other piercings. Almost every post is a question or a
show-and-tell, and what separates them is whether something is physically wrong
right now, whether the writer wants a verdict, or whether they just want
information or are sharing. My classifier sorts each post into one of four
labels: `problem`, `check`, `practical` or `share`.



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

### `problem`

**Definition:** The writer reports a current physical symptom — blood, pain,
swelling, wetness, a tear, a prolapse or a blowout.

**Example 1:**
> I woke up this morning, take out the jewelry, and there is blood. I feel like
> this is not normal. What should i do?

**Example 2:**
> my left lobe is sore and stings when cleaning it. No blood, but a small tear,
> no swelling.

### `check`

**Definition:** The writer wants a verdict — is it okay, normal, safe or
possible? — and reports no current symptom.

**Example 1:**
> Just wanting to check if my lobes look ok as they seem to shrink a fair bit
> when I take jewellery out? They don't feel irriated/painful and I stretch
> using half sizes.

**Example 2:**
> Basically im just asking if i have enough space on both ears? Like enough
> skin? Or would it tear?

### `practical`

**Definition:** The writer wants information, instructions or a product, not a
verdict.

**Example 1:**
> Are there any kits I can purchase online because all titanium plugs stop up
> to 10mm

**Example 2:**
> I really want to start stretching, but I have no idea what I'm doing. I want
> to go to either a 6g or a 4g. Please let me know the best and safest way to do
> it.

### `share`

**Definition:** The writer shows a result, story or setup and asks for
nothing.

**Example 1:**
> I finally made it to the end of my stretching journey guys!!! […] I'm
> officially a 00g 10mm and and I shall stay a 00g 10mm

**Example 2:**
> Honestly, I checked it with my current setup with dark purple and opal and...
> I don't hate it, I feel like it has potential to work.

### The hardest boundary

**Which two labels:** `problem` and `check`

**The decision rule I used every time:**

If the post reports something physically wrong now — blood, pain, swelling,
wetness, tearing, a prolapse, a blowout, or a condition that is still present
(skin that is still thin after a blowout) — it is `problem`. If the writer only
worries something might be wrong, or asks whether it looks or is okay, and
reports no symptom, it is `check`. **A mistake the writer admits to is not a
symptom.** "WAY too fast stretching" says "i am VERY aware of how badely i messed
up… how cooked am i?" but names no symptom, so it is `check`, even though the
tone sounds like `problem`.



---

## The Dataset

**Where the posts came from:** 81 public text posts from r/stretched, copied by
hand into `labels.csv` with each post's title and body joined into one `text`
field. That is short of the 200 the brief asks for. To give the model more to
train on I also have 119 Claude-generated synthetic posts in
`data/synthetic_posts.csv`. They are **not** in `labels.csv`, they are added to
the training split only, and they never appear in validation or test, so every
held-out number is measured on real posts. `results.json` records how many were
used.

**How I labelled them:** All 81 labels were produced by Claude from the
definitions in this README, as a pre-labelling pass. I did not label a cold set:
the 20 posts I had set aside to label unaided were labelled by Claude at my
request, so I have no cold labels, and the `note` column in `labels.csv` says
"pre-labelled by Claude" on every row. As of this commit I have not yet read and
corrected them. The synthetic posts were also written by Claude, each with an
intended label.

**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `check` | 29 | 36% |
| `share` | 21 | 26% |
| `problem` | 17 | 21% |
| `practical` | 14 | 17% |
| **Total** | 81 | 100% |

No label is above 70%. `practical` is under the 20% floor I set in criterion 4.

**Three hard cases**

**1.**
> *The post:* "WAY too fast stretching — so first up i wanna say that i am VERY
> aware of how badely i messed up. second, in two weeks i managed to strech to
> 10 mm. i dont want to make this long, how cooked am i?"
>
> *Could have been:* `problem` or `check`
>
> *I chose:* `check`, because the writer admits a mistake but names no symptom.
> My decision rule says an admitted mistake is not a symptom, even though the
> tone sounds like `problem`.

**2.**
> *The post:* "UPDATE: I joined the 2in club and got my ear/ear lobe tattooed —
> I have a down sized plug in at the moment because I got the bottom of my ear
> lobe tattooed as well and its a little swollen but im PUMPED on the end
> result"
>
> *Could have been:* `share` or `problem`
>
> *I chose:* `share`, because the swelling is mentioned in passing and the post
> is celebrating and asks for nothing. My `problem` definition ("reports a
> current physical symptom") technically covers it, so the written rule doesn't
> settle this one. The call I made is that the symptom has to be what the post
> is about.

**3.**
> *The post:* "Stretch It Body Jewelry restock! — Hey yall! Just in case anyone
> was in my shoes lately, the stretching kit for 25-30 mm kit is back up!"
>
> *Could have been:* `share` or `practical`
>
> *I chose:* `share`, because the writer is giving information and asking for
> none. My `practical` definition is about wanting information or a product, and
> this post is the other direction, so neither definition fits cleanly.

---

## The Training Run

**Base model:** `distilbert-base-uncased`, fine-tuned for four-way sequence
classification.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42. Trained on CPU (PyTorch 2.14.1+cpu); the run took about 77
seconds.

**Anything I changed from the defaults, and why:** I left every training
setting at its default. Two things differ from the starter notebook:

- `LABELS` is now `problem, check, practical, share`.
- The 119 synthetic posts in `data/synthetic_posts.csv` are added to the train
  split after the split is made. I did this because I only have 81 real posts,
  which is too few to train on. Validation and test are cut from the real posts
  alone, so synthetic posts never reach a held-out number. `results.json`
  records `synthetic_in_train: 119` and `test_is_real_only: true`.

**Split sizes:** The 81 real posts are split 70/15/15, stratified, on seed 42.

| | train | val | test |
|---|---|---|---|
| Real posts | 56 | 12 | 13 |
| Synthetic posts | 119 | 0 | 0 |
| **Total** | 175 | 12 | 13 |

Per-label counts in the test split: `problem` 3, `check` 5, `practical` 2,
`share` 3. Every label has fewer than 8 test posts, and `practical` has 2, so a
per-label score here moves a lot with a single post, and I expect large
variance across seeds in Unit 6. About two thirds of the training data is
synthetic, which is another reason to read the held-out numbers carefully.



---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

**Moment 1 — designing the labels**

- *What I asked for:* I pasted the r/stretched posts I had collected and asked
  Claude to read them and suggest distinctions I could point at. Claude could
  not read Reddit itself, so everything it worked from came from my paste.
- *What came back:* Candidate distinctions with counts, such as whether
  something is physically wrong now, what kind of question is being asked, and
  how much detail the post gives. After my first batch of 43 posts it said
  showcase posts were too rare to be their own label.
- *What I changed:* I said "not yet, see the updated post list" and had it
  redo the analysis on all 81 posts. The larger set showed that about 20
  posts were showcase, so Claude reversed that point and `share` became a
  label. From there I picked problem vs. check as my hardest boundary, and kept
  four labels instead of folding `practical` into `check`. Claude drafted the
  definitions and the decision rule and I accepted them as written.

**Moment 2 — synthetic training posts**

- *What I asked for:* I had only 81 real posts, so I asked Claude to analyze
  the whole set and write synthetic posts and titles, then to make 119 more.
- *What came back:* 119 posts across the four labels in
  `data/synthetic_posts.csv`, along with a warning that they were cleaner and
  more separable than the real ones. For example, 95% of its `practical` posts
  had a question mark, against 43% of the real ones, and it said a model could
  learn those cues and look better than it is. It advised against putting them
  in `labels.csv` or the test split.
- *What I changed:* I asked it to build a hybrid. The synthetic posts are added
  to the training split only, and validation and test are real posts alone. The
  notebook change is in `takemeter.ipynb`, and `results.json` records both
  facts.

**Pre-labelling disclosure:** Every label in `labels.csv` was assigned by
Claude from my written definitions, including the 20 posts I had reserved to
label unaided. I asked Claude to label those 20 as well, so I have no cold
labels, and the `note` column says so on every row. I have not yet read and
corrected the 81 labels. Claude also wrote all 119 synthetic posts and their
intended labels. Separately, Claude drafted the five criteria in `criteria.md`
and their reasoning; I committed them as drafted without changing the numbers.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

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

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



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
