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

**Anything I changed from the defaults, and why:** For the Unit 5 run I left
every training setting at its default. (In Unit 6 I changed epochs from 3 to 10
as my one improvement; see The Improvement.) Two things differ from the starter
notebook:

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
| Overall accuracy | 0.385 | 0.538 | +0.154 |
| Macro F1 | 0.308 | 0.533 | +0.225 |
| F1 — `check` | 0.333 | 0.333 | -0.000 |
| F1 — `practical` | 0.400 | 0.333 | -0.067 |
| F1 — `problem` | 0.500 | 0.800 | +0.300 |
| F1 — `share` | 0.000 | 0.667 | +0.667 |

Both models were scored on the same 13 real posts in `test_split.csv` (seed 42).
The baseline is `facebook/bart-large-mnli` using my definitions from
`label_definitions.txt`. A second run with bare label names scored accuracy
0.308 and macro F1 0.216 (`baseline_bare_names.json`), so my definitions were
worth about 8 accuracy points, which is one post out of 13. With 2 to 5 test
posts per label, none of the per-label gaps are reliable.

**What I predicted before I looked:** "I dont think the model will work very
well and may require some necessary work over on the training data." This is a
general prediction, not a claim about which label the trained model would win
on. It was not written blind: before I wrote it I had already been shown the
trained model's seed-42 results (accuracy 0.538) and the two baseline scores.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | at least 0.65 on every seed | 0.538 | 0.385 | 0.385 | MISSED |
| 2. `problem` recall | at least 0.70 on every seed | 0.67 | 1.00 | 1.00 | MISSED |
| 3. Per-label F1 (`problem` / `check` / `practical` / `share`) | at least 0.55 for `problem`, `check` and `share`; at least 0.40 for `practical`, on every seed | 0.80 / 0.33 / 0.33 / 0.67 | 0.75 / 0.00 / 0.40 / 0.00 | 0.55 / 0.00 / 0.57 / 0.00 | MISSED |
| 4. Label balance in `labels.csv` | no label under 20% or over 40% | 36% / 26% / 21% / 17% (`check` / `share` / `problem` / `practical`), the same file for every seed | same | same | MISSED |
| 5. Agreement with staff on 30 posts | at least 21 of 30 | 30 of 30 (not seed-dependent) | same | same | MET, with a caveat (see Agreement Report) |

Device for all three seeds: `cpu` (PyTorch 2.14.1+cpu), from
`results_three_seeds_before.json`. Each seed's test split is 13 real posts
(`problem` 3, `check` 5, `practical` 2, `share` 3); 119 synthetic posts are in
the training split for every seed.

Spread across the three seeds (from the notebook):

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.538 | 0.385 | 0.385 | 0.154 |
| Macro F1 | 0.533 | 0.287 | 0.279 | 0.254 |
| F1 — `problem` | 0.800 | 0.750 | 0.545 | 0.255 |
| F1 — `check` | 0.333 | 0.000 | 0.000 | 0.333 |
| F1 — `practical` | 0.333 | 0.400 | 0.571 | 0.238 |
| F1 — `share` | 0.667 | 0.000 | 0.000 | 0.667 |

### Confusion matrix

Seed 42, 13 test posts.

| true \ predicted | problem | check | practical | share |
|---|---|---|---|---|
| **problem** | 2 | 0 | 0 | 1 |
| **check** | 0 | 1 | 3 | 1 |
| **practical** | 0 | 0 | 1 | 1 |
| **share** | 0 | 0 | 0 | 3 |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->

On seed 42, 3 of the 5 real `check` posts were called `practical` and none of
the `practical` posts were called `check`, so the model has not learned the
boundary between asking for a verdict and asking for information, and it leans
toward `practical`. That is a single seed with only 2 `practical` posts in the
test split, and on seeds 7 and 2024 the `check` posts were called `problem` and
`practical` instead, so I read it as `check` being the label the model cannot
find, not as a stable `check`/`practical` confusion.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | at least 0.65 on every seed | MISSED | 0.538, 0.385 and 0.385. Below 0.65 on all three seeds. |
| 2 | `problem` recall | at least 0.70 on every seed | MISSED | 0.67 on seed 42, which is 2 of 3 test posts, then 1.00 and 1.00. It held on two seeds but not seed 42, so it fails the all-three-seeds rule. |
| 3 | Per-label F1 | at least 0.55 for `problem`, `check` and `share`; at least 0.40 for `practical` | MISSED | On seed 42, `check` is 0.33 and `practical` is 0.33. On seeds 7 and 2024, `check` and `share` are both 0.00. No seed met the target for every label. |
| 4 | Label balance | no label under 20% or over 40% of `labels.csv` | MISSED | `practical` is 17% (14 of 81). The other three are inside the range at 36%, 26% and 21%. This is measured on `labels.csv` only; the synthetic posts are not in that file. |
| 5 | Agreement with staff | at least 21 of 30 | MET, with a caveat | 30 of 30 matched. But the 30 labels were assigned by Claude in id blocks, not by me, so this doesn't show that I applied the rule consistently. See the Agreement Report. |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->

