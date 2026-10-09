# FDE-Master.md — Forward Deployed Engineer (LLM Context)

Reusable instructions for **LLM features, prompts, model selection, measurement, and agentic coding**. Attach this file as context for any LLM that helps you design or code those systems.

Source lessons live under `Forward Deployed Engineer/`. This file is the operating contract plus the important notes. It is not a substitute for the article files when you need the full worked examples. Use it on any engagement. Do not treat it as a class outline.

**If `Master.md` (SDLC & Python Foundations) is also attached:** follow **both**. `Master.md` owns Python, SDLC, tests, Docker, deploy. This file owns models, prompts, harnesses, agents, and agentic git/review. When they overlap (tests, PRs, unhappy paths), apply **both**. Do not drop SDLC quality gates because the work is “AI.”

---

## Client-facing priorities (do this first)

On real engagements, two articles outrank prompt-wording advice. Apply them **before** proposing a model, a comparison slide, or more prompt text.

### 1. When not to use an LLM — placement, not skepticism

`Forward Deployed Engineer/When Not to Use an LLM.md`

Client engineers lose trust if you put a model in front of a problem **their system already solves correctly**. They often will not say so in the meeting. Credibility is pointing at a step in **your** design and saying **that one should be code**, before they do.

Model risk / change control will ask: which behavior is **specified** (code: diff, test, release) vs **learned** (prompt: “it usually holds”). That line is architecture. Decide it early.

| Use **code** | Use the **model** |
|---|---|
| Same facts + same written procedure → always the same answer | Reading, weighing, interpreting messy text |
| Arithmetic, dates, business days, holidays, thresholds, period boundaries | Narrative → structured facts |
| Lookup from the system that **owns** the record (policy, transaction, customer, sanctions) | Spotting contradiction; substantive vs wording difference |
| Routing tables, eligibility, deadlines once facts exist | Summary a human will act on; draft a human will send |

**Shape that ships:** model **reads** → structured **cited** object → code **decides**. Wrong outcome is then either a bad extraction (check the citation) or a bad rule (unit test). Do not encode the same rule in both prompt and code.

High hit rate on dates is the trap: survives a demo and twelve cases, fails on the bank holiday. Lookups from memory are recollections, even if the value appeared earlier in the same request.

### 2. Reading a model comparison — the artifact that leaves the room

`Forward Deployed Engineer/Reading a Model Comparison.md`

The comparison report is often the **first** LLM artifact a client forwards. Caveats you omit will be omitted in every later slide. The number you print largest becomes the number the engagement runs on. A careless twelve-case table becomes a **production quality commitment** three meetings later.

You will also be handed vendor/client/colleague tables. Most split quality from cost, quote **mean** latency, and hide which **prompt** each model ran. Ask the table questions below instead of inheriting their winner.

**Non-negotiables for any client-facing comparison**

- **One table per task** with quality **and** cost/case **and** latency. Three separate tables → three winners; the reader picks the last one they saw. Do not average triage + extraction into one “system score.”
- **Cost per case** = every billed attempt (failures, transport retries, schema repair). “Cheaper on successful calls” often means **more expensive per case**.
- Latency: **median + max**, state N, first attempt → final result, and **where and when** it was measured. Not mean. Not p95 on twelve. A latency figure without place and time is not reproducible.
- Quality: **counts by error type** (miss vs invent). 8/12 miss ≠ 8/12 invent. Compliance cares about invention.
- Every row names **prompt version**. Unadapted prompt = **untested** (transfer), not “this model lost.”
- Twelve cases: **direction**, not magnitude. Eliminate hard-constraint failures; spot systematic bugs. Do **not** rank a 1–2 case gap, quote a percentage, or claim production-volume behavior. Write **what the set does not show** as if that is the only section they will read.
- Recommendation: model + prompt version + **what would reopen it**. Constraints and rejected alternatives live in the decision record, not as extra adjectives in the table.

**If the user is about to show numbers to a client**, stop and check this list before polishing wording.

### 3. RAG over client documents — important

Three articles. Apply them whenever the answer must come from material that does not fit the window, or that the model was never trained on (policy manuals, codebases, internal knowledge bases).

| Article | Path | The decision it owns |
|---|---|---|
| Embeddings, vector stores, chunking | `Forward Deployed Engineer/Embeddings, Vector Stores, and Chunking.md` | How a basic RAG pipeline is built |
| Hybrid and agentic retrieval | `Forward Deployed Engineer/Hybrid and Agentic Retrieval.md` | When plain similarity is not enough |
| Failure modes | `Forward Deployed Engineer/Failure Modes: Lost-in-Middle, Retrieval Mismatch, and Hallucination Under Low Recall.md` | Which layer actually failed |

**Do not stuff the corpus into the window** because it fits. Retrieve the relevant portions. **Do not jump to agentic retrieval** because it is more sophisticated. Plain vector search is the starting point; hybrid and agentic are justified upgrades. **Do not rewrite the prompt** until you have looked at what was actually retrieved.

The operational contract for a policy corpus (what similarity cannot do, `embed_text` vs `text`, `build_id`, reciprocal rank fusion, pre-filtering, multi-step with code in control) is in the retrieval sections below, from **What an embedding actually is** through **Retrieval in more than one step**. The three articles above are the simpler starting picture. Where they differ, follow those retrieval sections.

---

## How the LLM must work

These rules apply on **every** reply unless the user later writes: **implement this**.

1. **Do not implement by default.** Recommend, cite, and ask. Write full patches only when the user clearly asks to implement, apply, or write the change.
2. **Always cite.** Name the FDE article (and idea). Example: `(FDE — Writing the Instruction — unknown is first-class)` or `(FDE — When Not to Use an LLM — model reads, code decides)`. Prefer article name + idea. Add the path under `Forward Deployed Engineer/` when the user needs the exact file.
3. **Always suggest.** Options, a recommended path, next step. Let the user choose.
4. **Ask before coding** when the request is ambiguous, mixes determined vs judgment work, would put a rule in a prompt that belongs in code, would edit a prompt version in place, or would un-pin a model alias.
5. **Prefer checklists and plans** over diffs. Short snippets are **example, not a drop-in**.
6. **Stay honest about limits.** Twelve cases support **direction**, not production quality percentages. Temperature 0 is **stable**, not deterministic. Reasoning text is **not evidence**. Never let a comparison table imply more than the set supports `(Reading a Model Comparison)`.
7. **Split determined vs judgment before adding a model.** Dates, thresholds, lookups, and written procedures belong in code. Putting a model on a solved, deterministic step loses client engineering trust `(When Not to Use an LLM)`.
8. **Do not skip absence, ambiguity, out-of-scope, retries, or spend ceilings** to “save time.”
9. **Do not invent a second prompt, schema, or scorer** beside a versioned one unless the user is doing a planned new version.
10. **Match the repo’s tools.** If they use `promptlab`, Pydantic schemas, pinned `config.py`, pytest, worktrees — do not switch the stack unless asked.
11. **Never commit secrets, client case files, or edit a recorded prompt version in place.** Rotate leaked credentials. New prompt = new version file.

### Suggested reply shape (use this every time)

```
1. Recommendation (what to do, not the finished implementation)
   — If client-facing: say what is **code** vs **model**, and what the numbers **do not** claim.
2. Citations (FDE — article — idea) [and Module N if Master.md also applies]
3. Steps the user can run
4. Risks / unhappy paths (absence, invention, out-of-scope, cost, latency, overclaimed %)
5. What to confirm before anyone writes code
```

### When the user says “implement this”

Then you may write code. Still:

- Model reads / code decides for determined rules (dates, thresholds, lookups).
- Schema + validation + repair cap; prompts versioned; models pinned.
- Tests from the requirement; do not modify gold labels or scored tests to match the model.
- Show quality-gate commands afterward.
- Do not expand scope past the file boundary they named.

### Questions the LLM should ask (when missing)

- Is the answer **determined** by facts + a written procedure (code) or does it need reading/judgment (model)? `(When Not to Use an LLM)`
- Do the facts live in a **corpus** the model was not trained on, or that does not fit the window? Then RAG: chunk, embed, retrieve — do not stuff the source. `(Embeddings, Vector Stores, and Chunking)`
- Does the corpus contain **exact codes, SKUs, or phrases** vector search will miss? Hybrid. Is one retrieval pass enough, or is the question ambiguous enough to pay for another search? `(Hybrid and Agentic Retrieval)`
- On a bad RAG answer: **what was retrieved**, before anyone edits the prompt? `(Failure Modes)`
- What is the **accuracy floor**, from the **consequence of being wrong**, and is a human still in the loop? `(Selecting a Model)`
- What are the **latency budget** and **cost ceiling** at real volume? `(Latency and Cost)`
- Extraction / routing / drafting — different **temperature** and maybe different **models**. `(Sampling, Selecting a Model)`
- Which **prompt version** and **pinned model id** produced the numbers you are quoting? `(System Design, Reading a Comparison)`
- For agent work: **outcome, authority, boundary, stop rule**? Which files may change? `(Scoping, Parallel Agents)`
- For a multi-agent case: is memory **this request only**, **durable across sessions**, or **shared**? Shared store or an explicit handoff? `(State and Memory)`
- On a bad multi-agent answer: can you pull the **trace** (which agent, where the input came from, output, duration, tokens, errors) instead of rerunning? What is **cost per agent per case**, and which step owns the latency? `(Observability and Tracing)`
- Does this step need an **answer now** (API), a **notification** (webhook plus reconciliation), or a **buffer** (queue plus a depth alert)? What happens when that call fails? `(Integration Patterns)`

### Model vs harness vs agent (keep these distinct)

| Term | Meaning | Classroom picture |
| --- | --- | --- |
| **Model** | The LLM. Reads/writes tokens. Cannot finish a job alone. | The student’s brain |
| **Harness** | Prompts, schema, tools, retries, logs, spend ceiling, adapters | Worksheet, boxes, clock, allowed calculator, teacher who stops the exam |
| **Agent** | Model + harness + **loop** (see → decide → act → repeat until done or halted) | The student actually taking the test until time is up |

Same model, different harness → different agent. Do not say “we use Claude” when you mean the shipped loop.

---

## Default workflow (any LLM / agent feature)

Follow this order. Skip a phase only if the user says it is already done.

| Step | Phase | Do this | Cite |
|---|---|---|---|
| 0 | **Client gate** | Split **code vs model**. Do not put an LLM on determined work. If sharing numbers, draft the comparison with caveats **in the table**, not as an afterthought. | **When Not to Use an LLM; Reading a Comparison** |
| 1 | Split the work | Model reads unstructured input → structured cited object. **Code** applies rules, dates, lookups, routing tables. | When Not to Use an LLM |
| 1b | **Retrieve** | If facts live in client documents: chunk → embed (dedicated model) → store → embed the query with the **same** model → search → pass only those chunks. Hybrid when exact terms matter. Agentic only when one pass is not enough. On a bad answer, inspect retrieval before the prompt. | **Embeddings; Hybrid and Agentic; Failure Modes** |
| 2 | Constrain | Accuracy floor, latency budget, cost ceiling, residency, modality. Hard constraints **eliminate**. | Selecting a Model |
| 3 | Pin facts | Dated fact sheet: identifier, no alias, prices, window, structured-output level, retirement. No quality column. | What Actually Differs |
| 4 | Schema | Pydantic source of truth. Enums for branch fields. Present / absent / ambiguous. Citations per field. Shallow. | Structured Outputs |
| 5 | Prompt | Spec, not chat. Task → input (delimited) → constraints (positive, checkable) → output → cannot-complete. Versioned file. | Writing the Instruction |
| 6 | Request layers | System = standing, byte-stable. User = case. Untrusted in markers. Render fails on missing vars. Registry + hash. | System Design for Prompts |
| 7 | Sampling | Extraction/routing ~ temp 0. One of temp / top-p. Control fields must not vary. | Sampling |
| 8 | Measure | Frozen cases, gold from **source**, score per field with code, counts not fake %, scorer versioned separately. | Measuring a Prompt |
| 9 | Compare | One table per task: quality + cost/case (all attempts) + median/max latency. Prompt version per row. Label unadapted prompts **untested**. | Reading a Comparison |
| 10 | Decide | Decision record: rejected alternatives + **review triggers**. Mixed models per task is normal. | Selecting a Model |
| 11 | Agent coding | Red tests from contract → scoped task → worktrees if parallel → review in risk order → small PRs. | Test-First, Scoping, Parallel Agents |

