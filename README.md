# The Unofficial Guide

Jingwen Gu, advice_threads

> **This file is your submission.** Fill it in as you go — most sections get
> written during the milestone that produces them, not at the end.
>
> How the starter works, and every command you'll need, is in `RUNNING.md`.
> Leave that file alone.
>
> **Paste everything as text.** No screenshots, no video. A typed table gets
> full credit; a picture of the same table gets none.
>
> Delete these instruction blocks as you replace them. The `<!-- -->` comments
> are notes to you and don't show up when the page renders — you can leave them
> or remove them.

---

# Unit 1

## What This Does

This is a RAG Q&A system over a advice threads corpus. Each thread is a `THREAD:` title followed by 2-5
replies with vote counts. A user asks a plain-language question — for
example, "How much RAM should a CS student's laptop have?" — and the
system retrieves the most relevant threads. The system will refuse with "I don't have
enough information about that" for not covered qusstions.

## Chunking Strategy

**Chunk size:** 800 characters
**Overlap:** None. Just natural boundaries
**Splitter:** `chunker.py::split_document

**Why this strategy:**

I read all of the documents in advice_threads and found they all share the same
structure: a `THREAD:` title followed by 2-5 replies, each marked with
`--- reply N (votes) ---`. Each thread is a single discussion — replies
depend on each other (reply 3 in `thread_bike_commute.txt` says "Both
true", which only makes sense in the context of replies 1 and 2). The
vote counts are part of the information: they tell a reader which replies
the community actually endorsed.

The starter chunker cut these on a fixed 800-character window with
120 characters of overlap. Because every thread over 680 characters
triggered a second window, three threads were split into a main chunk
plus a redundant tail chunk, and `thread_meal_plan_tier.txt` produced a
2-character fragment (`"t."`). The tail chunks lost the `THREAD:` title,
so a reader couldn't tell what discussion they belonged to.

My splitter uses thread boundaries as chunk boundaries. A thread under
800 characters becomes one chunk. No overlap is needed because `THREAD:`
is already a natural boundary. If a thread exceeds 800 characters (none
currently do), it splits on `--- reply N ---` boundaries and every
sub-chunk carries the `THREAD:` title as a prefix.


## Sample Chunks

**Chunk 1** — source: `thread_bike_commute.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Is a bike worth it for a 20 minute walk commute?

--- reply 1 (14 votes) ---
Yeah. Cuts an 18 minute walk to about 6. The thing nobody mentions is storage — covered bike parking exists at three buildings and is full by 9am at all three.

--- reply 2 (9 votes) ---
Counterpoint, I sold mine. Between November and March the paths are either icy or salted and salt destroys a drivetrain in one season.

--- reply 3 (22 votes) ---
Both true. I keep a cheap bike for September to November and walk the rest of the year. Total cost was about $120 for the bike and I don't care what happens to it.

--- reply 4 (5 votes) ---
If you do get one, the campus does free registration and it's the only reason I got mine back after it was taken.
```

**Chunk 2** — source: `thread_first_gen.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Anything specific for first-generation students?

--- reply 1 (33 votes) ---
The advising office has a specific programme and it is genuinely good, but it is opt-in and badly publicised. Ask for it by name.

--- reply 2 (41 votes) ---
The thing I'd say: the unwritten rules are the hard part, not the coursework. Ask about the unwritten rules explicitly. People are happy to explain them and nobody volunteers them.

--- reply 3 (16 votes) ---
Emergency fund for textbooks and travel exists and is not means-tested beyond a short form.
```

**Chunk 3** — source: `thread_laptop_specs.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: How much laptop do I actually need for CS courses?

--- reply 1 (31 votes) ---
Less than the recommended spec page says. 16GB of RAM is the one number worth paying for; everything else you'll never notice.

--- reply 2 (18 votes) ---
Adding: the lab machines exist and are better than anything you'll buy. For the heavy assignments people just use those.

--- reply 3 (12 votes) ---
I did two years on an 8GB machine and it was fine until the last project, at which point it very much wasn't. 16 is the answer.
```

**Chunk 4** — source: `thread_office_hours_etiquette.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Is it weird to go to office hours with no specific question?

--- reply 1 (44 votes) ---
No, and this is the single most common thing first years get wrong. 'I'm following the lectures but I don't feel like I understand the shape of it' is a completely normal thing to say.

