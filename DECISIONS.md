# Decisions

# Week 1: the stack, the first call, and what it costs

Copy this into your `DECISIONS.md` and fill it in. Keep the headings. In
week 13 this becomes a section of your project report that you do not have
to write.

---

## Week 1

**Run conditions.** Everything below was produced on:

- machine: [Apple, M4, 16GB]
- model: [the model name, exactly as `ollama list` prints it]
  nomic-embed-text:latest 0a109f422b47 274 MB 13 days ago  
  qwen3:4b-instruct 0edcdef34593 2.5 GB 13 days ago
- served by: Ollama, one request at a time, locally
- date: [2026-09-29]

Every number in this file is meaningless without those four lines, so they
are stated once here and referred to rather than repeated.

### 1. Machine and model set

I am running the [required / required plus optional] model set.
[If you could not run the optional models, say so and say what you will do
before week 9. This is a constraint on your project, not a failure, and
naming it now is worth more than discovering it in week 9.]

REQUIRED ONLY - Since I don't have much storage

### 2. The first call

| | |
| finish reason |length|
| prompt tokens |24|
| completion tokens |2|
| elapsed |7.698328540995135|

One sentence on the finish reason: what my program would do differently if
it came back as a truncation rather than a normal stop.

Finish reason: stop means the model finished normally; length means it was cut off by the token limit.
Tokens: Prompt tokens depend on what I send. Completion tokens are limited by max_tokens=200.
Elapsed time: It is the time I wait for the model to generate the response.

[...]

### 3. Variance

My output:
(.venv) parsavafaei@MacBook-Air starter % python 02_variance.py --replay --full

Recording: qwen3:4b-instruct on Apple M4, 16 GB, Ollama, one request at a time, 2026-08-10

## cell distinct chars median s

closed_short|t00 1/12 10 0.17
closed_short|t10 1/12 10 0.17
open_short|t00 1/12 194 1.07
open_short|t10 5/12 198 1.07
open_list|t00 1/12 189 1.10
open_list|t10 11/12 178 1.14
open_reasoning|t00 1/12 1003 5.41
open_reasoning|t10 12/12 973 5.41
(.venv) parsavafaei@MacBook-Air starter % python 02_variance.py --replay
Recording: qwen3:4b-instruct on Apple M4, 16 GB, Ollama, one request at a time, 2026-08-10

## cell distinct chars median s

closed_short|t00 1/12 10 0.17
closed_short|t10 1/12 10 0.17
open_list|t00 1/12 189 1.10
open_list|t10 11/12 178 1.14
(.venv) parsavafaei@MacBook-Air starter % python 02_variance.py  
cell distinct chars median s

---

closed_short|t00 1/6 10 0.10
closed_short|t10 1/6 10 0.10
open_list|t00 1/6 189 1.06
open_list|t10 6/6 177 1.08

a. At temperature 0, how many distinct answers did you get? Does
your machine agree with the recording?

Yes, at temperature 0, I got 1 distinct answer for both open_list and closed_short. For the recording being 1/12, and for my machine ti being 1/6.

b. At temperature 1.0, one of the two cells still returns a single
distinct answer. Which one, and why that one? The answer is not
"the temperature did not work".

yes, on temperature 1.0, the closed_short still provides a distinct answer.
The reason is because closed_short asks: "What is the capital of Luxembourg? Answer in one word.
This is a closed question with only one correct answer "Luxembourg".

c. A unit test asserting exact string equality would pass on some
of these cells and fail on others. Name which, and say what
that tells you about testing this system.

An exact string equality test would pass on cells with 1 distinct answer and fail on 6/6 distinct answers.
You can't test LLMs by exact string matching, because open ended prompts give different but valid answers.

| cell | distinct (recording) | distinct (mine) | median latency |
| closed_short, t=0.0 | 1/12 | 1/6 | 0.10 |
| closed_short, t=1.0 | 1/12 | 1/6 | 0.10 |
| open_list, t=0.0 | 1/12 | 1/6 | 1.06 |
| open_list, t=1.0 | 11/12 | 6/6 | 1.08 |

Which cell still returns a single answer at temperature 1.0, and why that
one:

closed_short. reason above.

Which cells a test asserting exact string equality would pass on, and what
that tells me about testing this system:

answer above.

**The sentence that carries into week 10.** [One sentence about when you can
and cannot rely on repeating an output. Week 10 will ask you to find this
again. It should not say "the model is random", because your own table shows
otherwise in most cells.]

### 4. The cold start

- cold call: [1.04] s
- warm call: [0.10] s
- ratio: [10.3x]

What this implies for a system that uses more than one model, and what I
will do about it:

[Switching between models triggers cold starts (~1s vs ~0.1s warm). Batch calls by model to minimize swaps, keeping frequently-used models resident in memory.]

### 5. Cost, estimated

A 200-case golden set, at the token cost of my long case:

| | one run | nightly for the semester |
| small tier | €0.0002 | €0.47 |
| large tier | €0.0125 | €34.88 |

Estimates against the price list dated [date in `project/prices.py`], not
measurements. Running locally, my actual monetary cost was zero.

Which tier I would run nightly, which I would run before a release, and why
not the same one for both:

[Small tier should be run nightly, as it is more cost effective and also because it is ran repeatedly, and the large tier should be used before a release due to it's higher percision.]

### Deferred

[Due to time constraint and trying to understand how everything works, and my low speed in completion, this lab is completed later than the lab day, but had been started on the lab day itself.]