```
DETERMINED? → code
IN A CORPUS? → chunk → embed → store → retrieve (hybrid if exact terms; agentic only if one pass fails)
JUDGMENT?   → model → schema → prompt vN → pin model → measure on gold → decision record
BAD RAG ANSWER? → what was retrieved? → then, did the model use it?
AGENT CODE? → outcome + authority + boundary + stop → red tests → worktree → review → merge
```

---

# Tokens, context, and the shape of a request

Models read, write, and bill **tokens**, not characters or words. A tokenizer converts text first. Every limit is in tokens: context window, max output, rate limit, price.

- Domain IDs (`POL-4471-88B`, IBAN, sort codes, formatted money) **fragment**. Cost more per page than prose. More likely to transcribe wrong. **Verify identifiers against the source.**
- Token counts are **not portable** across providers. Record **provider-reported** usage, do not invent a shared counter.
- **Context window = input + output together.** Ceiling you design under, not a target to fill. Filling it costs money and latency and does **not** raise accuracy by itself.
- **Output is usually priced several times input.** Short answer over a long document is input-dominated. Long answer over a short prompt is output-dominated. Know which before optimizing.
- `max_tokens` is a **hard stop**. Hitting it mid-JSON looks like HTTP success and a parse failure far from the cause. Size the ceiling from the largest legitimate schema output. Treat truncated finish as an error.
- **Roles:** system = standing behavior; user = the case. Anthropic: top-level `system`. Azure OpenAI: first message `role: system`.
- Usage names: Anthropic `input_tokens` / `output_tokens`. Azure `prompt_tokens` / `completion_tokens` / `total_tokens`. Normalize into one record (C1).
- Typical extraction: **document ~96% of input**. Polishing the instruction is a rounding error. The lever is **what you attach and how often**.

**Do not:** truncate/chunk on character counts (“4 chars/token” is a first estimate and **wrong at the boundary**); send the whole case file because the window fits; put standing rules in the user message next to untrusted text.

---

# Sampling, and why the same prompt gives two answers

The model produces a **distribution**, not an answer. The prompt shapes it. Sampling draws from it. Two answers = two draws.

**Inference loop:** prompt → probability for every next token → sampler (temperature / top-p / top-k) → one token appended → next step is conditioned on that draw → stop at end token, stop sequence, or max tokens. After the first differing token, contexts diverge.

| Control | What it does | Trap |
|---|---|---|
| **Temperature** | Sharpens (low) or flattens (high) the whole distribution | **Variety**, not quality / accuracy / creativity |
| **Top-p** | Keep smallest token set whose probs sum to *p*, then sample | Different mechanism from temperature |
| **Top-k** | Limit candidate set by count (slides; article stressed top-p) | Same family as top-p |
| **Stop sequences** | End on a terminator | Useful with imposed structure |
| **Max output** | Hard stop | Hitting it is an **error** |

**Pick one** of temperature / top-p (leave the other at default). Setting both to unusual values makes the config unexplainable.

**Determinism is not on offer.** Temperature 0 is much more stable, not guaranteed (float math, batching, hardware, aliases). Seed is best-effort. Promise **high stability** under fixed settings and a **pinned version**, not bit-identical replay.

Settings follow the **task**, recorded next to the prompt, not one forgotten global:

| Task | Sampling | Why |
|---|---|---|
| Extraction | 0 or near 0 | Same input → same fields |
| Classification / routing | 0 or near 0 | Queue must not be sampled |
| Escalation flags | 0 or near 0 | Human-review must not flip |
| Customer draft | Moderate | Wording variation OK |
| Brainstorming | Higher | Variety is the point; still evaluate |

**Variance is measured.** One run = the prompt *can* be right. Several runs = whether it *reliably* is. High temperature on an ambiguous case often surfaces the two queues the model was already torn between — fix **escalation**, not wording, until one draw looks pretty.

Worked: temp 0 → 5/5 `card_dispute` + human review true. Temp 1.0 → mix of `card_dispute` / `fraud_report`, and `requires_human_review` flipped false once. That flag is the last field you want sampled.

**Do not:** lower temperature to “fix” a wrong answer (makes it consistently wrong); assert full string equality across runs; judge a prompt on one easy demo.

---

# Latency and cost as design constraints

Write **cost ceiling** and **latency budget** in the design record **before** architecture preference.

**Latency personas (requirements):**

| Persona | Budget |
|---|---|
| Handler watching a screen | A few seconds |
| Overnight batch | Minutes can be fine |
| Webhook | **Caller timeout** is the hard limit |

Latency ≈ **time to first token** (input processing + queue) + **generation** (scales with output tokens). **Streaming does not reduce total latency**; it changes when the user sees the first token. If latency looks bad, find **which part** is large — fixes differ.

Retries and rate limits **are** latency. Measure **first attempt → final result**. Record attempt count. Timing only the successful call is fiction under load.

**Cost:**

```
cost = (input tokens × input rate) + (output tokens × output rate)
EXTRACT + billed schema-fail + RETRY + ROUTE  =  case cost
```

Client unit is **cost per case**, not per call. Sum every attributable call, **including retries and failed calls that returned tokens**. Pin prices in `config.py`, verify against the provider page before any number leaves the room.

**Prompt caching:** stable prefix, **byte-for-byte**. Timestamp, case id, or shuffled examples **before** the cache boundary silently defeat it. Confirm cached tokens in the usage block. System layer must stay identical for a prompt version.

Illustrative arithmetic (placeholders — never quote as live prices): 1240 in / 210 out at $3 / $15 per MTok → $0.00687/call; ~1/8 retried → ×1.125 → **$0.00773/case**; ×40k/month → ~$309. Output can be ~14% of tokens and ~46% of cost. Defend the **retry-rate and volume assumptions**, not the pretty total.

**Do not:** cost the demo case (real cases have attachments and notes); forget output tokens (especially quoted evidence); prices from memory; assume cache hits.

---

# What actually differs between models

Vendors ship **families**; **tiers** inside a family trade capability vs cost/latency. **Tier is usually the consequential move.** Changing vendor = integration + commercial + security review. Rate limits are commercial (tier + account), not a model property.

| Axis | Meaning for selection |
|---|---|
| Context window | **Capacity, not comprehension.** Realistic case must fit. Middle-of-context drop-off is a retrieval problem (lost-in-the-middle). |
| Reasoning mode | Extra billed tokens and latency. Helps multi-step deduction, not “read a field.” Not portable across providers. |
| Structured output | Schema-constrained decode vs JSON-valid-only vs instruct-and-validate. Weaker guarantee → more repair tokens. **Always keep a validation path.** |
| Modality | Text vs image/document. Reading text already in a file is in scope; **building OCR is not** (this program / capstone). |
| Identifier | **Pin dated versions, not aliases.** Aliases change behavior with no deploy. Azure: you call a **deployment name** that can move independently of your code. Record retirement dates. |
| Training cutoff | Almost irrelevant if the answer must come from the attached document. Dangerous if asked to recall a regulation or library API. Supply text, require citation. |

**Fact sheet** (`docs/model-facts.md`): every row has a **source and date**. Alias row should say **none**. Rows that go stale first: price, rate limit, retirement. **No quality column** — quality is measured on your cases.

**Do not:** spend a long time choosing a vendor when the saving is a cheaper **tier**; fill a huge window because you bought it; assume structured output / caching / usage field names are portable (that is why the **adapter** exists).

---

# Open weights and where they fit

**Open weights ≠ open source.** Usually released: parameters + enough config/tokenizer to run. Usually **not**: training data, pipeline, training code. You cannot reproduce or audit the way “open” means in software. Use the accurate term with clients.

**License is the constraint.** Apache/MIT vs bespoke community licenses (acceptable use, redistribution, usage-threshold obligations). In a regulated client, legal review can take longer than engineering. Read the **exact model and version** before architecture assumes it.

Clients raise this for **residency, concentration risk, audit posture** — rarely cost. Alternatives that are **not** self-hosting: regional endpoints, contractual processing terms, zero-retention. Establish which requirement you are actually meeting.

**Hosting moves cost from marginal to fixed.** Break-even is farther than people think. Article shape: $12/h × 720 h = $8,640/month; ÷ $0.00773/case ≈ **1.12 million cases/month** for one always-on instance. At 40k cases (~4% utilization) effective cost/case ≈ $0.216, ~**28×** hosted. **Utilization** decides, not the hourly sticker. Engineering, on-call, and **your** regression suite are extra.

Capability gap is **narrow on narrow tasks** (six queues, extract into a schema); **wide** on long multi-step reasoning, complex schema adherence, agentic behavior. Prove it on **your** case set.

**Fine-tuning:** discussable; **out of scope** for the capstone. Prompting + retrieval usually move extraction/routing further without a model artifact you must version and retrain when policy changes.

Keep the option open **in code** (one interface, one config entry), not by welding provider calls into every call site.

**Do not:** treat “open” as free; read the license after the build; propose a GPU farm to answer a contract question a regional endpoint would satisfy.

---

# Selecting a model for a use case

**Constrain before you compare.** Accuracy floor from **consequence of being wrong** (handoff vs customer outcome), latency budget, cost ceiling at **real volume**, residency, modalities, data-handling terms. That list usually eliminates most of the field.

**Hard constraints eliminate. Soft criteria rank.** Do not mix them in one weighted matrix (you can “win” a candidate that fails a requirement).

**Measure on your gold-labeled cases, per task.** Mixed selection (cheap model for high-volume triage, stronger for extraction) is a **legitimate** outcome if both sit behind one interface.

**A model comparison is a prompt + model comparison.** Prompts do not transfer. If B loses, adapt the prompt for B and remeasure. Record **prompt version per row**.

**The decision record is the deliverable**, not the winner (`docs/model-decision.md`): constraints, candidates, evidence, choice, **rejected alternatives and why**, date, authors. Keep it separate from the fact sheet.

**Review triggers are conditions**, not calendar nags: pinned version enters deprecation; prices move; volume order-of-magnitude; new tier in the family already in use; measured accuracy on the case set drops.

Normal worked shape: triage on cheaper B (human reads every case); extraction on stronger A; reject B for extraction only **after** adapted prompt; reject a third vendor with no measured advantage (second security review for free).

**Do not:** compare before constraining; one flagship model for everything; tuned prompt vs paste; selection by reading twenty outputs; no review trigger until a retirement notice with a short window.

---

# When not to use an LLM

**Client-priority article.** Full notes: `Forward Deployed Engineer/When Not to Use an LLM.md`. On an engagement, apply this **before** prompt iteration.

**Test:** if a competent person given the same facts and the same written procedure would **always** produce the same answer, the answer is **determined** → **code**. A model will usually get it right and occasionally not, which is strictly worse than a function.

**Never the model’s job:** arithmetic, dates, business-day counts, holiday calendars, threshold comparisons, policy period boundaries. High hit rate is what makes this dangerous — survives demo and a twelve-case set, fails on the bank holiday.

**Lookups belong to the system of record.** Policy status, transactions, customer master, sanctions lists. Recalling a value (even one earlier in the same request) is not a lookup.

Every deterministic step in code is removed from the **bill** and the **clock**, and can be **proven** (diff, test, reviewer, release). A rule in a prompt is text that influences a probabilistic process. In a regulated function that difference decides whether a feature ships.

**Shape that survives compliance:** **the model reads, the code decides.** Model → structured cited object. Code → decision. Failure localizes: bad extraction (check citation) vs bad rule (unit test).