--- reply 2 (29 votes) ---
They're usually empty. You are doing the instructor a favour by turning up.

--- reply 3 (18 votes) ---
If it helps, treat it as a standing appointment. Go every week for a month and it stops feeling like a thing.
```

**Chunk 5** — source: `thread_professor_email.txt#0` — produced by: `chunker.py::split_documents`

```
THREAD: Do professors actually answer email?

--- reply 1 (21 votes) ---
Varies enormously. General rule I've found: if the syllabus states a response window, it's honoured. If it doesn't, assume 48 hours and don't panic before then.

--- reply 2 (33 votes) ---
Office hours are dramatically more effective than email for anything that takes more than two sentences to answer. They're also usually empty.

--- reply 3 (15 votes) ---
Empty office hours is the biggest unused resource here and I say that having wasted a year not going.
```

## Sample Answer

**Question:** How much RAM should a CS student's laptop have?

**Answer:**

```
A CS student's laptop should have 16GB of RAM. (Source: thread_laptop_specs.txt)

Sources retrieved: thread_first_gen.txt, thread_laptop_specs.txt,
thread_laundry_timing.txt, thread_pass_fail.txt, thread_printing.txt
```

**My relevance cutoff: 0.60**

The two groups separated cleanly with no overlap:

| Question | In corpus? | Best distance |
|---|---|---|
| How much RAM should a CS student's laptop have? | yes | 0.198 |
| When is laundry actually free in the dorms? | yes | 0.328 |
| When should I start looking for a summer internship? | yes | 0.262 |
| When is the deadline to declare pass/fail? | yes | 0.521 |
| What footwear do I need for a first winter here? | yes | 0.475 |
| What is the capital of Mongolia? | no | 0.948 |
| How do I change the oil in a diesel engine? | no | 0.930 |
| Who won the 1994 World Cup? | no | 0.952 |
| What is the recommended dosage of ibuprofen? | no | 0.828 |
| How do I write a for loop in Rust? | no | 0.871 |

The in-scope group ranged 0.198–0.521; and 0.828-0.952 for the out-of-scope group. 
The gap is 0.307, with no overlap. I would keep the original 0.6.

## How I Used AI

**1. Why crashed during indexing?**

My screen got frozen (crashed) while running `python app.py index` for the first time, I asked Claude and it told me some fix on the code level.
All of them failed, and suddenly found my storage and memory were both almost full, and the index needs lots of memory to move forward, the python process will crash if not enough space. 
I closed all unnecessary webpages and apps to free as much memory as possible and emptied my trash. Finally, it worked perfectly. 

**2. Write a new chunking strategy**

For milestone 3, I asked Claude to write a new chunking strategy,
and it suggested splitting each thread by individual reply with 100
characters of overlap. I read the threads and saw that reply 3 in
`thread_bike_commute.txt` says "Both true," which only makes sense in the
context of replies 1 and 2. I changed the strategy to keep each whole
thread as one chunk and dropped the overlap entirely, because `THREAD:`
is already a natural boundary. The result went from 26 chunks (with a
2-character fragment) to 23 complete threads.


<!-- ── Stretch features ─────────────────────────────────────────────────────
     Doing one? Say so here BEFORE you start. A feature this README never
     claims earns nothing.
     ───────────────────────────────────────────────────────────────────────── -->

---

# Unit 2

## Run Log — Before

Three runs per criterion, caching off. Produced by `run_eval.py::main`,
stored in `results/run_2026-09-28_0341_before.md`.

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 2. Every answer names a source | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks keep the thread they belong to | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Vote counts are preserved | 3 of 3 | 3/3 | 3/3 | 3/3 | MET |

Criteria 3 and 4 are deterministic, so the same number goes in all three
columns. Criteria 1, 2 and 5 depend on the generated answer and were
measured three times.

### Real output

**Criterion 1 — run 1 (laptop RAM):**

> Question: How much RAM should a CS student's laptop have?
> Best distance: 0.1978 (passed the gate)
>
> ```
> A CS student's laptop should have 16GB of RAM. (Source: thread_laptop_specs.txt)
> ```

**Criterion 1 — run 1 (pass/fail deadline):**

> Question: When is the deadline to declare pass/fail?
> Best distance: 0.5211 (passed the gate)
>
> ```
> You can declare the pass/fail option as late as week eight
> (*thread_pass_fail.txt* and *thread_first_year_regret.txt*).
> ```

