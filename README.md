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

<!-- These sections get ADDED to what's already above. Don't delete or rewrite
     unit 1 — the point is that someone can see what you said before you knew
     how it went. -->

## Run Log — Before

<!-- Your five criteria, three runs each. `python run_eval.py --label before`
     runs the questions, puts the OUT_OF_SCOPE ones through the gate, and
     writes it all into results/ for you. Targets come from criteria.md; the
     verdict column is your call.

     Criterion 3 is measured in one deterministic pass rather than three, so
     the same number goes in all three run columns. That's correct, not lazy.

     Milestone 1. -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

<!-- Underneath, paste the REAL output for each criterion from one of your
     runs — the actual text your system produced, not a description of it.
     Name the file and function that produced it. -->

## Verdicts

<!-- MET or MISSED for each of the five, against the target you wrote last
     unit — not a new one. Plus a sentence on how you decided. That sentence
     matters most where it was close.

     If your target said 4 of 5 and your runs came out 4, 3, 4, that's a MISS.
     The target has to hold, not show up occasionally.

     Milestone 2. -->

| # | Criterion | Verdict | How I decided |
|---|---|---|---|
| 1 |  |  |  |
| 2 |  |  |  |
| 3 |  |  |  |
| 4 |  |  |  |
| 5 |  |  |  |

## Diagnoses

<!-- For each miss: which stage caused it, and how. The stage alone isn't
     enough — you need the mechanism.

     Not a diagnosis: "Question 3 didn't work."
     A diagnosis:     "Question 3 asks about laundry costs. The answer is in
                       one sentence that got split across two chunks, so
                       neither chunk on its own contains it."

     The five stages: loading → chunking → embedding → retrieval → generation.

     Look for a pattern. If three misses all ask about numbers, that's one
     problem, not three.

     Missed nothing? Say so, then say honestly whether your targets were set
     low, and which one you'd tighten and to what.

     Milestone 3. -->

## The Improvement

**What I changed:**

**Why I picked it:**

<!-- Connect it to a specific diagnosis above in one sentence. If you can't,
     you picked a fix because it sounded impressive. -->

### Run Log — After

<!-- Same format, same five criteria, three runs each.
     `python run_eval.py --label after` -->

| Criterion | Target | Run 1 | Run 2 | Run 3 | Verdict |
|---|---|---|---|---|---|
| 1. Retrieved chunk contains the answer | 4 of 5 |  |  |  |  |
| 2. Every answer names a source | 5 of 5 |  |  |  |  |
| 3. Gate stops out-of-corpus questions | 4 of 5 |  |  |  |  |
| 4. | | | | | |
| 5. | | | | | |

**Did it help?**

<!-- Say plainly whether it did, and how you know. If it made things worse,
     say that — a change that backfired, honestly reported, earns full credit
     and is more interesting than one that worked. What matters is that you can
     tell.

     Milestone 4. -->

## What's Still Broken

<!-- For each criterion still missed after your fix: what you'd do about it,
     and why you stopped where you did.

     "I ran out of time" is fine if it's true. Pretending nothing is left is
     not.

     Milestone 5. -->

## What I'd Do Differently

<!-- Knowing what you know now — which of your five criteria would you write
     differently, and why?

     Milestone 5. -->