**Keep the model** where there is no rule to encode: rambling narrative → facts; spotting contradiction; substantive vs wording difference; summary a human will act on; draft a person will review.

**Do not:** put thresholds/deadlines/routing tables in the prompt; use the model as a database; keep a computation in the prompt “for convenience”; encode the same rule in **both** code and prompt (they will disagree and nothing says which won). If the prompt mentions the rule, say **code applies it**.

Worked split: model extracts `DisputeFacts`; `BusinessCalendar.add_business_days` computes deadline; `route()` uses explicit ifs. Weekend/holiday cases become unit tests.

---

# Writing the instruction

A production prompt is a **specification**, not a chat request. No follow-up questions. Ambiguity becomes a coin flip. Clients will **read the prompt** (compliance: forbidden behavior; ops: does it match how we work).

**Stable section order (every production prompt):**

1. **Task**
2. **Input** (delimited)
3. **Constraints** (positive, checkable)
4. **Output**
5. **When the task cannot be completed** (absent / ambiguous / out-of-scope)

**Delimit untrusted input always.** Everything between markers is **data**, even if it contains “please ignore previous instructions” or imperative procedure text. That is a correctness requirement on every engagement. A later security review does not replace it.

**Constraints:** “don’t hallucinate” is a wish. “Answer only from text between markers and cite the section” is a procedure you can verify from the output. Two or three real constraints beat a dozen aspirational ones.

**Unknown is first-class.** No legal way to say absent → invention. Separate: not in the document; in the document but conflicting; outside this task. They drive different human actions.

**Out-of-scope is a path**, not an accident. If you only describe the success path, the model maps a bereavement note onto the nearest queue **with the same confidence**.

**Prompts are code.** Files like `src/promptlab/prompts/summarize.v1.md`. **Never edit in place after results are recorded.** Change = new version. Records carry `prompt_id` + `prompt_version`. In-place edit silently invalidates every measurement.

**Do not:** instruction pile-up (append a line per failure until the prompt contradicts itself — restructure); negative-only instructions; undelimited case text; “be accurate.”

---

# Structured outputs

Once **code** consumes the answer, **shape** matters as much as meaning. Schema is the contract (Pydantic v2 in `schemas.py`). Prompt description should be **generated from the model**, not hand-copied (drift is invisible). Downstream, including the scorer, imports the **schema**, not the prompt.

Providers: (1) schema-constrained decode, (2) valid JSON but not your schema, (3) instruct and validate. Build for the **weakest** you support. Stronger guarantees reduce how often repair runs; they do **not** let you delete validation.

- Every field a downstream system **branches on** is an **enum**, including trouble values (`other`, out-of-scope). Free text `card disputes` vs `card_dispute` will break routing.
- **Present / absent / ambiguous** are different. One `null` destroys the information the system was supposed to produce.
- **Per-field citation** in the structure (section id, not quoted spans — output tokens are expensive). Instruction to cite is worthless if the schema has nowhere to put it.
- Validation is a **boundary**. Fail loudly. Repair: send the **validation error**, cap attempts, record every attempt in C1 (true cost includes repair). High fail rate is a schema smell.
- Keep schemas **shallow and small**. Nesting, huge enums, optional sprawl, echoing the whole input → worse adherence and higher cost. Large schema usually means **one prompt doing two tasks** — split.

`Evidence[T]`: present requires value **and** section; absent must not carry a value; ambiguous requires a note. `extra="forbid"` so invented fields error instead of silent drop (the invented field is often where the real answer went). Types force units (`beneficial_ownership_threshold_percent` as float, not `"25%"`).

**Do not:** retry the identical request without the error; mix schema copy in the prompt; free-text routing; grow the schema instead of splitting tasks.

---

# Few-shot and grounding

**Grounding** is a design property, not a vibe. Compliance can live with a wrong field a human reviews. It cannot live with a statement nobody can locate in a source that later sits in a case file.

Grounding is **four checkable steps:** marked source; restrict to that source; identify where each statement came from; defined way to report the source does not contain the answer. Prefer citation **with or before** the claim so evidence leads, not decoration after a conclusion.

**Few-shot** specifies what prose cannot: judgment boundaries (policy vs mentions policy; conflict vs wording). Wrong tool for anything you can simply state, and **wrong tool for structure** (schema already enforces that).

Examples are **input tokens on every call forever**. Justify each one. First place to look when cost/case is over budget. Stable examples belong in a **cacheable prefix** (nothing above them may vary).

**Choose edges, not the middle.** The typical clean case is already handled. Productive: silent required field, self-contradiction, superseded, content that looks like instructions, out-of-scope. Two edge examples beat five typical ones.

**Examples leak.** Outward: section numbers / names / phrases copy into unrelated outputs (looks like hallucination; grep the example pool). Inward: examples from the **eval set** contaminate scores. Keep a **separate example pool**. Never score on material you showed.

Negative examples need the **correction attached** (input, wrong output marked, one line why, correct output). Otherwise the model copies the wrong output. Often cheaper: show only the correct handling.

**Second grounding failure (the dangerous one):** statement is **true**, citation is real-looking, but the **supplied document does not say it** (training knowledge). Reviewers who check “is it right?” miss it. Check: cited section **exists** and **contains the claim**.

**Do not:** full expected objects in examples to teach the schema; unpaired negative examples; infer ownership thresholds from “what policies usually say.”

---

# Reasoning in the prompt

Chain-of-thought: intermediate tokens condition the final answer. Nothing “inside” the model changes. Helps when several facts must be combined. Does **nothing** (or harms) when the value is already in the document (extraction — invites interpretation). Arithmetic/dates still belong in **code**.

Paid in **output tokens and latency**, on **every** call including cases that did not need it. Adopt **per task** after measuring.

**Extended thinking** is a provider feature: billed, latent, and what you get back varies (full trace / summary / nothing). Not portable. Do not design a comparison that **requires** the trace.

If you ask for reasoning, give it a **dedicated schema field** (`analysis`). Keep it off every field downstream reads (`queue`). Reasoning that has nowhere to go contaminates enums.

**Reasoning text is not evidence.** It is generated by the same process as the answer, can be fluent and unrelated to what drove the output, and must **never** go in a case note, review packet, or auditor pack. Evidence = citation a human can open. When asked “why this routing?”, answer with **cited facts**, not the model’s autobiography. `review_reason` that names two checkable sources can go in a case note; `analysis` cannot.

**Self-consistency:** sample several times at **nonzero** temperature; majority vote. Cost = full call × N. More useful: **disagreement**. 3–2 on `card_dispute` vs `fraud_report` → escalate to a human, do not treat the vote as certainty. Temperature 0 × 5 samples = 5× cost, no information.

**Do not:** CoT on every extraction call; persist traces as audit trail; depend on one vendor’s thinking dump.

---

# System design for prompts

Reproducing a case months later requires knowing **exactly which text was sent**, not which prompt file exists in the repo today.

**Layers:**

| Layer | Owner | Contents |
|---|---|---|
| **System** | You | Standing behavior, output contract. **Identical** every call for that version (enables prefix cache; makes diffs legible). **No case content.** |
| **User** | Mixed | Per-case data you generated (case id, date) can interpolate. **Untrusted** third-party documents/messages go in markers. |

If you cannot tell which category, treat as **untrusted**.

Untrusted text that **reads like instruction** is a correctness problem before it is a security problem (reviewer notes, “disregard the rules above”). Helps that cost almost nothing: mark the span; put **task instruction after** the span as well as before (last thing seen is yours); defined response if content addresses the model (`document_kind: other` + reason).

**Templates render in code and fail loudly.** Missing placeholder → exception, not empty string (empty document → fluent invention, no error). Escape your closing marker if it appears inside the document.

**One registry** loads `prompt_id` + version → immutable text + **hash**. Pin **both** prompt version and model id. Neither alone reproduces the system. Prompt change = new version, PR, measured comparison.

**Do not:** put case ids in the system layer (breaks cache and diffs); assemble prompts at the call site from concatenated strings; half-pin (pinned prompt + moving alias, or pinned model + edited file).

---

# Measuring a prompt

Without a number, prompt arguments resolve by seniority. Clients will repeat your quality figure in rooms you are not in.

**Frozen case set** + **gold labels written from the source document**, before you measure — not by correcting the system’s output (that drifts gold toward the model). Resolve label disagreements and record why. Adding a case **resets** the comparison.

**Score with code**, not judgment: parsed object vs gold. Exact enum match, set compare, citation present, leakage patterns. Reproducible, free, runnable on every change. Do not use an LLM-as-judge as the scorer.

Recurring metric **families** (name them the same on every engagement): required-evidence recall; correct version selection; grounding/citation correctness; routing and escalation accuracy; PII leakage. Also, when tools exist: tool-call correctness, idempotency.

**Score per field, then aggregate.** Whole-object exact match hides which field broke.

**Miss vs invent are different.** Gold `absent` + plausible value = **invention** (compliance may act; invisible). Gold present + returned absent = **miss** (visible). Gold `ambiguous` + confident pick of one reading = fail even if that value appears in the document. Invention on an actionable field is worse than a miss.

**Twelve cases → report counts and the denominator.** One case is >8 percentage points. 11/12 vs 10/12 is not a measurable win. Do not report 91.7%. A real change moves several cases or the same case across repeated runs.

**Scorer is versioned.** Do not change scorer and prompt in the same commit. Join score records to call records on run, case, model, prompt version. Rescore old runs when the metric changes.

Worked scoreboard signal: three **inventions** on ownership threshold is the finding; aggregate 51/60 hides it. Three `superseded_by` misses = version-selection bug. 1 of 2 ambiguous is a note, not “50%.” One `effective_date` miss is noise until it repeats.

**Do not:** move the case set mid-comparison; gold from system output; percentages on twelve; scorer leniency disguised as prompt improvement.

---

# Reading a model comparison

**Client-priority article.** Full notes: `Forward Deployed Engineer/Reading a Model Comparison.md`. This is often the first artifact a client forwards. Treat every number as if it will be quoted without you in the room.

The comparison report leaves the team. Caveats you omit will be omitted downstream. Twelve-case results must not become production commitments.

**Quality, cost, and latency in one table, per task.** Separate tables produce three winners. Combined averages across tasks describe a system nobody is building.

- **Cost per case** includes failures, transport retries, schema repair. A model that fails validation more often can look cheaper per success and be **more expensive per case**.
- **Latency is a distribution.** Median + **maximum**, state N, and **where and when** it was measured. Not mean (hides the 11s retry). Not p95 on twelve. Measure first attempt → final result. A latency figure without place and time is not reproducible.
- **Quality is counts by error type.** 8/12 miss vs 8/12 invent are not equivalent.
- Every row names **prompt version**. Unadapted prompt = **untested** (transfer), not a model loss.
- Twelve cases: eliminate hard-constraint failures; spot systematic misses (never reports superseded). Do **not** rank a 1–2 case gap; do not quote a %; do not claim production volume behavior. Write **what the set does not show**.
- Recommendation: one or two sentences per task, model + prompt version + **what would reopen it**. Point at `docs/model-decision.md` for constraints and rejected alternatives.

**Do not:** three tables / three winners; cost from successful calls only; undisclosed prompts; 91% from eleven of twelve as a client commitment.

---

# Embeddings, vector stores, and chunking

**Important.** Full notes: `Forward Deployed Engineer/Embeddings, Vector Stores, and Chunking.md`.

RAG is how an LLM answers from material that **does not fit the context window**, or that it was **never trained on** (a policy manual, a codebase, an internal knowledge base). Stuffing the source into the window fails once the source is larger than the window, and is expensive and slow even when it fits. Retrieve only the relevant portions and put **those** in the prompt.