**Criterion 2 — run 1 (internship timing):**

> Question: When should I start looking for a summer internship?
> Best distance: 0.2623 (passed the gate)
>
> ```
> You should start looking for a summer internship earlier than feels
> reasonable, as large employers close their applications in October and
> November for the following summer (*thread_internship_timing.txt*). If
> you miss the autumn window, smaller and local places hire in February
> and March (*thread_internship_timing.txt*).
> ```

**Criterion 3 — deterministic, one pass:**

| Out-of-scope question | Best distance | Gate |
|---|---|---|
| What is the capital of Mongolia? | 0.948 | refused |
| How do I change the oil in a diesel engine? | 0.930 | refused |
| Who won the 1994 World Cup? | 0.952 | refused |
| What is the recommended dosage of ibuprofen? | 0.828 | refused |
| How do I write a for loop in Rust? | 0.871 | refused |

Refused 5 of 5.

**Criterion 4 — 5 sampled chunks:**

```
thread_changing_major.txt#0: starts with 'THREAD: How hard is it to change major in second year?'
thread_group_project.txt#0: starts with 'THREAD: How do you handle a group project where someone disa...'
thread_parking.txt#0: starts with 'THREAD: Worth getting a parking permit?'
thread_printing.txt#0: starts with 'THREAD: Is the printing quota enough?'
thread_commuting.txt#0: starts with 'THREAD: Commuting an hour each way — is it survivable?'
```

All 5 start with `THREAD:`.

**Criterion 5 — 3 sampled chunks:**

```
thread_winter_advice.txt#0: 3 replies with vote counts
   --- reply 1 (26 votes) ---
   --- reply 2 (31 votes) ---
   --- reply 3 (18 votes) ---
thread_laptop_specs.txt#0: 3 replies with vote counts
   --- reply 1 (31 votes) ---
   --- reply 2 (18 votes) ---
   --- reply 3 (12 votes) ---
thread_laundry_timing.txt#0: 3 replies with vote counts
   --- reply 1 (27 votes) ---
   --- reply 2 (8 votes) ---
   --- reply 3 (16 votes) ---
```

All 3 preserve vote counts.

## Verdicts

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 | Retrieved chunk contains the answer | MET | 5/5 in all three runs. Target was 4 of 5. |
| 2 | Every answer names a source | MET | 15/15 answers named a thread file. Target was 5 of 5. |
| 3 | Gate stops out-of-corpus questions | MET | 5/5 refused. Target was 4 of 5. |
| 4 | Chunks keep the thread they belong to | MET | 5/5 sampled chunks start with `THREAD:`. Target was 5 of 5. |
| 5 | Vote counts are preserved | MET | 3/3 sampled chunks keep `(N votes)`. Target was 3 of 3. |

**On targets that were too safe:**

All five criteria were MET on the first try, so my targets were safe
rather than informative. Two are worth tightening:

- **Criterion 1** (target 4 of 5, actual 5/5). A 4-of-5 target cannot
  distinguish a working system from a mostly-working one. I would
  tighten it to 5 of 5.
- **Criterion 3** (target 4 of 5, actual 5/5, deterministic). The gap
  between my worst in-scope distance (0.521) and best out-of-scope
  distance (0.828) is 0.307 wide with no overlap. 5 of 5 is realistic.

Criteria 2, 4 and 5 were already at their strictest values and passed.

## Diagnoses

Missed nothing. All 5 were MET. No criterion failed, so there is no pipeline stage to diagnose. The
finding is that the targets themselves were set too low — which is what
the brief asks me to say when nothing misses. The Criteria 1
and 3 are worth tighting from 4 of 5 to 5 of 5.

## The Improvement

**What I changed:**

I lowered the relevance cutoff in `config.py` from 0.60 to 0.50. Nothing
else changed — same chunker, same embedding model, same top-k.

**Why I picked it:**

My before run showed the narrowest margin in the whole test was the
pass/fail question at 0.521, only 0.079 below the 0.60 cutoff. I wanted
to find where the cutoff actually starts to bite — the lowest value that
still lets every in-scope question through.

### Run Log — After

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 2. Every answer names a source | 5 of 5 | 4/5 | 4/5 | 4/5 | MET |
| 3. Gate stops out-of-corpus questions | 4 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 4. Chunks keep the thread they belong to | 5 of 5 | 5/5 | 5/5 | 5/5 | MET |
| 5. Vote counts are preserved | 3 of 3 | 3/3 | 3/3 | 3/3 | MET |

