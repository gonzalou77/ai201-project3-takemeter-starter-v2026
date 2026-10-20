# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1. Overall accuracy is at least 0.65 on every seed

Accuracy on the held-out test split is at least 0.65 on each of the three seeds
(42, 7 and 2024).

**Why this target:** I have four labels, so guessing at random gets about 25%,
and always guessing my biggest label gets roughly 30%. 0.65 is more than double
chance. I did not go higher because two people applying written rules to
ambiguous posts only agree in the 70s, so a model beating 0.65 on about 30 test
posts would already be close to how consistently a person labels this. With 30
test posts, one post is 3.3 points, so I expect it to wobble and I am setting it
knowing it has to hold on all three seeds.

---

## 2. `problem` recall is at least 0.70 on every seed

Of the held-out posts I labelled `problem`, the model finds at least 70% of
them, on each of the three seeds.

**Why this target:** The mistake that matters most for this community is a post
reporting blood, swelling or a tear being called `check`, which is a
reassurance-seeking label. That is my hardest boundary (problem vs. check), and
recall is the number that shows which way the model errs. I set 0.70 rather than
a stricter number because `problem` will have only about 6 to 8 test posts, so
0.70 allows one or two misses and nothing more.

---

## 3. Per-label F1 is at least 0.55, and 0.40 for `practical`

On every seed, `problem`, `check` and `share` each have an F1 of at least 0.55
on the held-out set, and `practical` has an F1 of at least 0.40.

**Why this target:** `practical` is my smallest label. In my reading of 81 posts
it was about 16%, so it will have about 5 to 6 posts in the test split, and an
F1 on that few posts jumps between seeds, so I set it looser. The other three
get 0.55, a little above what overall accuracy of 0.65 implies for the weakest
of them, so one label cannot hide behind the others. `check` is the label I
expect to land closest to the line, because it sits on boundaries with both
`problem` and `practical`.

---

## 4. No label is under 20% or over 40% of `labels.csv`

Checked before training, on the full labelled file: every label is between 20%
and 40% of the rows.

**Why this target:** The brief caps a label at 70%, but with four labels 40% is
already lopsided, and it is the number I would call a problem. The floor is the
bigger deal. 20% of 200 is 40 posts, which gives `practical` about 6 test posts
and keeps it above the point where its score becomes noise. In the posts I have
read `practical` is naturally about 16%, so hitting this means collecting more
of those on purpose rather than taking the posts as they come.

---

## 5. I agree with staff on at least 21 of the 30 staff posts

My labels on the 30 staff posts, applied under the staff taxonomy, match staff
on at least 21 (70%).

**Why this target:** The staff note says a rate in the 70s is normal, because
several of the 30 were chosen to be ambiguous. 70% is the bottom of that range.
Below it, I would have no reason to trust that I applied my own rules the same
way across my 200, whatever the model scores. I am not setting it higher,
because the staff taxonomy is not mine and some of the posts are meant to be
argued.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