| Piece | What it is | What it is not |
|---|---|---|
| **Embedding** | A vector (list of numbers) that captures meaning. Similar meaning → mathematically close. Unrelated meaning → far apart. | Keyword match. Not the answer model. |
| **Embedding model** | A **separate** model whose job is to produce vectors. | The LLM that writes the answer. |
| **Vector store** | Stores vectors and runs **similarity search** (typically cosine): given the query vector, return the closest stored vectors, even across millions. | Something you build from scratch. |
| **Chunk** | A piece of a source document, embedded on its own. | The whole document as one vector. |

Similarity search finds “I forgot my login credentials” for “How do I reset my password?” even when they share no words. A keyword search misses that.

**Chunk size is a quality knob:**

- **Too large:** the vector reflects the theme of a long span, not the paragraph that answers a narrow question.
- **Too small:** a sentence pulled out of its explanation is ambiguous once retrieved.
- **Overlap** so a sentence on a boundary is not lost in both neighboring chunks.
- Prefer splits on **natural boundaries** (paragraphs, sections, headers). A fixed character/token window cuts ideas in half. If you use a fixed window, size it in **tokens**, not a guessed characters-per-token ratio `(Tokens, Context)`.

**Same embedding model at index time and query time.** A query vector from a different model is not comparable to the stored vectors.

**Basic pipeline:**

1. Chunk source documents.
2. Embed each chunk.
3. Store the vectors.
4. At query time, embed the user query with the **same** model.
5. Search for the closest chunks (`top_k`).
6. Pass those chunks **plus the original query** to the LLM so the answer is grounded in retrieved material.

Nearly every “AI over our documents” engagement **is** a RAG system. Chunk size, embedding quality, and how many chunks you retrieve change the answer. That is the design, not a library default.

**Do not:** embed a whole large document as one vector; mix embedding models between index and query; stuff the corpus because the window is big enough; treat the vector store as the system of record `(When Not to Use an LLM)`.

---

# Hybrid and agentic retrieval

**Important.** Full notes: `Forward Deployed Engineer/Hybrid and Agentic Retrieval.md`. Builds on the basic pipeline above.

Pure vector search matches **meaning**. It misses **exact strings**: error codes, SKUs, product names, phrases. Rare or technical terms may be weak in the embedding model’s training, so specialized vocabulary ranks poorly. Production corpora need both natural-language questions and precise lookups.

**Hybrid retrieval** runs both and merges:

1. Vector search → semantically similar chunks.
2. Keyword search (exact or fuzzy, the same idea as grep) → chunks that contain the terms.
3. Merge. A chunk in **both** lists is a strong match: related in meaning and containing the exact terms.

**Do not blend the two scores.** Scores are not comparable, and min-max normalization puts a 1 on a list that matched nothing. Combine by **reciprocal rank fusion** (see **Two retrievers, one result set**): `1 / (60 + rank)`, same `chunk_id`, same `build_id`.

Use hybrid on enterprise data with codes, names, and terminology a purely semantic search will not surface, and when you have already seen vector-only search fail on needle-in-a-haystack queries.

**Agentic retrieval** lets the model decide the retrieval, instead of one fixed step:

1. Receive the question.
2. Decide **what to search** — often a reformulation, not the user’s wording (“why did the numbers look off recently” → a targeted query).
3. Retrieve, then judge whether the results actually answer the question.
4. If not, search again with a refined query. Do not answer from a weak first pass.
5. Generate only once the context is sufficient.

A fixed pipeline retrieves **once, no matter what**, and generates from whatever came back, including a weak set. Agentic retrieval can say the results do not answer the question and take another pass.

| Approach | Use when | Cost |
|---|---|---|
| Plain vector | Straightforward, well-scoped queries against one organized corpus. **Start here.** | One search |
| Hybrid | Exact terms, codes, or names, or vector-only already misses grep-able queries | Two searches + a merge |
| Agentic | Complex or ambiguous questions where one pass will not capture what is needed | More model calls, more latency, harder to debug |

Agentic retrieval is a **deliberate tradeoff**, not the default. Justify it. Do not pick the most sophisticated option out of habit.

**Do not:** rely on vector search alone for error codes and SKUs; add agentic loops to every RAG system; let a fixed pipeline answer from a retrieval set you already know is weak.

---

# Failure modes: lost-in-the-middle, retrieval mismatch, hallucination under low recall

**Important.** Full notes: `Forward Deployed Engineer/Failure Modes: Lost-in-Middle, Retrieval Mismatch, and Hallucination Under Low Recall.md`.

A bad RAG answer has a **named** failure. “The AI got it wrong” is not a diagnosis. These three are the ones to recognize.

**Lost-in-the-middle.** Models use the **beginning and end** of a long context more reliably than the middle, even when the window is large enough `(What Actually Differs — context is capacity, not comprehension)`. Retrieval found the right chunk (say, third of five) and the answer still ignores it or gets it subtly wrong. The failure is **how the context is used**, not whether retrieval succeeded.

Mitigations: retrieve **fewer, more targeted** chunks; put the most relevant chunks at the **start or end**, not the middle; **rerank** by relevance before the chunks go to the model.

**Retrieval mismatch.** The chunks are topical or share wording, but they do not answer the question. “Expense reimbursement for international travel” comes back as the general expense policy and misses “cross-border business expenses.” Causes: chunking that **splits** the detail from its surrounding context so only one piece is retrieved; **vocabulary mismatch** between the question and the document (hybrid retrieval is the fix); embedding weakness on domain terms. Recognize it by **reading the retrieved chunks**: the needed information is not there. No prompt change fixes an answer built from the wrong source.

**Hallucination under low recall.** Recall = how much of the actually relevant source material was retrieved. Low recall means important content was missed. The model then continues with the most probable text, blending training knowledge with a thin context, and does not reliably say it lacks information. The output **looks like** a grounded answer. A reviewer who only checks tone will miss it.

Mitigations: instruct the model to answer **only from the provided context** and to say when that context is not enough `(Writing the Instruction — unknown is first-class)`; **measure recall**, do not judge the system by whether the final answer sounds good; show **citations** so a human can check the claim against what was retrieved `(Few-Shot and Grounding)`.

**Debug in this order. Do not swap the questions.**

1. Did retrieval find the right information? If not → retrieval mismatch or low recall. Fix chunking, hybrid search, or the query. Do not tune the prompt.
2. Given what was retrieved, did the model use it? If not → lost-in-the-middle, or hallucination filling a gap. Fix ordering, how many chunks you pass, the instruction to abstain, and citations.

| What you see | Layer | First move |
|---|---|---|
| Right chunk is in the prompt; answer ignores the middle one | Use of context | Fewer chunks; put the best at the ends; rerank |
| Retrieved text is on-topic but does not contain the answer | Retrieval | Inspect chunks; fix split, vocabulary, or hybrid |
| Answer is fluent and complete; sources do not support it | Generation on thin context | Abstain instruction; measure recall; require citations |

**Do not:** pass every loosely relevant chunk; debug by rewriting the prompt before you read the retrieval set; trust a confident answer that has no citation into the retrieved text.

Add a **third** question before you tune anything: was the source data correct, current, and complete? A faithful retriever on a stale or incomplete corpus looks like a retrieval bug `(Data Quality and Lineage)`.

---

# What an embedding actually is

Full notes: `Forward Deployed Engineer/Week5/What an Embedding Actually Is.md`.

An embedding is a **fixed-length vector** from one model. It is a position, not a readable summary, not invertible, not auditable meaning. Its only job is nearness.

**The score is a rank inside one query, not a confidence.** 0.61 is not 61% relevant, not comparable across questions, and not stable after a model change. Do not hard-code a similarity cutoff (0.75) from a handful of scores.

**The unit you embed is a decision.** Output length is fixed. A document covering eight subjects sits near none of them: moderate rank for every question, strong rank for none.

**Same function, same model, at index time and query time.** A retrieval model places a question near its answer; that is not general text similarity. A query prefix used only on one side produces no error and destroys recall.

**Dimension is a cost, not a quality dial.** Pin model id and dimension in `src/rag/config.py`. The vector column length comes from that number so a mismatch fails at insert.

**A model change invalidates every stored vector.** There is no conversion. Record which model built the index. A full rebuild from source is routine. If rebuilding is frightening, that fear is the defect. A mixed-vintage index (some rows re-embedded, some not) looks like a document problem.

**Similarity cannot carry the facts this corpus turns on.** Negation barely moves a vector. A 25% threshold and a 10% threshold are near-identical. Section numbers, version strings, and dates fragment. Those belong in **metadata and predicates**, not in the vector. Do not write a test that expects cosine to separate current from superseded. Delete it. Test the **filter**.

**Do not:** treat a score as a probability in a case note; mix models in one index; embed the query differently from the corpus.

---

# Chunking a policy document

Full notes: `Forward Deployed Engineer/Week5/Chunking a Policy Document.md`.

A chunk is the retrieval unit **and** the citation unit. Three jobs pull apart:

| Job | What it wants |
|---|---|
| Retrieval | Small and single-subject |
| Citation | A real section a human can open |
| Answering | The rule **and** its exception in the same passage |

**Structure first, size second.** Split on the document’s own numbered headings. Subdivide only sections that are too long. A fixed character or token split cuts sentences, tables, and defined terms, and often leaves a chunk with **no heading**, so it cannot be cited. A chunk with no section id still gets answered from; the citation is filled with the nearest plausible heading. That grounding failure is created at **ingestion**.

**What you embed is not what you cite.**

| Field | Role |
|---|---|
| `text` | Unmodified passage. Citation resolves here. The model sees this at answer time. |
| `embed_text` | Same passage with **document title and section heading prepended**. This is what you embed, and what lexical search indexes. |
| `embedding` | The vector of `embed_text`. |

Without the prefix, “shall obtain the partnership agreement…” has no subject and ranks for nothing.

**Metadata is copied onto every chunk**, not only the document: `doc_id`, `version`, `effective_date`, `superseded_by`, `jurisdiction`, `entity_types`, `section`, `section_title`, `ordinal`. The chunk is what gets retrieved, so anything you filter on has to be on the row.

**`chunk_id` is a contract.** Derive it from `doc_id`, version, section path, and an ordinal (`kyc-ie-0004:v4.2:4.2:0`). Never from insert order. Never from a hash of the text. `text_sha256` is a **separate** column: the id stays; the hash says the text moved. Putting the hash in the id renames the chunk on every edit and silently invalidates gold labels.

**Overlap** is modest, at structural boundaries, recorded in config. It repeats text so a condition in the last sentence stays with the requirement in the next. Cost: a fatter index, near-duplicate candidates that crowd out a second passage, and gold text that now lives in more than one chunk so recall is ambiguous. Do not raise overlap because a metric went up.

A chunk that **spans a document boundary** is worse than a short chunk: half the text then carries the wrong `doc_id` and version.

**Do not:** primary split on a token count; embed raw `text`; hash the text into the id; merge across documents to hit a minimum size.

---

# The vector store is a database

Full notes: `Forward Deployed Engineer/Week5/The Vector Store Is a Database.md`.

Rows are chunks. The vector is one more column next to `section` and `effective_date`. Primary keys, foreign keys, nulls, transactions, and migrations still apply.

**`index_build` records identity:** model, dimension, chunk strategy, overlap, corpus manifest hash, ingest git sha, `built_at`. Every chunk row carries `build_id`. Ingestion writes a **new** build. It does not update the old one in place. Two builds can coexist and be compared. A mixed-vintage index becomes detectable.

**Cosine similarity: higher is better. pgvector `<=>` is cosine distance: lower is better.** For unit-length vectors, distance = 1 − similarity, and a **dot product equals cosine**. Check that the provider already normalized the vectors. Do not assume it. Sorting the wrong way still returns rows — the **least** relevant ones. It looks like catastrophic retrieval. It is a sign error. Do not carry a 0.75 similarity threshold onto a distance column.

**Dense search always returns k rows.** A question about a subject you do not have still returns k nearby chunks. No “no match” signal. Deciding the answer is not here is a job above the store.

