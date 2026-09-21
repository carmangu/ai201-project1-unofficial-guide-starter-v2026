# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:**
One of my five questions — "When is the deadline to
declare pass/fail?" — depends on the phrase "week eight," which appears
exactly once in the whole corpus. If the chunk boundary falls badly, that
phrase could end up in a chunk retrieval doesn't return, even though the
answer exists. The other four questions are anchored in short, self-contained
threads, so 4 of 5 is the honest target. 5 of 5 would be a claim I can't
defend before seeing results.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:**
Grounding without a source is just a model's guess.
My pipeline always retrieves at least one chunk before the model runs, and
my GROUNDING_INSTRUCTION explicitly requires the model to name the document
it used. The only way this fails is if the model ignores the instruction,
which is exactly what I want to catch. A target of 4 of 5 would let one
uncited answer through, and one is too many for a system whose value is
verifiability.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

**Why this target:**
The five OUT_OF_SCOPE questions are from entirely
different domains (Mongolia, diesel engines, football, medicine, Rust), so
their distances should be well above any in-corpus question. I pick 4 of 5
rather than 5 of 5 because one question could land just under the cutoff by
coincidence — distance is a heuristic, not a proof — and I'd rather report
an honest miss than pretend the gate is perfect.

---

## 4. Something about your chunks

Every chunk in my index either contains a complete THREAD (from its
`THREAD:` line to its last reply) or, if a thread was too long to fit in
one chunk, every sub-chunk carries the `THREAD:` title as a prefix. When I
sample 5 chunks at random, all 5 name the thread they belong to.

**Why this target:**
In Milestone 1 I saw the starter chunker produce a
2-character chunk and I noticed that any thread over 800 characters gets
cut mid-reply, with the second chunk losing its `THREAD:` title. Without
the title, a reader can't tell what discussion a reply belongs to — reply 3
saying "Both true" is meaningless without the thread it's responding to. I
pick 5 of 5 because this is a structural property of my splitter, not a
quality judgment.

---

## 5. Your choice

Every chunk preserves the vote count on each reply it contains (e.g.
`--- reply 1 (14 votes) ---`). When I check 3 chunks, all 3 show the vote
count for every reply they include.

**Why this target:**
The vote count is how a reader knows which replies the
community actually endorsed versus which are one person's opinion. In
thread_bike_commute.txt, reply 3 has 22 votes and says "Both true" — that's
meaningful because it's the highest-voted reply and it resolves the
disagreement between reply 1 and reply 2. If a chunk stripped the vote
counts, the reader would lose that signal. I pick 3 of 3 because this is a
formatting requirement, not a retrieval quality question.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->