**What the agreement report can and can't tell me here.** It can't separate a
labelling problem from a model problem for me, because the 30 labels were
Claude's, not mine (see the Agreement Report). So the evidence below comes from
the training logs, the predicted-class counts per seed, and the label counts,
and I have not tested whether my own 81 labels were consistent. They are also
unreviewed pre-labels, which is a second reason to be careful with any claim
that the labels are fine.

**Criterion 1 (accuracy, MISSED on all three seeds).** Cause: the model barely
trained, and I can't tell whether that is too few real examples or a task out
of reach at this training budget. Evidence: the final training loss was 1.288,
1.328 and 1.281 on seeds 42, 7 and 2024, against 1.386 for guessing uniformly
among four labels, and validation accuracy stayed between 0.25 and 0.42 on every
epoch. Three epochs of about 11 steps each is 33 optimiser steps. On top of that
the test split is 13 posts, so one post moves accuracy by 7.7 points.

**Criterion 2 (`problem` recall, MISSED on seed 42 only).** Cause: partly a test
split too small to measure, partly a model that guesses `problem` too often.
Evidence: seed 42's 0.67 is 2 of 3 `problem` posts, so one post decides it. On
seeds 7 and 2024 recall is 1.00, but the model called 5 and 8 of the 13 posts
`problem`, with precision of only 0.60 and 0.38, so those passes come from
over-predicting the label and do not show it was learned. The criterion as
written can be met by always answering `problem`, which is a weakness in how I
set it.

**Criterion 3 (per-label F1, MISSED on every seed).** Cause: the model collapsed
onto two labels on two of the seeds, and `check` is a hard pair with
`practical`. Evidence: on seeds 7 and 2024 the model never predicted `check` or
`share` at all (0 of 13 each), so both score F1 0.00. On seed 42 it predicted
`check` once, and 3 of the 5 real `check` posts went to `practical`, which is
the pair my decision rule was hardest to apply to. `practical` has only 2 test
posts, so its F1 of 0.33 to 0.57 swings with one post. I have a hypothesis that
the synthetic posts taught it cues that real posts don't share (95% of my
synthetic `practical` posts contain a question mark against 43% of the real
ones, and they are far shorter), but I haven't tested it, for example by
training with `USE_SYNTHETIC = False`.

**Criterion 4 (label balance, MISSED).** Cause: too few examples of
`practical`. Evidence: `practical` is 14 of 81 posts, 17%, against my 20% floor.
I did not collect more real posts, so the shortfall is mine. It also explains
why `practical` has only 2 posts in the test split.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** 30 / 30 = 100%

**Who labelled the 30, and why that matters:** The 30 labels in
`my_staff_labels.csv` were assigned by Claude under the staff taxonomy, not by
me, and the `note` column says so on every row. Claude assigned them in id blocks
(sp_001–010 `analysis`, sp_011–020 `hot_take`, sp_021–030 `reaction`) after
noticing that `staff_posts.csv` is sorted that way, so it also had the layout to
go on. A 100% match is what that would produce, and it is not independent
evidence about how well *I* apply a written rule. It also leaves me with no
direct measure of whether my own `labels.csv` labels were consistent, which is
what this report is meant to tell me. I am not claiming the set was easy or the
staff rules are tight, only that this run can't say.

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

There were no disagreements, so there is nothing to sort into piles (a), (b) or
(c). `agreement_results.json` records 30 agreed and 0 disagreed.

**What the pattern in my disagreements tells me:** With no disagreements there
is no pattern to read. The two things I can say are that the result doesn't tell
me my own labelling was consistent, and that it doesn't help me separate a
labelling problem from a model problem in my diagnoses. I should treat the
consistency of my 81 labels as untested.


---

## The Improvement

**What I changed:** One training setting: `EPOCHS` from 3 to 10, in
`takemeter.ipynb`. Nothing else changed: the same four labels, the same 81 real
posts, the same 119 synthetic posts in training only, learning rate 2e-5, batch
size 16, and the same seeds 42, 7 and 2024, with `RUN_LABEL = "after"`. I tried
10 once, before seeing any after numbers, and tried no other value, so it was
not tuned against the test set. The number 10 is my choice and somewhat
arbitrary.