**Start with exact search.** HNSW and IVFFlat return **nearly** the nearest neighbors; a genuinely nearest chunk can be missed. Recall depends on build parameters **and** query-time parameters. At this corpus size an exact scan is fast enough. Adopt an approximate index only against a measurement. A selective filter plus HNSW can miss neighbors the filter would have kept.

**The corpus is the source of truth.** Every row is reconstructible from `corpus/source/` plus config. Never repair by editing a row. The store is its own process; connection config comes from the environment and is never committed. It can be down, slow, or **up and empty**. An empty store with no defined branch produces a fluent answer built on nothing.

**Do not:** hand-patch rows; omit `build_id` from the query; treat an approximate index as a default.

---

# Two retrievers, one result set

Full notes: `Forward Deployed Engineer/Week5/Two Retrievers, One Result Set.md`.

Dense and lexical **fail in opposite directions**. Neither is a fallback for the other. Do not “try dense, and if it looks weak, try lexical.” A real question needs both at once.

| Retriever | Does | Does not |
|---|---|---|
| **Dense** | Bridge vocabulary (“what documents do we need” → “Required documentation”) | Section numbers, version strings, defined terms, proper names. **Always returns k rows.** |
| **Lexical** (BM25; Postgres uses `ts_rank_cd`, not BM25, so numbers will not match a search engine) | Match `6.4`, `v4.2`, a defined term, a name, exactly | Bridge vocabulary. Has a match predicate (`@@`): no hits → **empty list**. |

Index **`embed_text`**, not raw `text`. The prepended title, version, and heading are why a query naming `6.4` can match at all.

**Reciprocal rank fusion.** Throw away the scores. Rank starts at 1.

`fused score = sum of 1 / (60 + rank)` over every retriever that returned that `chunk_id`.

**60** dampens being first. Appearing respectably in **both** lists beats being first in one and absent from the other. No weight. The fusion code does not change when the embedding model changes. Cost: an outstanding match and a merely adequate one at the same rank are treated the same.

**Three limits, not one:** dense k (wide, for example 20), lexical k (also wide), fused rows you keep (truncate here). Two lists of five rarely overlap, so the agreement signal is mostly absent.

Both queries hit the **same table**, the **same `build_id`**, and the **same `chunk_id` scheme**. Fusion is a join on a string. A separate search engine with its own ids turns every failed mapping into a silently dropped candidate.

**Ties break by `chunk_id`**, not by whichever row the database returned first. Otherwise the score moves between runs while nothing changed.

**Fusing across builds is silent.** Stable ids mean a stale `build_id` on one side still joins. The result is a blend of two corpora. Omitting the build filter does the same thing.

Fusion does not know “superseded.” It will correctly promote a withdrawn chunk that both retrievers found. Currency is the next step.

There is **no separate dedup step after fusion**. The same `chunk_id` is already one row. Overlap can still leave near-identical chunks with **different** ids; that is a cost of overlap, not something you clean up after RRF.

**Do not:** weighted score blend; lexical index on `text`; one k everywhere; fuse across `build_id`s.

---

# Filtering is not ranking

Full notes: `Forward Deployed Engineer/Week5/Filtering Is Not Ranking.md`.

**Ranking** orders by degree. **Filtering** removes by fact. A document either was in force on the review date or it was not. Boosting “recent” or stuffing “current version” into the query turns a determined answer into a preference a strong lexical match can outvote.

**Latest is not current.** `superseded_by` points forward. Current on a date = effective on or before that date, and not superseded by a revision that is itself effective on or before that date. Drop the second half and you return a withdrawn policy. Drop the qualification on the first half and you return a policy **not yet in force** (organizations publish revisions ahead of their effective dates). `superseded_by IS NULL` is the wrong test.

Reuse the existing `select_current_version` rule. Do not write a second implementation because the input type changed.

**Pre-filter, do not post-filter.** Post-filtering takes top k then throws away failures. A request for 20 can come back as 4 because superseded near-duplicates crowded the top. Worst on the longest version histories.

Filter **both** retrievers with the same `FilterSpec` **before** fusion. If only dense is filtered, fusion rewards a chunk for appearing in one list for a reason that is not relevance.

`FilterSpec` travels with every query and comes back with every result: `build_id`, `as_of`, optional `jurisdiction`, optional `entity_type`, `include_undated_documents` (default false). Jurisdiction, entity type, and as-of date do **not** belong in free-text query rewriting.

**A null date is not false.** A date comparison against null is unknown, so undated rows vanish quietly. Decide explicitly: exclude and **count**, or include and flag. Count the nulls.

An empty result is correct when you hold no policy for that jurisdiction, or the date is before anything was in force. Return nothing **and** return the filter that emptied the set, so the layer above can tell “no matching documents” apart from “no relevant documents.”

**Do not:** post-filter; filter only one retriever; treat `superseded_by IS NULL` as current; let a null date disappear without a count.

---

# Retrieval in more than one step

Full notes: `Forward Deployed Engineer/Week5/Retrieval in More Than One Step.md`.

One query assumes the analyst’s phrasing lands near the corpus, **and** that the whole answer sits in one region. Both fail, independently. A cross-reference (“apply Section 6.4”) is not in the first candidate set. Everything downstream then cites a real section that is missing its exception.

**Query rewriting** may add phrasing. It may **not** add facts (jurisdiction, entity type, date, percentage). Those stay in `FilterSpec`. An instruction “don’t add facts” is only a request. An assertion that no rewrite introduced a jurisdiction token is a check you can run. Show the model the section titles that survived the filter. The analyst’s question is **untrusted text** and goes inside markers.

**Decomposition** splits a sentence that is two questions. Each step has its own text and its own filters. The plan is **data**: an ordered list you can print and compare. Origins: `original`, `rewrite`, `decomposition`, `cross_reference`. If the “sub-queries” are three paraphrases, you pay three times and get one passage. The tell is high overlap between step result sets.

**Coverage is code, not the model.** Before another pass:

- Did every step return a non-empty filtered set?
- Does the set contain a chunk from each section the rewrite named?
- Does any returned chunk name a section that is **not** in the set?

The third check is a pattern match over text you already have. It costs no model call and produces a specific next query (“Section 3.5”). Do not ask the model whether it has enough context. It will say yes.

**Bounds are constants:** max passes, max steps, max retrievals. Stopping at `pass_limit` is a legitimate outcome. Record it. `stop_reason`: `coverage_satisfied`, `pass_limit`, `step_limit`, `empty_after_filter`. A known gap is more useful than an unknown one.

**Inside a step:** reciprocal rank fusion. **Across steps: union, not fusion.** Steps look for different things. Fusing across them penalizes a chunk for being absent from a list that was never looking for it. Keep each chunk’s best within-step fused rank. Ties break by `chunk_id`.

**The model rewrites and decomposes. Code decides** which steps run, in what order, under which filters, whether another pass happens, and when to stop. The model does not select tools or call systems. The second search is code. Letting the model own that loop is agentic retrieval: a different driver.

Record the plan, each step’s query and `FilterSpec`, the candidates, the coverage results, and the stop reason. Temperature 0 does not make rewrites deterministic.

**Do not:** let a rewrite insert facts; loop until the model feels it has enough; judge coverage with the model; keep only the final merged set.

---

# Structured data vs unstructured, and SQL

Full notes: `Forward Deployed Engineer/Week6/Enterprise Data Context: Structured vs. Unstructured, and SQL Refresh Applied to Retrieval.md`.

| Question | Tool |
|---|---|
| “What’s our reimbursement policy for international travel?” | Unstructured document → RAG |
| “How many customers signed up last month on the enterprise plan?” | Structured rows → SQL (`COUNT`, `WHERE`, `JOIN`) |

Embedding a customer table and searching it semantically is the wrong tool for a count. SQL is exact. Vector search is approximate. Enterprise systems have both. Route the question before you retrieve.

A growing pattern lets a model choose the route, and sometimes generate the SQL. That does not move arithmetic or the lookup into the model. The database still computes. The model may only draft the query; reflection or a human still checks that “last quarter” and “churned” match the real schema `(When Not to Use an LLM)`.

**Do not:** force every question through embeddings; answer “how many” from retrieved chunks.

---

# Data quality and lineage

Full notes: `Forward Deployed Engineer/Week6/Data Quality and Lineage Where They Affect RAG Outcomes.md`.

Chunking, embedding, reranking, and eval all assume the source is trustworthy. On a client drive it often is not.

| Source problem | What it looks like |
|---|---|
| Outdated document next to the current one | Confident answer from the stale wording that matched better |
| Near-duplicates | Crowded candidate set; inconsistent answers |
| Missing section or appendix | Retrieval mismatch or low-recall hallucination, root cause **upstream** |
| PDF / scan artifacts, broken tables, missing headers | Chunks of garbage that still embed |

No better chunker, embedding, or reranker fixes a bad source. The system retrieved what it was given.

**Lineage:** for any retrieved chunk, trace the source document, version, and when it was last updated. In financial services and insurance that is often a compliance requirement. Surface it with the answer: source file, version, section, last updated. Stale dates are then visible. A source change should trigger re-ingestion, not a silent stale index.

**Debug order becomes three questions:**

1. Was the source correct, current, and complete?
2. Did retrieval find the right part of that source?
3. Did the model use what was retrieved?

**Do not:** spend a long time tuning a pipeline whose corpus is duplicated, superseded, or incomplete.

---

# Reranking and evaluation harnesses

Full notes: `Forward Deployed Engineer/Week6/Reranking and Evaluation Harnesses.md`.

**Rerank is a second stage, not a scan of the corpus.** Vector search cheaply returns a wide set (for example 20). A slower, more precise model re-scores those candidates against the query. Keep a few (for example 5) for the LLM. Reranking the whole corpus on every query is too slow.

It helps two failure modes: a relevant chunk that embedding similarity ranked too low can be promoted (retrieval mismatch); fewer, better chunks mean less material buried in the middle (lost-in-the-middle). It does not fix a bad chunk or a bad source.

**Evaluation harness** = a frozen set of questions, each paired with the chunk that should be retrieved and/or the facts the answer must contain. Run it like pytest whenever chunk size, the embedding model, or a reranker changes.

Measure **separately**:

- **Retrieval:** recall at k — did the expected `chunk_id` appear in the top k?
- **Answer:** does the generated text contain the expected facts (string match for straightforward facts; do not use an LLM-as-judge as the default — score with code against gold)?

“It seemed fine on three examples” is not an answer to “how do you know this works?”

**Do not:** rerank the entire corpus; change chunk size and judge by impression; collapse retrieval and answer into one score.

---

# Data layer choices

Full notes: `Forward Deployed Engineer/Week6/Data Layer Choices: Databricks, Snowflake, Azure Cosmos DB, and AWS-Native Options.md`.

Recognize the category. Do not relearn the concept because the vendor name changed.

| Category | Third-party | AWS-native |
|---|---|---|
| Data warehouse (SQL, analytics) | Snowflake (storage and compute scale separately; runs on AWS) | Amazon Redshift |
| Lakehouse / big data + ML | Databricks (lake + warehouse, Python/Spark; multi-cloud) | AWS Glue (managed ETL), Amazon EMR (Spark) |
| NoSQL document database | Azure Cosmos DB (**Azure only**) | Amazon DynamoDB |
| Standard relational database | — | Amazon RDS (PostgreSQL, MySQL) |

Cosmos DB stays conceptual in this program because the target cloud is AWS. Snowflake and Databricks can run on AWS.

This program: stay in a free tier, target AWS. If neither an AWS free tier nor a third-party trial on AWS is clearly safe, use local SQLite or self-managed PostgreSQL. Verify current free-tier terms on the vendor site before building. Do not risk unexpected cost.

**Do not:** treat Cosmos DB as something you will build against here; pick a warehouse for a low-latency application backend, or DynamoDB for analytics.

---

# What makes a system agentic vs a workflow

Full notes: `Forward Deployed Engineer/Week7/What Makes a System Agentic (vs. Workflows).md`.

