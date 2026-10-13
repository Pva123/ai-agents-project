# Week 2: a structured-output extractor, measured

Copy this into your `DECISIONS.md` and fill it in.

---

## Week 2

**Run conditions.** model: [qwen3:4b-instruct] | temperature: 0.0 | prompt version: [week02-zero-shot-v1] |
served locally | date: [2026-10-13] | scored on: [live runs of my own prompts (the recording was used only to validate the scorer)]

### 1. The output contract

The conventions I chose, and why:

- due_date, when the message states no date: [null (JSON null / Python None, never "" or "null"). When a date is stated: ISO YYYY-MM-DD, which is unambiguous across English, French and German]
- due_date, when the message states only a relative expression: [null, because a relative expression is not a calendar date and resolving it means guessing today's date, which is the invention the model is prone to]
- quote, and what "verbatim" means in my scorer: [plain substring test, `quote in document_text`: no lowercasing, no punctuation or whitespace normalisation, an empty quote fails]
- what my scorer does with a record that failed validation: [counts it as wrong on all 4 fields, increments invalid counter]

It matters because a skipped record would silently raise the score as the model gets worse; here invalid was 0 in every run, so no record was skipped or hidden.

A scorer that skips the records it could not parse reports a number that improves as the model gets worse, hiding real failures.

### 2. Zero-shot, per field

| field | correct | of |
| category | 8 | 10 |
| urgency | 8 | 10 |
| due_date | 6 | 10 |
| quote | 10 | 10 |
| invalid records | 0 | 10 |

My prediction, written before block 3: examples will help most on [due_date]
because [the model invents plausible dates when a message states none, and an example showing null demonstrates the convention instead of only describing it].

### 3. Few-shot

Examples chosen, and the job each one does:

| example | why it is in the block | field it should move |
| EX-01 (en, badge reader, "not blocking me") | A message with no deadline, so null is demonstrated; also the access/hardware boundary and why urgency is standard | due_date, category |
| EX-02 (fr, lift stuck with a person inside) | A non-English example so English does not become the implied norm; a clear facilities + urgent case with a null date | category, urgency |
| EX-03 (de, laptop charger, "bis zum 20/09/2026") | The positive date case (DD/MM/YYYY becomes ISO), so the model does not over-learn "always null" | due_date |

| field | zero-shot | few-shot | move |
| category | 8/10 | 7/10 | -1 |
| urgency | 8/10 | 10/10 | +2 |
| due_date | 6/10 | 7/10 | +1 |
| quote | 10/10 | 10/10 | 0 |

### 4. What got worse

Category got worse, 8/10 to 7/10. REQ-04 was correct zero-shot and is wrong few-shot (got access, expected facilities), and no category error was fixed to offset it: REQ-08 (access instead of hardware) and REQ-09 (facilities instead of other) are unchanged. Diagnosis: REQ-04 is about the main entrance door of the Bierger-Center not locking, and EX-01 is about a badge reader at the side entrance, labelled access. Both are about an entrance door, so EX-01's access label most likely pulled REQ-04 towards access. This rests on comparing the two texts, not on removing EX-01 and re-running, so it is a likely cause, not a proven one.

I read the failure lines for both variants rather than the counts. Two urgency errors disappeared (REQ-02 and REQ-10 are now standard) and one date error disappeared (REQ-08). One error changed shape rather than being fixed: REQ-01 due_date went from 2023-10-05 to 2026-09-20, which is exactly the date in my EX-03, so the model copied the date from my example. REQ-05 and REQ-10 still get 2023-12-31, unchanged. Reading the texts, every date the zero-shot run invented came from a message that states only a relative expression, which the gold says must be null: REQ-01 ("tomorrow", "today"), REQ-05 and REQ-10 ("before the end of the month") and REQ-08 ("ce matin"). The same expression produced the same date each time. None of my three examples contains a phrase like "before the end of the month", which is the case that fails; EX-04 in the pool ("next week") does. Quote did not get worse (10/10 in both).

My prediction was partly wrong: I expected due_date to help most and urgency to stay flat. Urgency moved most (+2), due_date moved by one, and category moved the wrong way. I had assumed urgency was near ceiling from the reference recording, but my own zero-shot was 8/10.

### 5. What the examples cost

- extra input tokens per call: [242]
- per thousand calls: [242,000]
- estimated euros per thousand calls on the small tier: [0.242 million input tokens x 0.20 EUR per million (small tier) = 0.0484 EUR, about 0.05 EUR], against the
  price list dated [2026-08-10]. Estimate, not a measurement.

### 6. Ship it or not

Not the few-shot block as it stands, and not the role variant. Evidence: few-shot gained 2 fields net on ten records (32/40 to 34/40), all from urgency, at 242 extra input tokens per call (242,000 per thousand calls). The problem it was meant to fix, invented dates, only went from 4 failures to 3, and one of those is now copied from my own example (REQ-01); category also regressed (REQ-04). The role sentence bought nothing (quote 10/10 to 9/10, other fields unchanged) and cost about 10 tokens per call. On ten records one document moves a field by 10 points, so none of this is conclusive. At temperature 0 the results reproduced on these ten documents, but that says nothing yet about other messages.

If I kept anything, I would keep the urgency calibration from EX-01 and EX-02, replace EX-03 (its date leaked) with, or add, EX-04 from the pool (a relative expression, "next week", that maps to null), and re-measure; if cost matters, I would test a single example.

What would change my mind: a larger gold set (50+ cases) with per-language slices, repeated runs, the same comparison on qwen2.5:7b, and a re-run with revised examples showing that REQ-01 stops copying the date while the +2 on urgency holds.

### Sensitivity variant

Variant assigned: [role]. What I changed: [prepended "You are a senior service desk analyst." to my few-shot prompt, nothing else]. What moved: [quote 10/10 to 9/10 (-1); category, urgency and due_date unchanged; about 10 extra tokens per call (5013 to 5113 total over ten documents)].

By language, field errors went from 4 to 5 in English (5 documents), 2 to 2 in French (3 documents) and 0 to 0 in German (2 documents), so the extra error is in an English document. `03_sensitivity.py` does not print failure lines, so I have not identified which document it was or whether the model paraphrased the quote, and one record out of ten cannot separate a real effect from noise. What it shows: a sentence no reviewer would flag moved a measured number in the wrong direction and bought no gain, so on a closed-label schema task a persona is not worth its tokens. The baseline in this run matched my few-shot result exactly (7/10, 10/10, 7/10, 10/10), so the comparison is like for like.

### The gold set

Ten cases written to `artifacts/goldset.json`, tagged by language.

One thing my scorer cannot currently detect:

The quote check proves the model copied, not that the copied text is relevant: any verbatim substring passes, even a greeting, so it cannot tell whether the quote supports the urgency decision. It also counts an invented date, a wrong date and a malformed "" or "null" the same way, and cannot see that a wrong date was copied from an example (REQ-01) unless I read the failure line.

### Deferred

Testing the REQ-04 explanation by removing EX-01 and re-running; I only compared the two texts. Finding which document broke under the role variant, because `03_sensitivity.py` prints no failure lines. The other three sensitivity variants (reordered, no_delimiter, english_only), which belong to other groups.