**Which diagnosis pointed at it:** The criterion 1 diagnosis. The final training
loss in the before run was 1.288, 1.328 and 1.281 against 1.386 for guessing
uniformly, after only 33 optimiser steps, so the model had barely trained. I
picked this over relabelling because the agreement report couldn't point at a
boundary I had applied inconsistently, and over adding posts because I could
not collect more real ones.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | at least 0.65 on every seed | 0.846 | 0.846 | 0.846 | MET |
| 2. `problem` recall | at least 0.70 on every seed | 0.67 | 0.67 | 1.00 | MISSED |
| 3. Per-label F1 (`problem` / `check` / `practical` / `share`) | at least 0.55 for `problem`, `check` and `share`; at least 0.40 for `practical`, on every seed | 0.80 / 0.89 / 0.80 / 0.86 | 0.80 / 0.91 / 0.80 / 0.80 | 0.75 / 0.75 / 1.00 / 1.00 | MET |
| 4. Label balance in `labels.csv` | no label under 20% or over 40% | 36% / 26% / 21% / 17% (`check` / `share` / `problem` / `practical`), unchanged | same | same | MISSED |
| 5. Agreement with staff on 30 posts | at least 21 of 30 | 30 of 30, unchanged | same | same | MET, with a caveat (see Agreement Report) |

Device for all three seeds: `cpu`. Final training loss was 0.674, 0.772 and
0.726. Macro F1 was 0.837, 0.827 and 0.875. Each seed's test split is 13 real
posts, as before. The raw numbers are in `results_three_seeds_after.json`.

Seed 42 confusion matrix, after:

| true \ predicted | problem | check | practical | share |
|---|---|---|---|---|
| **problem** | 2 | 0 | 0 | 1 |
| **check** | 0 | 4 | 1 | 0 |
| **practical** | 0 | 0 | 2 | 0 |
| **share** | 0 | 0 | 0 | 3 |

**Did it help, and how do I know:** Yes. Accuracy went from 0.538, 0.385 and
0.385 to 0.846 on all three seeds, macro F1 from 0.533, 0.287 and 0.279 to
0.837, 0.827 and 0.875, and criteria 1 and 3 went from MISSED to MET. The model
now predicts all four labels on every seed, where before it never predicted
`check` or `share` on seeds 7 and 2024. Criteria 2 and 4 are still missed,
because `problem` recall is 2 of 3 on two seeds and I did not touch the data. I
would not read 0.846 as a precise number: the test split is 13 posts, so one post
is 7.7 points and 11 of 13 could plausibly be anywhere from about 0.7 to 1.0 on
a different draw. I also can't say how much of the gain depends on the synthetic
posts, since they were in training both times.



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->

**Criterion 2 (`problem` recall).** Still missed: 0.67, 0.67 and 1.00. I would
collect more real `problem` posts so the test split holds more than 3 of them,
because as it stands one post decides whether the target is met. I stopped
because I only have 81 real posts and could not read Reddit to collect more. I
would also rewrite the criterion, since "always answer `problem`" would meet it.

**Criterion 4 (label balance).** Still missed: `practical` is 14 of 81, 17%.
Three more real `practical` posts would bring it to about 20%. I stopped
because I haven't collected any more real posts, and I didn't want to fix it
with synthetic ones, since the criterion is about `labels.csv`.

**Criterion 5 (agreement, MET with a caveat).** The 30 staff-set labels were
Claude's, so this doesn't show I apply a rule consistently. What I would do is
label the 30 myself, or get a different set from my TF. I stopped because the
agreement run had already shown the answer, so a fresh attempt on the same set
would not be independent.

**Other limits I know about:** the 81 labels are Claude's pre-labels and I have
not reviewed them. The held-out set is 13 posts, so every score here has a wide
error bar. The model trains on 119 synthetic posts and I haven't tested a run
without them.

**The gap between what I meant my labels to capture and what the model
learned:** I meant `problem` to capture a current physical symptom and `check`
to capture a verdict with no symptom, and the seed 42 matrix now has 2 of 3
`problem` and 4 of 5 `check` posts right. The one error on each is a `problem`
post called `share` and a `check` post called `practical`, which looks like the
boundary being learned. With so few test posts and unreviewed labels, though, I
can't tell whether the model learned my definitions or the way Claude applied
them.



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