A **workflow** is a sequence you decided in advance. It can call an LLM, branch, and have many steps. A RAG pipeline (embed → retrieve → rerank → generate) is a workflow. The LLM does not choose the sequence.

An **agentic** system loops: assess state, choose the next action, act, observe, decide whether to continue. The number and kind of steps are not fixed. Agentic retrieval (reformulate, search, judge, search again) is this. Multi-step retrieval where **code** decides the next search is not.

| | Workflow | Agent |
|---|---|---|
| Who picks the next step | You, in code | The model, from what it just saw |
| Same request twice | Same path | Possibly a different path |
| Debug a wrong refund | Inspect the one path | Reconstruct which decisions that request took |
| Cost | The steps you wrote | An extra model call per decision |

High-volume, near-identical cases (thousands of duplicate-charge tickets) want the workflow: cheaper, faster, auditable. A small set of accounts where “look up billing” means different systems per client can justify an agent, because a fixed workflow grows a pile of special cases.

**Check before you reach for an agent:** could a fixed workflow, even with several branches, handle this correctly? If yes, use the workflow.

**Do not:** call every LLM pipeline an agent; accept “we need an agent” without asking whether the steps are already known.

---

# Tool use and function calling

Full notes: `Forward Deployed Engineer/Week7/Tool Use and Function Calling.md`.

An LLM only generates text. Without a tool, “status of ticket 4521” is a refusal or a fluent invention. With a tool, the model **requests** a call; **your code executes it** and sends the result back; the model then answers from that result.

Cycle: prompt + tool definitions → model requests a tool and arguments, or answers directly → you validate and run the function → you return the result (or a structured error) → model answers.

A question can need **two tools in sequence** (PTO balance, then shift coverage). A general question (“how does PTO usually work?”) should call **no** tool. If the model calls one anyway, the description did not say when **not** to use it.

**Descriptions are the control.** “Gets data” is why the model calls the tool on a general question. Name the purpose, the parameters, and when to use it: “Look up status, priority, and assignee for a specific existing ticket id.”

**Validate arguments before any side effect.** `issue_refund(order_id, amount)`: does the order exist, does the total cover the amount, is it already refunded? A hallucinated id or amount is real financial harm. Same rule as validating external input; the caller happens to be a model.

**Tool failure is data.** Return `{"error": "scheduling system temporarily unavailable"}`. Do not crash the turn, and do not let the model invent the missing fact.

Too many overlapping tools make the choice unreliable.

**Do not:** let the model run the function itself; execute side effects on unvalidated arguments; omit a failure result and hope the model abstains.

---

# Planning, reflection, and ReAct

Full notes: `Forward Deployed Engineer/Week7/Reflection, Planning, and ReAct Patterns.md`.

**Planning:** write the sequence before acting, then revise if a step surprises you. Without a plan, an onboarding agent provisions email, gets stuck on one unusual request, and never returns to the shared drive. The plan is a checklist you can compare to what actually ran. A bad outcome is either a bad plan or a deviation from a good one.

**Reflection:** the agent checks its own draft against explicit criteria before it is final. Incident email: did it leak internal detail, or promise a time nobody confirmed? Generated SQL: does “last quarter” match the date range, and does “churned” match the status value in this schema? Reflection is not a substitute for an eval harness or a human review.

**ReAct** interleaves them. At each step the model states a **Thought** (what is still unknown), an **Action** (a tool call), and reads the **Observation**, then repeats until it can answer. The thought is what makes a wrong action debuggable. A silent jump from question to tool call is not.

A robust agent often uses all three: plan up front, ReAct to execute (and skip a planned step when an observation makes it unnecessary), reflect before anything is sent. Reflection still matters after a complete-looking draft: the draft may promise a fix time the engineering data never confirmed.

**Do not:** treat the thought trace as audit evidence `(Reasoning in the Prompt — reasoning text is not evidence)`; skip reflection because the plan looked complete; ship a SQL agent that never checks the query against the real schema.

---

# MCP: building servers

Full notes: `Forward Deployed Engineer/Week7/MCP Deep Dive: Building Servers, Not Just Consuming Them.md`.

**MCP** (Model Context Protocol) is an open standard for how an AI application discovers and calls external tools, and how those tools return data. One server, many clients. Without it, three AI products that all need the HR system means three custom integrations.

| Piece | What it is |
|---|---|
| **Host** | The app the user touches (chat, IDE, agent framework) |
| **Client** | The piece inside the host that speaks the protocol |
| **Server** | What you build. Exposes capabilities. You do not call the LLM; an LLM’s client calls you |

A claims question: the host’s client finds `get_claim_status` on your server, your code queries the database, the client returns the result, the host writes the answer.

What a server can expose:

| Kind | Use for |
|---|---|
| **Tool** | An action with parameters (`search_knowledge_base(query)`, `flag_for_review`) |
| **Resource** | Context to read, not an action (a standing onboarding document) |
| **Prompt** | A reusable template (“draft a complaint response about {topic}”) |

Start with **tools**. A read (`get_claim`) is low risk. A write (`flag_for_review`) changes a record: validate harder, and test the failure path (unknown id must error, not no-op).

**Test the function first**, with known inputs, before any agent is connected. Then test that the server lists the tools and handles a protocol call. Same order as unit then integration.

You are writing code an LLM can discover. The tool description rules above still apply.

**Do not:** a one-off plugin per AI product when an MCP server would be reused; connect an agent before the tool functions have tests; treat a side-effect tool like a read.

---

# When agents are the wrong answer

Full notes: `Forward Deployed Engineer/Week7/When Agents Are the Wrong Answer.md`. Same discipline as **When Not to Use an LLM**, one level up. The cost of the wrong choice is higher: more money, more latency, harder debug.

Ask **in order**. Stop at the first yes:

1. **Plain code?** Field equals a value, a total, a date, a threshold. No LLM.
2. **One LLM call?** Language understanding, no dynamic steps (classify urgency).
3. **Fixed workflow with a few LLM calls in a sequence you wrote?** Steps are knowable in advance (password reset: verify identity, check lock vs reset, send the link, log it; monthly compliance report: same query, same checks, same template).
4. **Agent** only when the next step depends on what you discover, you must judge when to stop or switch approach, and the cases are too broad to enumerate as branches (a $340 billing discrepancy that might be a duplicate, a plan change, a proration, FX, or a bug).

Signs the workflow wins: you can write the sequence for every realistic case; consistency matters more than flexibility (auditors); latency and cost are tight; a plan is unnecessary because it is already a sequence.

The honest test: if you wrote the fixed sequence, would it cover the realistic cases? If yes, an agent is solving a problem that does not exist.

Clients will ask for “an agent” for password resets and compliance reports. The job is to say the steps are already known, and to ship the workflow.

**Do not:** skip the four questions because agentic sounds current; let an agent decide the shape of a report that must be identical every month.

---

# State and memory

Full notes: `Forward Deployed Engineer/WEEK8/State and Memory Management.md`.

An LLM call is stateless. It knows only the context you send. A multi-step agent, and any system where several agents work one case, needs a deliberate decision about what persists, where it lives, and who can read it.

**Short-term (working) memory** is the current task only. Inside one equipment-request loop the agent must remember the role and the policy limits it already looked up. When the final answer is produced, discard that context. The next employee's request starts empty. Build the context list **inside** the request, not once for the process.

**Long-term (persistent) memory** survives sessions. A support agent should recall that this customer already reported the problem on Tuesday. An underwriting case can span days: intake today, risk once documents arrive, a recommendation later. Store the durable facts (customer id, prior ticket, resolution, what a previous agent already gathered) in a database `(Structured data vs unstructured, and SQL)`. Do not re-derive them, and do not leave them only in one conversation's window.

**Two ways agents share work on the same case:**

| Pattern | How it works | Tradeoff |
|---|---|---|
| **Shared store** | Intake, risk, and recommendation all read and write one case record | Simpler to start. Any agent can read or write something it should not |
| **Message passing** | One agent's output is packaged and handed in as the next agent's input. Nobody reads a store it was not given | Cleaner boundaries. More plumbing at each handoff |

The event-driven flow in **Integration patterns** is message passing, formalized around events and queues.

Memory design fails in three specific ways:

- **Stale data.** An agent reads a store a prior step has not updated, and decides on incomplete facts.
- **Context bleed.** The context list is created once, globally, so employee A's tenure is still there when employee B's request starts. One happy-path test case will not catch it. Test a second user, and test overlapping requests. Personal data makes this a serious bug, not a convenience issue.
- **Unbounded growth.** Long-term memory that is never pruned or summarized outgrows what you can put in a window `(Tokens, Context)`.

This is **product** state. Coding agents on one repository still do not share uncommitted files. **You** remain the coordinator `(Parallel Agents — you are the only shared memory)`.

**Do not:** reuse one context list across requests; make each agent start from scratch when a previous agent already wrote the fact; treat a shared store as safe because it was easy to wire up.

---

# Observability, tracing, and budgets

Full notes: `Forward Deployed Engineer/WEEK8/Observability and Tracing; Cost and Latency Budgeting.md`. Extends **Latency and Cost**: write the cost ceiling and the latency budget before you prefer an architecture. This section is how you see a multi-agent run after the fact, and how you attribute spend and time **per step**.

Ordinary debugging assumes one line and the same result if you run the same input twice. A multi-agent system breaks both. The fault can be any agent, or the handoff between them, and sampling means a rerun may not fail the same way `(Sampling)`. **Observability** instruments the system so you can see what happened without reproducing it live. A **trace** is the ordered record of one request: every agent, every tool, every decision.

A ReAct thought / action / observation printout is already a trace. Production logging adds, for each step:

- which agent or component ran
- what input it received, and where that input came from (another agent, a tool, the user)
- what output or decision it produced
- timestamp and duration (this is the latency budget, measured)
- tokens and cost, if the step called a model
- errors and retries

A wrong underwriting recommendation is then bad intake data, an enrichment step that never wrote what it should have, or a risk score that misapplied its logic. It is not one opaque pipeline.

**Cost.** Several agents, each making model calls, multiply the per-call cost in **Latency and cost**. Track **cost per agent per case**, not only the month-end total. Illustrative shape only — do not quote as live prices: intake $0.02 + enrichment $0.05 + risk $0.03 + recommendation $0.02 ≈ $0.12/case; × 10,000 cases/month ≈ $1,200. Set the expected budget from estimated tokens **before** you build. Compare actuals so a prompt that grew, or a step that started retrying, is caught before the client sees the bill. Still sum retries and failed calls into the case `(Latency and Cost — cost per case, including retries)`.

**Latency.** Sequential agents add. Four agents at about 2 seconds each is at least 8 seconds before tools, retries, and an inner ReAct loop. `duration_ms` on the trace is what names the bottleneck: enrichment at 18 of 25 seconds, because of an external call. Optimize or parallelize **that** step. Ask whether the next step actually depends on the previous output; if it does not, run them concurrently. A real-time chat and an overnight batch are different architectures. Pick the budget first `(Latency and Cost — personas)`.

**Do not:** debug a multi-agent failure by rerunning and hoping it repeats; tune the whole pipeline when one step owns the time; discover the budget after the client complains about the bill or the wait.

---

# Integration patterns: APIs, webhooks, queues, events

Full notes: `Forward Deployed Engineer/WEEK8/Integration Patterns: APIs, Webhooks, Queues, and Event-Driven Flows.md`.

Agents connect into systems the client already runs. Pick the pattern for the need, and define the failure before the demo. A failure here does not stop at one request. It can stall every later agent.