**Did it help?**

No — and it caused a measurable regression. At cutoff 0.50, the
pass/fail question (best distance 0.5211) is refused by the gate. That
drops:

- **Criterion 1** from 5/5 to 4/5. Still MET against its 4-of-5 target,
  but only just.
- **Criterion 2** from 5/5 to 4/5. **MISSED** against its 5-of-5 target,
  because the refused answer names no source.

The out-of-scope questions are unchanged — 5 of 5 refused, because
their lowest distance (0.828) is far above either cutoff.

The honest reading: 0.50 is too strict for this corpus. It costs a
real in-scope question and buys nothing on the out-of-scope side, where
there was already plenty of margin. 0.55 or 0.60 is the right place;
0.50 is worse than both.

## What's Still Broken

One criterion is still missed after the improvement: **criterion 2**
(every answer names a source, target 5 of 5). At cutoff 0.50, the
pass/fail question is refused by the gate, so only 4 of 5 answers name
a source. 4 of 5 is below the 5-of-5 target.

**Diagnosis.** The pipeline stage is **generation**, and the mechanism
is that no generation happened at all. The gate refused the question
before the model was called, so there is no answer and no source line
to check. This is the gate doing exactly what it was told to do at
cutoff 0.50 — the failure is not in the gate, it is in the cutoff I
chose for the improvement.

**Would I fix it?** Yes, by raising the cutoff back. The pass/fail
question has a best distance of 0.5211 and the next-closest in-scope
question is the winter footwear one at 0.4753. The cutoff needs to sit
between those two numbers to keep every in-scope question through:
anything above 0.5211 works, anything at or below it refuses pass/fail.
0.55 and 0.60 both satisfy that, and 0.55 gives less margin to the
out-of-scope side without buying anything. I would revert to 0.60.

**Why I stopped where I did.** The improvement was deliberately chosen
as a threshold probe, and the probe answered its question: 0.50 is the
first value that costs an in-scope question. Reverting it in this unit
would mean making a *second* change, which the brief forbids — the unit
allows one change, and mine is the threshold. The revert belongs in the
next unit, or as a described next step here.

## What I'd Do Differently

Knowing what I know now, I would rewrite two of my five criteria and
keep the other three.

**Criterion 1 — raise from 4 of 5 to 5 of 5.**

I set 4 of 5 to leave room for one hard question. Both the before run
and the after run at 0.60 used none of that room: the before was 5/5
and the after at 0.50 landed on exactly 4/5, which the 4-of-5 target
still accepts. A target the system clears (or exactly hits) under two
very different cutoffs is measuring whether the system is catastrophically
broken, not whether it works. 5 of 5 would at least tell me when one
question starts slipping.

**Criterion 3 — raise from 4 of 5 to 5 of 5.**

Same problem, weaker version. The gate is deterministic, and the gap
between my worst in-scope distance (0.521) and best out-of-scope
distance (0.828) is 0.307 wide with no overlap. There is no realistic
setting inside that gap at which the gate refuses 4 of 5 but not 5 of 5.
4 of 5 was safe; 5 of 5 would be honest.

**Criterion 2 — keep at 5 of 5, but add the qualifier.**

The interesting finding from this unit is that criterion 2 is not a
statement about the generator — it is a joint statement about the gate
and the generator. At cutoff 0.50 the gate refused a question, and the
refusal names no source, so criterion 2 failed without the model ever
being called. Next time I would write it as: "Every answer the system
produces, including refusals, names a source document — refusals may
cite the closest source they considered, or explicitly say none was
close enough." That would measure what I actually care about, which is
that no output leaves the system unattributed.

**Criteria 4 and 5 — keep as written.**

Both passed at every cutoff I tried, and both describe structural
properties of the chunker rather than properties of the answer. They
are the two criteria I would not change.

**The improvement I would make next.**

Not another threshold probe. The threshold result is already clear:
0.50 is too strict, 0.55 and 0.60 are equivalent on this corpus. The
next improvement I would make is one the earlier city_guides unit
already suggested — a hybrid retrieval variant that can promote a chunk
from outside the semantic top-20, because my current BM25 re-ranker can
only reorder what semantic retrieval already returned. That is the one
place where the retrieval stage, rather than the gate, could still move
the numbers.