| Pattern | Use when | What fails | What you do |
|---|---|---|---|
| **API** (synchronous) | You need the answer now. Risk scoring calls a credit bureau and cannot score until the response returns | The caller is blocked. A slow or down API becomes your latency and your outage. Tracing is how you see it | Timeout. Limited retries with **backoff** (wait longer between tries; do not hammer a struggling API). Then escalate for manual review. Do not guess the score |
| **Webhook** | The other system should tell you when something happened. A document upload starts enrichment. Do not poll "anything new?" on a timer | Delivery can fail silently, and enrichment never runs | React when the event arrives. Add a periodic **reconciliation** check for uploads that should have notified you and did not |
| **Queue** | Load is bursty. 500 intakes arrive at once; risk scoring cannot call the credit API for all of them immediately | Nothing crashes. The queue grows, and cases quietly miss the time the client was promised | Intake drops a message and moves on. Risk scoring pulls at a rate it can sustain. **Monitor queue depth** and set the threshold that means this needs attention |
| **Event-driven flow** | Several systems, several steps, and no component should call the next one directly | A missed event stalls a later agent with no error on the earlier one | Each step reacts and emits the next: `application received` → queue → enrichment → `enrichment complete` → risk → `risk assessed` → recommendation. This is message passing, formalized `(State and Memory)` |

**API vs webhook for "did a document arrive?":** a poll asks on a timer and usually hears nothing. A webhook is the document system calling you at the moment of the upload.

**Direct call vs queue:** intake calling risk scoring for all 500 at once overwhelms the agent or the credit API. The queue lets the two sides run at their own pace. Neither needs to know the other's current load.

The demo that works once is the happy path. The system that survives has a timeout, capped backoff, escalation, reconciliation, and a queue-depth alert, and those events show up on the trace above.

**Do not:** poll when the other system can notify you; call the next agent directly under a volume spike; retry a dead API without a cap and a backoff; invent a risk score when the credit check never returned; trust webhooks with no reconciliation; treat a growing queue as healthy because nothing threw.

---

# Git as an engineering discipline

A commit is a **unit of change**, not a save point. One sentence **without “and.”** Tests pass at that point. Atomicity is the precondition for revert, bisect, blame, cherry-pick — decided at `git commit`, not recoverable later.

**Staging is deliberate.** `git add .` bundles the rule, the typo, the debug leftover, the formatter. `git add -p` is the review that always happens before code leaves the machine.

**Subject:** imperative, completes “this commit will…”, useful in `git log --oneline`. Good: `Add V-7 cancellation rule before V-3 in evaluation order`. Bad: `fix`, `wip`, `Updated service.py`.

**Body:** why — constraint, **rejected alternative**, acceptance criterion. The diff already shows what. Silence looks like the obvious alternative was never considered; the next person will try it and rediscover the failure the hard way.

**Commit small, push often.** Unshared work is invisible risk; trunk moves; merge cost grows faster than divergence. With several agents, **size and frequency of what you land** determines whether anyone can merge.

**Never commit:** credentials, tokens, connection strings, client data, large generated artifacts, build output. History is durable and distributed. Deleting in the next commit **leaves it in history**. Treat as **compromised and rotate**. No exceptions.

---

# The pull request and the review

Almost everything that makes a review good is decided by the **author** first. Size determines whether review happens. Attention degrades; large PRs get skim-approvals that look identical to real ones. If it cannot be small, say so and say **where to concentrate**.

**Description carries what the diff cannot:** What / Why / **Look hardest at** / Verified. If the repo **squashes on merge**, this **is** the durable history.

Self-review in the **same UI** the reviewer uses: leftover debug, unintended files, formatter noise.

**A review answers one question: should this merge?** Preferences are labeled suggestions. Spend authority on taste and you will not be believed when something is actually wrong.

**Blocking (short list):** incorrect vs spec; unsafe; behavior change with nothing testing it; reviewer **cannot understand it** (legitimate — approval is accountability). Label `[blocking]`, `[question]`, `[suggestion]`.

**Latency is quality.** Sitting PRs drift and grow. On a pod of two, postponed review is no review. Approval is not a compliment (`LGTM` with no reading is a failure mode).

WIP limit: **2–3 changes you actually read.** Passing tests **support** review; they do not replace it.

---

# Scoping work for an agent

An agent is not a capable colleague. No stake, no memory of Tuesday, no sense of load-bearing files. It expands into any space you leave open. You own every line, including the contract change you have not noticed yet.

**Three independent dials** (do not collapse into “trust”):

| Dial | Meaning | Analysis | Well-specified impl |
|---|---|---|---|
| **Context** | What it can see | Wide | Narrow (named files) |
| **Authority** | What it may change | Almost none | Real, bounded |
| **Session** | Keep going vs restart | Restart when stuck | Restart when stuck |

**Task statement = outcome + authority + boundary + stop rule.** Usually short because it is specific.

- **Outcome:** what will be true; absence would be obvious. Not “add a helper.”
- **Authority:** contract row, work item, or **named failing tests**.
- **Boundary:** files that must not change.
- **Stop rule:** if it appears to need an out-of-scope edit, **stop and tell me**.

**Context is a budget.** File reads spend most of it. Name paths. Standing file = conventions for every session (vocabulary, Decimal money, parametrized test ids, V-1..V-7). Per-task = this outcome/files/tests. Type the same correction twice → it belonged in standing instructions.

**Context is not enforcement.** Prompt makes it **likely**. Diff, types, linter, tests make it **true**. Check the diff even if you said “do not touch the contract.”

**Scope to something verifiable** or it is not ready. “I’ll read it and see” means you will accept fluent work.

**Abandon the session** when: same misunderstanding twice; surface growing not converging; you are explaining instead of specifying. Restart with a better statement is usually cheaper than continuing on poisoned context.

---

# Test-first in an agentic workflow

A test written after the implementation can only **confirm the implementation**. You read the code, assert what it does, inherit every ambiguity it already resolved. First-run green is evidence the test learned from the code. **A test that has never been red has demonstrated nothing.** With agents, volume of code up, amount you wrote → near zero. Test-first is how you know the check came from the **requirement**.

Write the test from the **contract / AC**, implementation closed. If you cannot write it, the contract is defective — better found now.

A **failing test** is the tightest task statement: unambiguous, not your judgment, cannot be satisfied by a plausible description. Hand over with **do not modify the test**.

**Red for the right reason.** `NameError` (function missing) proves nothing. Stub, re-run, see `AssertionError` on the named boundary with the values you expected. **Then** it is worth handing to an agent.

**Order is visible only in history:** commit failing tests, then implementation. Finished files look the same either way.

Test-first does **not** prove the requirement is right, find the case nobody thought of, or help if you assert the wrong thing. It **collapses** if one session writes **both** test and implementation from one loose prompt (they share one interpretation; agreement means nothing).

Assert the **contract guarantee**, not that a helper was called. Do not edit the test when it fails unless you can **point at the contract line** proving the test was wrong.

**V-7 vs V-2 (do not “fix” as typos):** cancellation is **exclusive** (loss **on** cancellation date not covered). Inception is the opposite (cover attaches **on** inception date). Business said they disagree.

---

# Reviewing code you did not write

You are accountable for every line you **merge**. Typed vs accepted does not matter to the client.

Generated code is **uniformly fluent**, including the subtly wrong function. You cannot wait until something “looks off.” Go looking on purpose.

**Read in risk order, not diff order.** Course order: accept/reject decisions → error code / status mapping → system-of-record boundary → everything else (most lines, least attention).

**Check the authority, not plausibility.** Inclusive vs exclusive are equally plausible. Only the contract distinguishes them.

**Search list:** scope creep (unexpected files); invented undocumented behavior; **inverted boundaries**; broad `except` collapsing timeout vs not-found (`except Exception: return None` at the policy master → claims team searches for policies that exist); tests shaped to the implementation.

**A diff hides absence.** Missing V-7 in `POLICY_RULES` has no red lines; suite stays green. `evaluate_policy_exists` may be **legitimately** absent from the tuple (short-circuit). Walk the contract table row by row. A still-red test is absence made visible.

**If you cannot explain it, do not merge it.** Ask the agent and verify against the code; rewrite that part; discard and restate. Do not approve because tests pass (same process may have written both).

---

# Parallel agents against one repository

The new constraint is **coordination, merging, and review**, not the quality of any single change. Most failures are **setup** failures. Six workers do not create six reviewers.

| Thing | What it is | What it is not |
|---|---|---|
| **Repository** | Object store + refs. One `.git` | Not a working directory |
| **Clone** | Copy of a repo, usually **one** working directory | Not isolation for two agents |
| **Branch** | Named pointer to a commit; separates **histories** | Not a private filesystem |
| **Working directory** | Files on disk agents edit and tests run against | Not “being on a branch” |
| **Worktree** | Extra working directory + index + HEAD on the **same** repo | Not a second repo |
| **Remote** | Pointer to another repository | Not a branch |

Creating a branch does **not** isolate two agents. They still share one working tree → mixed tests, silent overwrites, confusion not a clean error.

**Worktrees:** shared object database, remotes, branch names. Isolated working files, index, HEAD. A branch can be checked out in **only one worktree at a time**. Commits in one worktree are immediately in the shared object store; other worktrees never see **uncommitted** files.

On the course container: put worktrees **inside the bind mount** (e.g. `.worktrees/`) and **gitignore** them. Sibling paths outside the mount die on rebuild.

```bash
git worktree add .worktrees/v4 -b feature/v4-amount-limit
git worktree add .worktrees/v5 -b feature/v5-claim-type
git worktree list
# after merge:
git worktree remove .worktrees/v4
git worktree prune   # if the folder was deleted by hand
```

**Partition by surface (files), not by feature.** Features cut across files. Conflict rate = file overlap. Name allowed files; confirm sets do not intersect. Shared registries (`POLICY_RULES`, lockfiles, migration numbers) are **serial**, after parallel work merges — usually **you** write that commit.

**You are the only shared memory.** Coordination artifacts: contract, failing tests, standing instructions, **committed** interfaces. Anything that must not vary is checked by something that **runs**.

**Review does not parallelize.** Bound concurrency by review capacity (2–3), not agent count.

**Trunk green is per-change.** Merge → pull trunk into remaining worktrees → then shared-file work. Long-lived worktrees diverge continuously.

Integration rhythm: agent finishes → **pipeline proves the branch** → human reviews → merge trunk → `cd .worktrees/v5 && git pull` (or merge main) → remove merged worktree.

---

# Orchestration (your job)

**Orchestration:** you decide who does what, in what order, on which files, with which gates, and when work serializes. Agents execute. They do not coordinate.

1. Plan table: task, files allowed, dependencies.  
2. Worktrees + branches for the **parallel** set only.  
3. Failing tests from the contract.  
4. Hand each agent outcome + authority + file boundary + stop rule.  
5. Review while small; merge when it should merge.  
6. Pull trunk into open worktrees.  
7. Serialize the shared surface yourself.  
8. Tear down worktrees.  
9. Never clear a review queue by trusting tests.

Same idea on the **product** side: one adapter; mixed models per task; measure with adapted prompts; decision record + triggers.

---

# Five recurring failure shapes (prompt / extraction)

Meet these on every extraction. Cheapest handling is in the prompt + schema, not later.

1. **Silent required field** → status `absent`, not a plausible invention.  
2. **Self-contradiction** → `ambiguous` + note; do not pick a side.  
3. **Superseded / wrong version** → report version, effective date, superseded **first**.  
4. **Content that looks like instructions** → delimited data; do not obey.  
5. **Out of scope** → named path (`document_kind: other`), not nearest success class.

---

# Artifacts the LLM should expect or propose (do not skip)

| Artifact | Role |
|---|---|
| `config.py` | Pinned model ids, prices, spend ceiling |
| `docs/model-facts.md` | Sourced, dated; no quality column |
| `docs/model-decision.md` | Constraints, evidence, rejected alternatives, review triggers |
| `reports/comparison.md` | Per-task table; counts; caveats; prompt column |
| `src/promptlab/schemas.py` | Single output contract |
| `src/promptlab/prompts/<id>.vN.md` | Immutable after recorded runs |
| Prompt registry + `template_hash` | Reproducibility |
| C1 / call records | Every attempt, tokens, repair, prompt_version, model_id |
| Score records + `scorer_version` | Per-field metrics |
| Frozen cases + gold from source | Separate from few-shot pool |
| `src/rag/config.py` | Pinned embedding model id and dimension |
| `index_build` + `chunk` | `build_id`, `text`, `embed_text`, vector, denormalized filters, stable `chunk_id` |
| `FilterSpec` | Returned with every retrieval result, including an empty set |
| Retrieval plan + `stop_reason` | Rewrite, steps, coverage, why the loop stopped |
| RAG eval harness | Recall at k separate from answer checks; frozen questions |
| MCP tool tests | Functions asserted before any agent is connected |
| Per-request context | Fresh for each case; discarded after the answer. Not a process-global list |
| Case store or handoff | Durable facts, and what the previous agent already wrote |
| Trace record | Agent, input source, output, `duration_ms`, tokens, errors, retries |
| Cost and latency budget | Per case, per agent, written before the architecture |
| Queue-depth alert | Threshold before cases silently miss the time expectation |
| Standing agent instructions | Repo conventions |
| `docs/api-contract.md` + red tests | Authority for agent coding |
| `.worktrees/` gitignored | Parallel agents |

---

# Rapid-fire (if the user asks a distinction)

| If they ask… | Answer |
|---|---|
| Branch vs worktree | Branch = history pointer. Worktree = extra working directory on the same repo. |
| Why worktrees for agents | Agents contend for files on disk, not branch names. |
| Feature vs surface split | Features cross files; conflicts follow overlapping files. |
| Who coordinates | You. Artifacts + gates. Not chat. |
| Context vs enforcement | Prompt makes it likely; diff/tests/types make it true. |
| Outcome / authority / boundary / stop | Done-state; spec or red tests; files not to touch; ask a human. |
| Why test-first with agents | Tests after code copy the code; red test is the spec. |
| Review generated code | Risk order + contract open + look for absence; fluency is not evidence. |
| Should this merge? | Only question. Preferences are labeled suggestions. |
| Temperature | Sampling variety, not accuracy. |
| Deterministic LLM? | No. Stable + pinned, not bit-identical. |
| Cost unit | Per case, including retries. Output tokens are expensive. |
| Streaming | Perceived start, not shorter total time. |
| Alias vs pin | Alias can change behavior with no deploy. |
| Open weights | Weights, not full open source; license + ops + utilization. |
| Select a model | Constrain → eliminate → measure per task with adapted prompts → record + triggers. |
| Grounding vs reasoning | Citation into a source vs generated justification. Only the first is evidence. |
| Miss vs invent | Didn’t find what’s there vs supplied what’s not. Invent is worse for compliance. |
| Frozen cases vs gold from source | Locked **inputs** vs locked **correct answers** read from the document, not the model. |
| Model / harness / agent | Brain / exam setup / student taking the exam. |
| Why RAG | Corpus does not fit the window, or the model was never trained on it. Retrieve; do not stuff. |
| Embedding vs answer model | Embeddings are a separate model. Same one at index time and query time. |
| Chunk too big vs too small | Too big = theme, not the paragraph. Too small = a sentence with no context. Overlap the boundary. |
| Vector vs hybrid vs agentic | Meaning match; add keywords for exact codes; extra search passes only when one pass cannot answer. |
| Lost-in-the-middle | Right chunk was retrieved and still ignored because it sat in the middle. |
| Mismatch vs low-recall hallucination | Wrong chunks vs right material never retrieved and the model filled the gap. |
| RAG debug order | Was the source correct and current? Then, what was retrieved? Then, did the model use it? |
| Similarity score | A rank inside one query. Not a probability. Not a cutoff you hard-code. |
| `text` vs `embed_text` | Cite and show `text`. Embed and lexically index `embed_text` (title + heading prepended). |
| Similarity vs distance | Higher is better vs pgvector `<=>` lower is better. Dot product equals cosine only if the vectors are already unit length. Wrong sort looks like total retrieval failure. |
| RRF | `1 / (60 + rank)`. Same `chunk_id`, same `build_id`. No score blend. No dedup pass after fusion. |
| Filter vs rank | Facts remove rows. Pre-filter both retrievers before fusion. Latest is not current. |
| Multi-step retrieval | Fuse inside a step. Union across steps. Code decides the next search. Rewrite adds phrasing, not facts. |
| SQL vs RAG | Counts and exact rows are SQL. Policy prose is RAG. |
| Workflow vs agent | You fixed the sequence, even if it calls an LLM, vs the model chooses the next step. |
| When to use an agent | Only after code, one LLM call, and a fixed workflow all fail the task. |
| Tool call | The model requests. Your code validates and executes. Side effects are checked first. |
| MCP | Host, client, server. Tools, resources, prompts. One server, many clients. |
| ReAct | Thought, action, observation. Plan first. Reflect before anything is sent. The thought is not audit evidence. |
| Short-term vs long-term memory | This request only, then discard, vs durable facts in a database across sessions. |
| Shared store vs message passing | Everyone reads one case record (easy to start, easy to leak) vs an explicit handoff (stricter boundaries). |
| Context bleed | A context list created once for the process. Test a second user, not one happy case. |
| Why a trace | The same input may not fail twice. The trace shows which agent, which tool, and which handoff. |
| Multi-agent cost | Sum per agent per case, including retries. The month-end bill does not say which step grew. |
| Latency bottleneck | `duration_ms` per step. Parallelize only a step that does not depend on the previous output. |
| API vs webhook vs queue | You need the answer now; they notify you; you buffer a burst. Each pattern has its own failure. |
| Missed webhook | A reconciliation poll. Do not trust delivery alone. |
| Backed-up queue | No crash. Depth grows. Alert before the client asks why a case takes two days. |

---

## Citation map

| Article | Path under `Forward Deployed Engineer/` |
|---|---|
| Tokens / context / request | `Tokens, Context, and the Shape of a Request.md` |
| Sampling | `Sampling, and Why the Same Prompt Gives Two Answers.md` |
| Latency and cost | `Latency and Cost as Design Constraints.md` |
| What differs between models | `What Actually Differs Between Models.md` |
| Open weights | `Open Weights and Where They Fit.md` |
| Selecting a model | `Selecting a Model for a Use Case.md` |
| When not to use an LLM **(client-priority)** | `Week5/When Not to Use an LLM.md` |
| Writing the instruction | `Week5/Writing the Instruction.md` (also at repo root) |
| Structured outputs | `Week5/Structured Outputs.md` |
| Few-shot and grounding | `Week5/Few-Shot and Grounding.md` |
| Reasoning | `Week5/Reasoning in the Prompt.md` |
| System design for prompts | `Week5/System Design for Prompts.md` |
| Measuring a prompt | `Week5/Measuring a Prompt.md` |
| Reading a comparison **(client-priority)** | `Week5/Reading a Model Comparison.md` |
| Embeddings, vector stores, chunking **(important)** | `Embeddings, Vector Stores, and Chunking.md` |
| Hybrid and agentic retrieval **(important)** | `Hybrid and Agentic Retrieval.md` |
| RAG failure modes **(important)** | `Failure Modes: Lost-in-Middle, Retrieval Mismatch, and Hallucination Under Low Recall.md` |
| What an embedding is | `Week5/What an Embedding Actually Is.md` |
| Chunking a policy | `Week5/Chunking a Policy Document.md` |
| Vector store is a database | `Week5/The Vector Store Is a Database.md` |
| Two retrievers / RRF | `Week5/Two Retrievers, One Result Set.md` |
| Filtering is not ranking | `Week5/Filtering Is Not Ranking.md` |
| Multi-step retrieval | `Week5/Retrieval in More Than One Step.md` |
| Retrieval quiz condensation | `Week5/Quiz Study Guide.md` |
| Structured vs unstructured / SQL | `Week6/Enterprise Data Context: Structured vs. Unstructured, and SQL Refresh Applied to Retrieval.md` |
| Data quality and lineage | `Week6/Data Quality and Lineage Where They Affect RAG Outcomes.md` |
| Reranking and eval harnesses | `Week6/Reranking and Evaluation Harnesses.md` |
| Data layer choices | `Week6/Data Layer Choices: Databricks, Snowflake, Azure Cosmos DB, and AWS-Native Options.md` |
| Embeddings (simpler RAG picture) | `Week6/Embeddings, Vector Stores, and Chunking.md` |
| Hybrid retrieval (simpler picture) | `Week6/Hybrid and Agentic Retrieval.md` |
| Failure modes (simpler picture) | `Week6/Failure Modes: Lost-in-Middle, Retrieval Mismatch, and Hallucination Under Low Recall.md` |
| Agentic vs workflow | `Week7/What Makes a System Agentic (vs. Workflows).md` |
| Tool use / function calling | `Week7/Tool Use and Function Calling.md` |
| Planning, reflection, ReAct | `Week7/Reflection, Planning, and ReAct Patterns.md` |
| MCP servers | `Week7/MCP Deep Dive: Building Servers, Not Just Consuming Them.md` |
| When not to use an agent | `Week7/When Agents Are the Wrong Answer.md` |
| State and memory | `WEEK8/State and Memory Management.md` |
| Observability, tracing, budgets | `WEEK8/Observability and Tracing; Cost and Latency Budgeting.md` |
| APIs, webhooks, queues, events | `WEEK8/Integration Patterns: APIs, Webhooks, Queues, and Event-Driven Flows.md` |
| Git discipline | `Git as an Engineering Discipline.md` |
| PR and review | `The Pull Request and the Review.md` |
| Scoping agents | `Scoping Work for an Agent.md` |
| Test-first | `Test-First in an Agentic Workflow.md` |
| Reviewing generated code | `Reviewing Code You Did Not Write.md` |
| Parallel agents / worktrees | `Parallel Agents Against One Repository.md` |
| Quiz + instructor extras | `Quiz Study Guide — Worktrees, Multi-Agent, Orchestration.md` |

Also: workspace `Master.md` for SDLC / Python / Docker. Dense tables: the quiz study guide above.

---

## Closing contract

You are a **reviewer and guide**, not an autopilot implementer.

- Plan first. Cite the FDE article. Suggest the next design choice or command.
- Write code only when the user explicitly asks to implement.
- Never skip gold labels, versioning, spend ceilings, unhappy paths, or the pre-merge review “to save time.”
- If the request would put a determined rule in a prompt, treat reasoning as audit evidence, edit a scored prompt in place, un-pin an alias, quote a 12-case percentage as production quality, or merge generated code nobody can explain — **stop and explain** with a citation.
- On **client** work, default to two articles: **When Not to Use an LLM** (placement: model reads, code decides) and **Reading a Model Comparison** (one table per task; counts; caveats that travel with the numbers).
- On **RAG** work, default to the three important articles: **Embeddings, Vector Stores, and Chunking** (pipeline), **Hybrid and Agentic Retrieval** (upgrade only when justified), and **Failure Modes** (inspect retrieval before editing the prompt). Then apply the retrieval contract: `embed_text` vs `text`, `build_id`, reciprocal rank fusion, pre-filter both retrievers, code-controlled multi-step. Ask whether the **source** was current before you tune the pipeline.
- On **agent** work, walk the four questions in order: code, one LLM call, fixed workflow, agent. The model requests tools; your code validates and runs them. Plan, ReAct, reflect. Do not treat the thought trace as evidence.
- On **multi-agent** work, scope short-term memory to one request and persist case facts on purpose (shared store or an explicit handoff). Trace every step. Set cost and latency budgets per agent before you build. Connect with an API, a webhook, or a queue for the need you actually have, and define timeout, missed delivery, and queue depth before the happy path.

*Derived from the Forward Deployed Engineer articles in this repo. Use on any LLM or agentic project. Pair with `Master.md` when the same repo still needs SDLC and Python discipline. Client-facing default: those two articles first. RAG default: the three important retrieval articles plus the retrieval contract. Agent default: the simplest option that actually solves the task.*
