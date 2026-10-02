# Master.md — SDLC & Python Foundations (LLM Context)

Reusable instructions for **any project**. Attach this file as context for any LLM that helps you code.

Source lessons live under `MODULES/` (Modules 1–12). This file is the operating contract, not a quiz guide and not a Task Tracker-only playbook. Apply the same habits to web apps, CLIs, APIs, scripts, and libraries.

**Pair with `FDE-Master.md`** when the work involves models, prompts, harnesses, agents, measurement, or agentic git/review. Attach **both** files. This file still owns Python, SDLC, tests, Docker, and deploy. `FDE-Master.md` owns LLM/agent rules. When they overlap, apply **both** — do not drop quality gates because the feature uses a model. Cite FDE as `(FDE — article — idea)`.

---

## How the LLM must work

These rules apply on **every** reply unless the user later writes: **implement this**.

1. **Do not implement by default.** Do not write full files, large patches, or “here is the finished code” unless the user clearly asks to implement, apply, or write the change.
2. **Always cite.** Name the module (and lesson idea) the advice comes from. Example: `(Module 6 — Single Responsibility)` or `(Module 8 — catch specific exceptions)`. Prefer Module number + idea. Add a `MODULES/` path when the user needs the exact lesson file. For prompts, models, measurement, or agents, also cite `(FDE — article — idea)` from `FDE-Master.md`.
3. **Always suggest.** Give options, a recommended path, and the next step. Let the user choose.
4. **Ask before coding** when the request is ambiguous, touches more than one file, changes behavior vs structure, or would deploy something that cannot stay running.
5. **Prefer checklists and plans** over diffs. If a snippet is needed to explain an idea, keep it short and label it **example, not a drop-in**.
6. **Stay honest about limits.** If a requirement is vague, untestable, or a deploy target does not match how the app runs, stop and say so `(Module 1, Module 12)`.
7. **Do not skip unhappy paths** to “save time.” Empty input, bad types, missing files, corrupt JSON, and missing config are first-class work `(Module 1 Q4, Module 5, Module 8)`.
8. **Do not invent a second system** next to a working one (two Task types, two persistence formats, two config styles) unless the user is doing a planned migration.
9. **Match the repo’s existing tools.** If they use GitLab + pytest + flake8 + mypy + Docker, do not switch the stack unless asked.
10. **Never commit secrets, `.venv/`, image tarballs, or local data files.** Commit the recipe, not the meal `(Module 4, Module 11)`.

### Suggested reply shape (use this every time)

```
1. Recommendation (what to do, not the finished implementation)
2. Citations (Module N — idea; FDE — article — idea when `FDE-Master.md` applies)
3. Steps the user can run
4. Risks / unhappy paths
5. What to confirm before anyone writes code
```

### When the user says “implement this”

Then you may write code. Still:

- Keep SRP, tests, and unhappy paths.
- Show the quality-gate commands afterward.
- Do not expand scope past what they asked.

### Questions the LLM should ask (when missing)

- Who is the user of this change, and what must happen when input is wrong? `(Module 1)`
- Is this a behavior change or a structure change (tests should still pass)? `(Module 5–6, 9)`
- Which branch strategy does this repo already use? `(Module 2)`
- Must this process stay running (HTTP/service) or is it a one-shot CLI? `(Module 11–12)`
- Where should durable data live if a container is replaced? `(Module 11–12)`
- If an LLM is in the design: is the answer determined (code) or judgment (model)? What are the cost ceiling and latency budget? `(FDE-Master.md)`

---

## Default workflow (any feature)

Follow this order. Skip a phase only if the user says it is already done.

| Step | Phase | Do this | Cite |
|---|---|---|---|
| 1 | Planning | Restate the goal and why it matters. Rough scope only. | Module 1 |
| 2 | Requirements | Write **specific, testable** requirements. Include unhappy paths. | Module 1 |
| 3 | Design | Data shape, flowchart decisions, functions/classes, files, config. Not code yet. | Module 1 |
| 4 | Branch | Pull latest main. Create `feature/...`. Small commits that explain **why**. | Module 2 |
| 5 | Environment | Use a venv. Change deps via `requirements.txt`. Never commit `.venv/`. | Module 4 |
| 6 | Tests | Prefer TDD: Red → Green → Refactor. Happy + boundary + invalid. | Module 5 |
| 7 | Implement | Small units. One job per function. Docstrings. No leftover debug. | Module 6 |
| 8 | Types / validation | Hints + mypy for code; Pydantic (or equivalent) at trust boundaries. | Module 9–10 |
| 9 | Quality | `pytest`, `flake8`, `mypy` locally, then push. Pipeline must go green. | Module 3, 10 |
| 10 | Review | Self-review, then someone else. README answers five questions. | Module 12 |
| 11 | Package / ship | Same image locally, on-prem, or cloud. Data lives **outside** the container. | Module 11–12 |

**Cost of change** `(Module 1)`: Requirements (cheap) → Design → Development → Testing → Production (most expensive). Fix unclear “what” before writing “how.”

```
PLANNING → REQUIREMENTS → DESIGN → DEVELOPMENT → TESTING → DEPLOY/MAINTAIN
                              ▲                                    │
                              └────────── maintain & iterate ──────┘
```

---

# Module 1 — SDLC, requirements, design

**SDLC** is the structured process that turns an idea into working software. Teams fail in predictable ways when they skip steps: they build the wrong thing, build the right thing badly, ship bugs that could have been caught earlier, or ship something nobody can maintain.

| Phase | Question | Produces | LLM should |
|---|---|---|---|
| Planning | What & why? | Rough scope + business case | Restate goal. Not a tech spec. |
| Requirements | What exactly? | Specific, testable statements | Can QA pass/fail this? |
| Design | How will it work? | Architecture, data model, UI, tech | Blueprint, not implementation. |
| Development | Build it | Code that executes the plan | Flag gaps back to design/requirements. |
| Testing | Does it work? | Unit → integration → UAT | Tests against written rules. |
| Deploy & Maintain | Ship / support | Release, patches, improvements | Gate + docs + persistence. |

**Roles (peak involvement)**

| Role | Peaks in |
|---|---|
| Product Manager | Planning, Requirements |
| Business Analyst | Requirements |
| Software Architect | Design |
| Developer | Development |
| QA / Test Engineer | Testing |
| DevOps | Deploy & Maintain |
| Project Manager | Across phases (especially Planning) |

**Waterfall** = locked linear phases; late change is expensive; strength is predictability.  
**Agile** = short sprints, each a mini-SDLC; strength is adaptability; needs discipline.  
Phases are universal. Methodologies only change **how** you move through them.

### What counts as a requirement

A requirement is a **specific, unambiguous** statement of something the system must do, or a constraint it must satisfy.

| Vague (not a requirement) | Specific (testable) |
|---|---|
| Users manage tasks easily | Logged-in user creates a task with title, due date, priority; empty title is rejected and nothing is saved |
| The system should handle errors well | Empty title → show “Title required”; past due date → reject and let the user correct |
| Make it fast | Respond to a user action within 2 seconds under normal load |

### Functional vs non-functional

| | Functional | Non-functional |
|---|---|---|
| Asks | **What** can the user do? | **How well** / under what conditions? |
| Examples | Create, complete, filter, notify | <2s, 99.9% uptime, encrypt passwords, 500 concurrent users |

### Four gathering questions (every feature)

1. **Who** is performing the action?
2. **What** are they trying to accomplish?
3. **What** information goes in, and what comes back?
4. **What** happens when things go wrong (empty input, invalid data, system failure)?

Question 4 is the one beginners skip. A requirement without it is incomplete `(Module 1)`.

### Requirements vs design

- Requirement: “The system must let a user create a record.”
- Design: stored in a list, dict, class, or table? CLI, web form, or API? Step-by-step internal flow?

For a small project, design usually means a **data model table** plus a **flowchart**.

### Flowchart shapes

| Shape | Meaning |
|---|---|
| Oval | Start / End |
| Rectangle | Process / action |
| Diamond | Decision (yes/no) |
| Arrow | Flow |

Unhappy paths are extra diamonds **before** save: empty field, invalid date, duplicate title, missing permission. Yes → error and back to input. No → continue.

### Data model table (fill this before coding)

| Field | Type | Required? | Default | Notes / validation |
|---|---|---|---|---|
| example_title | Text | Yes | — | Non-empty |
| example_status | Enum | Yes | Not Started | Allowed values listed |
| created_at | Timestamp | Yes | set by system | Do not take from user |

If **Required?** is left blank, developers, testers, and error-handling all disagree. Define Yes/No explicitly.

### Mypy vs Pydantic (intro; detail in Modules 9–10)

| | **Mypy** | **Pydantic** |
|---|---|---|
| When | Before run (static) | When an object is created (runtime) |
| Looks at | Source + type hints | JSON, dicts, env, forms, API bodies |
| Failure | Report; Python can still run | `ValidationError`; no instance |
| Typical place | Shared / large code | Trust boundary |

Use both. Mypy does not catch a bad JSON body. Pydantic does not catch a wrong list type in untyped internal code.

---

# Module 2 — Git, PRs, branching

**Git** = the engine (local history, commits, branches).  
**GitHub/GitLab** = the road system (remote host, PRs, review, permissions, issues).

```
Working Directory  --git add-->  Staging Area  --git commit-->  Repository
     (edits)                    (next snapshot)              (history)
                    --git restore / checkout -->
```

### Core commands (suggest these; do not run unless asked)

| Command | Does |
|---|---|
| `git init` | Start a repo in this folder |
| `git clone <url>` | Copy remote + history |
| `git remote add origin <url>` | Connect local repo to host |
| `git status` | Changed / staged / untracked |
| `git add <path>` | Working → staging |
| `git commit -m "..."` | Staging → history |
| `git log --oneline` | History |
| `git push` / `git pull` | Local ↔ remote |
| `git branch` / `git switch` / `git checkout` | List / switch |
| `git merge` | Combine histories |
| `git restore <file>` | Discard **uncommitted** edits (matches last commit) |
| `git reset --soft HEAD~N` | Undo last N commits; keep files (only if not pushed, or user understands) |

**Bad message:** `"fixed stuff"`.  
**Good message:** `"Add validation to reject empty titles"` — explains **why**.

### Feature-branch workflow (necessary steps)

1. Pull latest `main` (or the team default).
2. Create `feature/<short-name>`.
3. Commit related changes often.
4. `git push -u origin <branch>`.
5. Open a PR/MR into `main`.
6. Automated checks run.
7. Human review: approve or request changes.
8. Merge only when checks pass and review is done.
9. Delete the source branch.

Catch problems **on the branch**, before they touch the shared main everyone depends on.

### Repository terms

| Term | Meaning |
|---|---|
| Remote | Hosted copy (not your laptop) |
| Origin | Default remote name |
| Clone | Full local copy of history |
| Fork | Your hosted copy of someone else’s repo |
| README | Root `README.md` — what it is and how to run it |
| PR / MR | Request to merge one branch into another |

Connecting a new project: create empty remote → `git clone` **or** `git remote add origin` → commit locally → `git push`.

### Writing a good PR

Answer:

- **What** changed?
- **Why?** Which requirement or problem? Link the requirements doc when relevant.
- **How tested?** What you personally verified.
- **Special attention?** Edge cases, unsure design, places you want feedback.

Weak: “Added priority feature.”  
Strong: names the behavior, the requirement, and how you tested it.

### Review outcomes

| Outcome | Meaning |
|---|---|
| Approved | Can merge |
| Changes Requested | Must fix before merge |
| Commented | Feedback, not a formal block |

Reviewer comments should be specific and actionable: “This throws if `due_date` is missing — should we handle that?” not “this looks off.”  
Author: a comment on code is not a comment on you.

### Merge options

| Option | Effect |
|---|---|
| Merge commit | Keeps full branch history + merge commit |
| Squash | One clean commit on main |
| Rebase | Replay commits onto main, no merge commit |

If the assignment grades **checkpoint commits**, do **not** squash.

### Branching strategies

A branching strategy is the team’s rule for **where work lives**, **when it merges**, and **what may deploy**. Git does not care. The team does.

| | GitHub Flow | Gitflow | Trunk-Based |
|---|---|---|---|
| Long-lived | `main` (deployable) | `main` + `develop` | `main` / trunk |
| Extra branches | Short `feature/*` | `feature/*`, `release/*`, `hotfix/*` | Hours or none |
| How you ship | Merge `main` → deploy | `develop` → `release` → `main`, back-merge | Merge trunk often |
| Best for | Small–mid SaaS | Versioned / scheduled releases | Strong CI + flags |
| Typical failure | Long “feature” branches pretending to be GitHub Flow | Forgotten back-merges; stale `develop` | Broken trunk; flag debt |

**GitHub Flow steps:** branch from `main` → PR → merge → deploy `main` → delete branch.

**Gitflow branch types**

- `main` — tagged production (`v1.2.0`)
- `develop` — integration, not necessarily released
- `feature/*` — from `develop`, back to `develop`
- `release/*` — freeze/QA, then merge to **`main` and `develop`**
- `hotfix/*` — from `main`, merge to **`main` and `develop`**

**Trunk-based:** tiny changes, tests must pass, merge in hours, hide unfinished **behavior** with flags.

**How to choose**

- One product, frequent deploy, PRs without extra types → **GitHub Flow**
- Numbered versions, old versions still supported → **Gitflow**
- Fast tests + flags + smallest integration delay → **Trunk-based**
- Many teams use GitHub Flow **with** a trunk rule: merge within a day or two

### Feature flags vs feature branches

| | Feature branch | Feature flag |
|---|---|---|
| Where unfinished work lives | Separate Git branch | On `main`, behind a condition |
| How you ship | Merge the branch | Flip the flag on |
| How you unship | Revert the merge | Flip the flag off |
| Risk if left too long | Painful merges | Dead code and nested `if`s |

**Branch** hides unfinished **code**. **Flag** hides unfinished **behavior**. Deploy ≠ release.

When a flag is stable for everyone, **delete the flag and the old path**. A permanent flag becomes a second unofficial branching system inside the code.

Flag source can be an env var, config file, or a flag service. Example idea (not drop-in): `FLAG_DARK_MODE=true` in staging, `false` in production until ready.

### Multiple AI agents in one repo

Default: **GitHub Flow** — one task → one branch → one PR. Humans (or a review agent) are the gate.

1. Do not let two agents share a branch unless one is reviewer-only.
2. Slice work by file/module, not “both implement the whole feature.”
3. Rebase/merge from `main` often on each agent branch.
4. Serialize merges to `main` (smaller PR first, or a merge queue).
5. Never let agents push straight to `main` unless the change is trivial and CI is mandatory.
6. Gitflow + agents: only if humans already use Gitflow; constrain agents to `feature/*` off `develop`.
7. Trunk + agents: only if ownership is disjoint and CI fails bad merges.

---

# Module 3 — CI/CD and pipelines

CI/CD removes “remembering to run the checks.” Checks run on every push, without depending on memory.

| Term | Automatic | Human |
|---|---|---|
| Continuous Integration | Build + test every push | Review / when to merge |
| Continuous Delivery | Build + test + package | **When** to deploy |
| Continuous Deployment | Also auto-deploys | None |

**Delivery** = ready anytime + human gate. **Deployment** = ship automatically. Deployment needs high trust in tests.

### Pipeline anatomy

In GitLab: **`.gitlab-ci.yml` at repo root**. The file is version-controlled; pipeline changes go through the same branch/review as app code. Exact filename and location matter.

- **Pipeline** = full automated run on push/PR
- **Stage** = ordered phase (`build` → `test` → later `deploy`)
- **Job** = one task. Same-stage jobs run **in parallel**; all must pass
- If a job fails, later stages usually **do not run**
- Many teams **block merge** unless the pipeline is green

```
BUILD (install deps) → TEST (pytest ‖ flake8 ‖ mypy) → (later) DEPLOY
```

### YAML meaning

| Key | Meaning |
|---|---|
| `stages:` | Ordered list of stage names |
| Job name | Label you choose (`run_unit_tests`) |
| `stage:` | Which stage this job belongs to |
| `script:` | Shell commands; nonzero exit → job fails |

Example shape (example, not drop-in):

```yaml
stages:
  - build
  - test

install_dependencies:
  stage: build
  script:
    - pip install -r requirements.txt

run_unit_tests:
  stage: test
  script:
    - python -m pytest tests/

check_formatting:
  stage: test
  script:
    - python -m flake8 .

check_types:
  stage: test
  script:
    - python -m mypy .
```

`run_unit_tests` and `check_formatting` do not wait on each other; both must pass.

### Building a first pipeline (steps to suggest)

1. Create `.gitlab-ci.yml` at repo root.
2. Define stages. Start with `build` and `test`. Add `deploy` only after containers exist.
3. First job: install dependencies. If that fails, nothing else can work.
4. Add test / lint / type jobs.
5. Commit and push like any other file. Pipeline runs on that push and every later push.
6. Watch the UI: which stage, which job, pass/fail.

By default a pipeline runs on **every push to any branch**, so feature-branch work is checked before it is near `main`. The PR shows a green check or red X next to human review.

Design habits: cheap / likely-to-fail checks early; one concern per job.

### Local commands the pipeline should match

```text
pip install -r requirements.txt
python -m pytest tests/
python -m flake8 src tests    # or flake8 .
python -m mypy .
```

If local and CI disagree, the first suspect is **uncommitted files** `(Module 3)`.

### Stack traces

A **stack trace / traceback** is the chain of calls active when an error occurred.

**Read Python traces bottom → top.** Last line = **what** failed. Lines above = **how** you got there.

```
Traceback (most recent call last):   ← read last (start of chain)
  File "...", line N, in main
  File "...", line M, in helper      ← line that triggered it
KeyError: 'due_date'                 ← READ THIS FIRST
```

### Failure categories

| Category | Looks like | Typical cause |
|---|---|---|
| Dependency | Fails in `build` | Bad/missing `requirements.txt` |
| Syntax | `SyntaxError` | `:`, brackets, indent |
| Test | `FAILED` in test stage | Code or test wrong |
| Runtime | Traceback while running | Logic, missing data, edge case |
| Environment | Works local, fails CI | Version, env vars, paths, uncommitted files |

### Debug a failed pipeline (7 steps)

1. Which job failed?
2. Open that job’s log; start at the **bottom**.
3. Read the error literally (do not guess “flake8 is broken”).
4. Read the traceback bottom-up.
5. Reproduce locally with the same command.
6. Fix the **root cause**, not a symptom.
7. Push and confirm green.

---

# Module 4 — Python foundations and venv

- Code says **what**. Comments say **why** (business rule, edge case, assumption).
- Bad comment: `# add one` next to `count += 1`.
- Good comment: `# after a successful create, so the next id stays unique`.

Course practice also uses variables, types, `print`, `input`, and f-strings. Prefer clear names (nouns for data) over cleverness.

### Virtual environments

Without a venv, one global package set means project A can break project B.

```text
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install <package>
pip freeze > requirements.txt
pip install -r requirements.txt
deactivate
```

| | `.venv` | `requirements.txt` |
|---|---|---|
| What | Local installed packages | Shared recipe |
| Commit? | **No** | **Yes** |
| Recreates | — | A new venv on any machine / CI |

Add `.venv/` to `.gitignore`. CI builds a **fresh** environment from `requirements.txt` every run. After adding pydantic/mypy/pytest, freeze so the pipeline can install the same set.

---

# Module 5 — Control flow, tests, TDD

### 5-step problem solving (before code)

1. Restate the problem in plain language.
2. List inputs and outputs.
3. Write the steps in English.
4. Mark which steps are `if` and which are `for`/`while`.
5. **Then** write code.

| Comparisons | Logic |
|---|---|
| `==` `!=` `>` `<` `>=` `<=` → `bool` | `and` / `or` / `not` |

### Unit tests (pytest)

| Term | Meaning |
|---|---|
| Unit test | One isolated unit (usually one function) |
| Discovery | Files `test_*.py` or `*_test.py`; functions `test_*` |
| `assert` | False → fail; True → silent |

**Arrange → Act → Assert**

1. Arrange: set up data.
2. Act: call the unit **once**.
3. Assert: check the result.

| Category | Example |
|---|---|
| Happy path | Valid expected input |
| Boundary | Smallest/largest valid |
| Invalid / unexpected | `0`, `99`, empty string |
| Wrong type | `"high"` instead of `int` |

Run from project root, venv active: `pytest` or `python -m pytest tests/`.

**Commit code and tests together.** A feature without tests is incomplete for this course’s quality bar.

### TDD — Red → Green → Refactor

```
RED (failing test) → GREEN (minimum code to pass) → REFACTOR (clean; stay green)
                    └──────── repeat for the next small behavior ────────┘
```

- **Red:** write a test for behavior that does not exist yet. Run it. Failure is success of Red. `NameError` (function missing) or a failed assert both prove the test is real.
- **Green:** smallest code that satisfies **this** test. Incomplete on purpose is fine.
- **Refactor:** improve structure without changing behavior. The passing test is the safety net. Skip refactor if nothing needs cleaning — it is not padding.

| TDD is worth it | Less useful |
|---|---|
| Real branching / edge cases | Throwaway exploration |
| Others depend on the code | Trivial wrappers |
| Bug cost is meaningful | Specs still unclear |

Do not write the whole feature in Green. Add the next Red test for the next piece.

When **refactoring structure** (dict → class): existing tests are the safety net. Change access (`["title"]` → `.title`) so tests still prove the same outcomes. Add new tests only for new rules (e.g. `ValidationError`).

---

# Module 6 — Functions and modular design

Functions exist for more than DRY:

- **Name the behavior** so readers do not re-parse a boolean every time.
- **Single point of change** when the rule changes.
- **A unit you can test** — TDD needs this.

| Part | Role |
|---|---|
| Name | Verb / verb phrase (`is_urgent`, `calculate_...`) |
| Parameters | Inputs |
| Body | Logic when called |
| Return | Output |
| Docstring | Purpose in one short description |

Variables are **nouns**. Functions are **verbs**.

### Single Responsibility Principle (SRP)

One function → **one job**. The name must match. Litmus: describe it without “and”; test it without unrelated side effects.

A function that validates, calculates urgency, prints, and writes a file violates SRP. Split into validate / calculate / format / persist.

Defaults + keyword args at the call site: `create_item("Write report", priority_level=3)`.

| Module 1 idea | Module 6 form |
|---|---|
| Functional requirement | A function that enforces it |
| Flowchart diamond | `if` inside a function |
| Data-model defaults | Default parameters |

---

# Module 7 — Collections and data modeling

Turn the data-model table into structures before inventing classes.

Lists are **zero-indexed** (first item = `0`). Both lists and dicts are mutable.

| | List | Dict |
|---|---|---|
| Access | Index `items[0]` | Key `item["title"]` |
| Best for | Many **similar** items | **One** thing with named fields |

Common pattern: one record = **dict**; many records = **list of dicts**. Nested: `items[0]["title"]`.

| List | Dict |
|---|---|
| `.append(x)` | `d["k"] = v` add/update |
| `.remove(x)` by value | `.pop("k")` |
| `.pop(i)` by index | `"k" in d` |
| `len` / `x in list` | `.keys()` / `.values()` |
| `.sort()` in place | `.get("k", default)` |

| Access | Missing key |
|---|---|
| `d["k"]` | `KeyError` — use when absence is a bug |
| `d.get("k")` | `None` / default — expected absence |

**Mutability trap:** `backup = original` is two names for the **same** list. Use `.copy()` (shallow) or `copy.deepcopy()` for nested independence.

When suggesting a design: say whether the new thing is “one named record” (dict/class) or “many similar records” (list).

---

# Module 8 — File I/O and error handling

In-memory data dies when the process ends, crashes, or the terminal closes. Anything that must survive a restart needs persistence.

| Mode | Meaning |
|---|---|
| `"r"` | Read (file must exist) |
| `"w"` | Write / **overwrite** |
| `"a"` | Append |

Prefer `with open(...) as f:` so the file closes even if an error occurs. Do not rely on a manual `close()`.

### Why not `str(dict)`

`str(task)` looks like a dict but is only text. There is no reliable built-in round-trip back to a real dict.

### JSON

Human-readable and machine-parseable for nested lists/dicts.

| Direction | Function |
|---|---|
| Write | `json.dump(obj, file, indent=2)` |
| Read | `json.load(file)` |

Note JSON uses `false` / `true` / `null`, not Python `False` / `True` / `None`. `json` converts for you.

### Errors — catch specific types

```
try open/read
  success → use data
  FileNotFoundError / JSONDecodeError → empty list + warning
Program continues — no crash
```

Catch `FileNotFoundError`, `JSONDecodeError`, `PermissionError`, `KeyError`. **No bare `except:`.**

Typical split:

| Function | Job |
|---|---|
| `save_*` | Write; handle permission failures |
| `load_*` | Read; missing/corrupt → safe empty result |

Tests: temp files + cleanup (`os.remove` or tmp dirs). Do not leave files that pollute CI.

`print` inside `except` matters more in unattended deploy: it becomes `docker logs` `(Module 8 + 12)`.

---

# Module 9 — Classes, encapsulation, Pydantic

| Term | Meaning |
|---|---|
| Class | Blueprint |
| Object / instance | One specific thing from the blueprint |
| `__init__` | Constructor — initial state |
| `self` | This object |
| Method | Function on the class |
| Encapsulation | Data + behavior together; object protects valid state |
| `_name` | Internal **by convention**, not enforced |

```
Class (blueprint)
  ├── instance A
  ├── instance B
  └── instance C
```

| Dict (M7) | Class (M9) |
|---|---|
| `task["title"]` | `task.title` |
| `is_high_priority(3)` | `task.is_high_priority()` |

A setter/method can reject invalid values (priority not in 1–3) so bad state never lands on the object.

When migrating dict → class: **replace**, do not run two systems. Keep the same list helpers and tests; change item type and attribute access. That is structure, not behavior, if outcomes stay the same.

### Pydantic (runtime validation)

Opt in by inheriting `BaseModel`. Fields are the schema. Success = typed object. Fail = `ValidationError` — **no half instance**.

```text
pip install pydantic
pip freeze > requirements.txt
```

| Input | Typical result (lax) |
|---|---|
| `priority_level="3"` | Coerced to `int` 3 |
| `priority_level="high"` | `ValidationError` |

- No default → field **required**
- `T | None` is still required unless `= None`
- Extra keys: ignore (default) / forbid / allow
- `model_dump()` / `model_dump_json()` out (not `dict(model)` as the recursive API)
- In: constructor, `model_validate`, `model_validate_json`
- `model_construct` skips validation — trusted data only
- `@field_validator` + `@classmethod`: **return** the value or **raise `ValueError`**
- after / before / plain / wrap validators; model-level validator for cross-field rules

Handle errors like Module 8:

```python
try:
    Model(...)
except ValidationError:
    # reject / skip — object never created
```

```
M8: json.load → raw dicts (trust the file)
M9: json.load → Model(**row) each row (skip or reject invalid)
```

| Approach | Validation |
|---|---|
| Dict | None |
| Plain class | Manual where you remember |
| Pydantic | Automatic at **trust boundaries** (input, files, APIs, env) |

V1 → V2 names you may see: `.dict` → `model_dump`; `@validator` → `@field_validator`; `Config` → `ConfigDict`.

---

# Module 10 — Inheritance, polymorphism, mypy

**Problem inheritance solves:** copying an entire class to add one field duplicates bugs.

| Term | Meaning |
|---|---|
| Base / parent / superclass | Original class |
| Subclass / child | Specialized class (`Child(Parent)`) |

Subclass gets parent fields/methods. Only add or change what is different.

### Is-A vs Has-A

| Relationship | Tool |
|---|---|
| RecurringTask **is a** Task | Inheritance |
| TaskList **has** Tasks | **Composition** |

- **Override:** redefine a method; call `super().method()` to reuse parent logic.
- **Polymorphism:** one loop `item.summary()` — each type’s version runs. Avoid `isinstance` trees when a method will do.
- List helpers written for the parent should work on children **unchanged**.
- A child may force a field (e.g. urgent always priority 3) in `__init__`, then `super().__init__(**data)`.

Where a parent is expected, a child is OK. The reverse is not.

### Mypy (static typing)

Python is **dynamically typed**. Type hints are annotations; Python does **not** enforce them at runtime. Mypy is a separate tool that checks hints **without running** the app.

```text
pip install mypy
pip freeze > requirements.txt
python -m mypy .
```

**Gradual typing:** untyped functions are treated more permissively (bodies often skipped). A large untyped repo can look “clean.” Annotate new code going forward. Tighten later (`--disallow-untyped-defs`, `--strict`) — too harsh for legacy on day one.

| Hint | Meaning |
|---|---|
| `list[Item]` | Mutable list of that type (subclasses OK as elements) |
| `-> None` | Returns nothing (`__init__` too) |
| `Item \| None` | Maybe missing — **guard before use** |
| `Iterable[str]` | Wide input if you only loop |
| `Any` | Unchecked — hides bugs |
| `cast` | Lie to mypy; **no runtime check** |

Config lives in **one** file (no merge): `mypy.ini` / `.mypy.ini` / `pyproject.toml` `[tool.mypy]` / `setup.cfg`.

`# type: ignore[code]` is a targeted bypass. As the first line of a file it can ignore the **whole module** — avoid that.

Guard None:

```python
found = find_item(...)
if found is not None:
    found.summary()
```

Empty `[]` often needs an explicit hint. `list[Cat]` is not `list[Animal]` (invariant).

Pipeline quality trio: **pytest** (behavior) + **flake8** (lint) + **mypy** (types). Together with Pydantic: four layers — behavior, style, static types, runtime data.

---

# Module 11 — Containerization

Fix “works on my machine” by shipping **app + runtime + dependencies**.

| | `.venv` | Container |
|---|---|---|
| Isolates | Python packages | App + Python + OS pieces |
| Portability | Tied to the machine | Portable image |
| Python version | Host’s | Image’s |

Not competitors. pip/venv often run **inside** the image during build.

| | VM | Container |
|---|---|---|
| Kernel | Full guest OS | Shares host kernel |
| Size / start | Heavy / slow | Light / faster |

| Docker | Analogy |
|---|---|
| Dockerfile | Recipe (like `.gitlab-ci.yml`) |
| Image | Blueprint (**class**) |
| Container | Running instance (**object**) |

```
Dockerfile --build--> Image --run--> Container
                          \--run--> another Container
```

### Dockerfile instructions

| Instruction | Meaning |
|---|---|
| `FROM python:3.12-slim` | Base image (`-slim` = smaller) |
| `WORKDIR /app` | Working directory inside the container |
| `COPY requirements.txt .` | Copy deps **first** (cache) |
| `RUN pip install --no-cache-dir -r requirements.txt` | Install into the image |
| `COPY . .` | Copy app code |
| `ENV NAME=value` | Default environment variable |
| `CMD ["python", "-m", "package.module"]` | Default start (exec form) |

**Layer cache:** install deps before copying source so a code-only change does not reinstall packages.  
**`.dockerignore`:** `.venv/`, `__pycache__/`, `.git/`, local data, tests if unused at runtime.

### Tags

| Tag | Meaning |
|---|---|
| `app:1.0` | Specific snapshot — prefer this |
| `app:latest` | Moving pointer — easy to confuse |

### Volumes vs ENV (critical)

Container disk is temporary. Without a volume (or other external store), files die when the container is removed.

- **ENV / `-e`** = configure behavior (file path, flags).
- **Volume / `-v host:container`** = **link** the same files. Not a copy. Change one side, the other updates.
- Read config: `os.environ.get("NAME", "default")` — same `.get` idea as dicts.

```
Host ./data/file.json  ◄── mount ──►  /app/data/file.json in container
```

```text
docker build -t app:1.0 .
mkdir -p data
docker run --rm \
  -v "$(pwd)/data:/app/data" \
  -e DATA_FILE=/app/data/file.json \
  app:1.0
```

Second run should load existing data if the app persists correctly.

| Flag / cmd | Purpose |
|---|---|
| `-it` | Interactive (`input()`, TTY) |
| `-d` | Detached / background |
| `-e` | Env var |
| `-v` | Volume mount |
| `--rm` | Remove container on exit (local demos) |
| `--name` | Stable name for logs/stop |
| `--restart unless-stopped` | Restart on crash/reboot unless a human stopped it |
| `docker ps` / `ps -a` | Running / all |
| `docker logs` / `logs -f` / `logs --since 1h` | Output when nobody has a terminal |
| `docker exec -it NAME bash` | Inspect inside |
| `docker stop` / `start` / `rm` | Lifecycle |
| `docker images` / `rmi` | List / remove images |
| `docker save` / `load` | Image as `.tar` for transfer |

A CLI that **prints and exits** will show **Exited** in Docker Desktop. Start will run it once and exit again. That is expected, not a broken image. `--restart unless-stopped` will **loop**. Document that `(Module 11–12)`.

Use the real entrypoint (`python -m package.module`), not a guessed filename.

---

# Module 12 — Release readiness and deploy

A **pre-deployment gate** is a named checkpoint: automated **and** human. All items must be true before deploy.

A pipeline can tell you the code passes rules. It cannot tell you the code is a good idea, well designed, or solves the right problem `(Module 1 + 12)`.

**Done** = works on the author’s machine.  
**Ready to deploy** = done + reviewed by someone else + documented for a stranger + config is explicit.

### Automated (all green)

- `pytest`, `flake8`, `mypy` green in the pipeline (not “probably fine”)
- Image builds
- Container persists data across a restart **because of a volume** (or external storage)

### Human

- Self-review completed
- Code review by someone other than the author
- README answers the five questions
- Env vars and paths documented, not only implemented
- Known limitations stated honestly

An on-prem/cloud deploy is harder to undo than a PR. There is no test stage stopping a bad deploy after you have already shipped.

### Self-review checklist

| Check | Looking for |
|---|---|
| SRP still holds? | Deadline pressure often reintroduces “and” functions |
| Leftover debug / TODOs / commented-out code? | Looks unfinished |
| Commit messages explain why? | Often the only context a future reader has |
| New functions have docstrings? | Reader should not dig into the body |
| Unhappy paths tried? | Empty title, bad priority, missing/corrupt file |
| Would a stranger understand this diff? | Step away before answering |
| Config hardcoded? | Paths/env should be configurable `(Module 11)` |
| Failure output useful? | Logs must help in production, not be a dead end |
| Works only on your machine? | Module 3 environment-difference risk |

### Pre-deploy review questions (beyond a normal feature review)

- Do tests cover failure modes that would matter in production, not only convenient ones?
- Is any Module 11 config hardcoded?
- If this fails in production, will logs explain it?
- Does anything depend on the author’s laptop?

### README — five questions (this order)

1. **What is this?** One or two sentences of purpose, not a feature dump.
2. **How do I run it?** Exact commands: local Python, local Docker, on-prem, cloud if relevant.
3. **How do I run the tests?** Same commands as the pipeline.
4. **What configuration does it need?** Env vars, defaults (e.g. `TASKS_FILE` → `tasks.json`).
5. **What are the known limitations?** Honest gaps (unvalidated dates, CLI exits, no HTTP yet).

Accuracy beats length. A short honest README is more useful than an impressive incomplete one.

### On-prem vs cloud

**On-prem** = infrastructure your organization owns and manages. You own hardware failures, OS patches, 2 AM restarts.

**Cloud** = same container, different landlord. Provider takes hardware (and often host OS/network on managed services). **You still own:** app correctness, container config, what data you store and how it is secured.

| | On-prem | Cloud |
|---|---|---|
| Hardware / failures | Your org | Provider |
| Scaling | Buy/install hardware | Often a config change |
| Cost | High upfront + staff | Usage-based (can spike) |
| Image | Same Module 11 image | Same Module 11 image |

Regulated industries (banking/insurance) often **require** on-prem for data residency, access, and audit — not because it is technically easier.

### Moving an image on-prem

**Option 1 — save / transfer / load**

```text
docker save -o app-1.0.tar app:1.0
scp app-1.0.tar user@server:/opt/app/
ssh user@server
docker load -i /opt/app/app-1.0.tar
```

Save and load must use the **same filename**.

**Option 2 — private registry** (git push/pull for images)

```text
docker tag app:1.0 registry.internal.example.com/app:1.0
docker push registry.internal.example.com/app:1.0
# on server
docker pull registry.internal.example.com/app:1.0
```

### Production-shaped run

```text
docker run -d \
  --name app-prod \
  --restart unless-stopped \
  -v /opt/app/data:/app/data \
  -e DATA_FILE=/app/data/file.json \
  app:1.0
```

Without a volume, restart brings the container back with an **empty** filesystem — data is gone.

Host path for a bind mount is the **left** side. `cat ./data/file.json` or `cat /opt/app/data/file.json`, not a made-up `/data/` path unless that is actually the host folder.

Logs: `docker logs`, `docker logs -f`, `docker logs --since 1h`.

### Cloud compute

| Kind | What you get | Ops burden |
|---|---|---|
| IaaS (VM) | Rented VM; you install Docker | Closest to on-prem |
| CaaS | Provider runs your image (Cloud Run, Fargate, ACI) | Natural after containers |
| PaaS | Push code; platform builds/runs | Least control |

For an already-containerized app, **CaaS** usually fits better than a raw VM.

Registry shape: login → tag with registry address → push → service uses that image + env + persistent storage.

Cloud persistence: managed disk, file share, object storage, or database. Same principle as a volume: **independent of any one container**. Managed container platforms replace instances more often than a single on-prem box.

### Cost (not “cloud is always cheaper”)

| | On-prem | Cloud |
|---|---|---|
| Upfront | High (buy hardware) | Low / none |
| Ongoing | Maintenance + staff | Usage-based; can spike |
| Small / bursty | Hardware often underused | Often more efficient |
| Huge constant scale | Owned HW can win long-term | Bills can exceed owned HW |

Decision drivers: usage pattern, need for cost predictability, regulation, ability to staff ops — not a blanket “cheaper.”

### Example Google path (plan shape)

1. Create a Docker repo in **Artifact Registry**.
2. Authenticate Docker (`gcloud auth configure-docker`).
3. Tag: `REGION-docker.pkg.dev/PROJECT/REPO/app:1.0`.
4. Push.
5. Create a **Cloud Run** service from that image.
6. Set env vars (same idea as `-e`).
7. Persist files in **Cloud Storage** (or similar), not only a container-local file.
8. Verify start; replace the revision; confirm data still exists.

Exact `gcloud` flags are optional in a plan. Numbered steps are enough unless the user asked for a live deploy.

**Cloud Run will not run a CLI that prints and exits.** It needs a process that stays up and serves HTTP. Also needs external storage. Say this explicitly when the app is a one-shot demo `(Module 12)`.

### What stays the same on-prem vs cloud

- Same image
- Same pre-deployment gate
- Same need for env + storage outside the container
- Same already-verified app logic (pytest / flake8 / mypy)

---

## Necessary command sequences (suggest; run only if asked)

### Quality gate (every change)

```text
python -m pytest tests/
python -m flake8 src tests
python -m mypy .
```

### Feature branch → PR

```text
git checkout main
git pull
git checkout -b feature/<name>
# small commits that explain why
git push -u origin feature/<name>
# open PR/MR into main; wait for green pipeline; review; merge; delete branch
```

### Discard accidental uncommitted edits (keep last commit)

```text
git restore README.md
git status
```

Do **not** use `git pull origin main` to undo local edits.

### Local container + persistence proof

```text
docker build -t app:1.0 .
mkdir -p data
docker run --rm -v "$(pwd)/data:/app/data" -e DATA_FILE=/app/data/file.json app:1.0
docker run --rm -v "$(pwd)/data:/app/data" -e DATA_FILE=/app/data/file.json app:1.0
```

Second run should load existing data.

### On-prem-shaped simulation

```text
docker save -o app-1.0.tar app:1.0
docker load -i app-1.0.tar
mkdir -p /tmp/onprem-app/data
docker run -d --name app-prod --restart unless-stopped \
  -v /tmp/onprem-app/data:/app/data \
  -e DATA_FILE=/app/data/file.json \
  app:1.0
docker logs app-prod
docker stop app-prod
docker start app-prod
docker logs app-prod
# confirm host file still has data
docker stop app-prod && docker rm app-prod
```

Do not commit `*.tar`, `data/*.json`, or `.venv/`.

---

## PR / README templates (suggest text; user fills facts)

### PR description (gate + pointers)

```text
## Pre-deployment gate
pytest, flake8, and mypy are green in the pipeline.
Docker image builds; data persists across restart.
Self-review: README.md (Self-review).
Code review: <name / date>.
README answers the five questions.

## How I ran it
<exact commands you actually used>

## Cloud / limits
<plan only or live deploy; say if the process exits>
```

### README skeleton

```text
# <Name>
<one or two sentences: what this is>

## How to run it
### Local (Python)
### Local Docker
### On-prem (if relevant)
### Cloud (if relevant)

## How to test it
<pytest / flake8 / mypy>

## Configuration
<env vars and defaults>

## Known limitations
<honest list>
```

---

## Confused pairs (use these in explanations)

| Pair | Remember | Cite |
|---|---|---|
| Planning vs Requirements | Rough scope vs specific & testable | M1 |
| Functional vs Non-functional | What vs how well | M1 |
| Requirements vs Design | What vs How | M1 |
| Waterfall vs Agile | Locked linear vs short sprints | M1 |
| Git vs GitHub/GitLab | Tool vs hosting/PRs | M2 |
| Delivery vs Deployment | Ready + human gate vs auto-ship | M3 |
| Stage vs Job | Ordered phase vs parallel task | M3 |
| Branch vs Feature flag | Hide code vs hide behavior | M2 |
| `.venv` vs `requirements.txt` | Local meal vs shared recipe | M4 |
| List vs Dict | Many similar vs one named thing | M7 |
| `[ ]` vs `.get()` | KeyError vs default | M7 |
| `"w"` vs `"a"` | Overwrite vs append | M8 |
| Class vs Object | Blueprint vs instance | M9 |
| Mypy vs Pydantic | Static code vs runtime data | M1, M9, M10 |
| Inheritance vs Composition | Is-A vs Has-A | M10 |
| Image vs Container | Class vs object | M11 |
| Venv vs Container | Packages only vs whole runtime | M11 |
| ENV vs Volume | Config vs durable files | M11 |
| Done vs Ready to deploy | Works for author vs reviewed + documented | M12 |
| On-prem vs Cloud | You own hardware vs provider; **same image** | M12 |
| IaaS vs CaaS | You manage VM+Docker vs provider runs the image | M12 |
| `-d` vs `-it` | Detached service vs interactive terminal | M11–12 |

```
SDLC & requirements
  → Git / PRs / branching
    → CI pipeline (build → test)
      → Python + venv
        → if/for + pytest + TDD
          → functions + SRP
            → list[dict]
              → JSON + try/except
                → class + Pydantic
                  → inheritance + polymorphism + mypy
                    → Docker image + volume persistence
                      → pre-deploy gate → on-prem and/or cloud
```

---

## Citation map

| Module | Lesson files under `MODULES/` |
|---|---|
| 1 | `Module 1 SDLC Foundations/` — SDLC; Requirements & Design; Mypy and Pydantic |
| 2 | `Module 2 Git & GitHub/` — Git fundamentals; GitHub PRs; Branching strategies |
| 3 | `Module 3 GitLab Pipelines/` — CI/CD jobs; Debugging stack traces |
| 4 | `Module 4 Python Foundations/` — Variables/comments; Virtual environments |
| 5 | `Module 5 Control Flow, Testing & TDD/` — Conditionals; Unit tests; TDD |
| 6 | `Module 6 Functions & Modular Design/` — Functions, SRP, system design |
| 7 | `Module 7 Collections & Data Modeling/` — Lists, dicts, data structures |
| 8 | `Module 8 File IO & Error Handling/` — Save/load, JSON, exceptions |
| 9 | `Module 9 OOP — Classes, Objects & Encapsulation/` — Classes; Pydantic |
| 10 | `Module 10 OOP — Inheritance & Polymorphism.md/` — Override; mypy |
| 11 | `Module 11 Containerization/` — venv vs containers; Dockerfile, images, run |
| 12 | `Module 12 Deployment & Release Readiness/` — Gate, on-prem, cloud |

Also useful: `MODULES/Modules 1-12 Quiz Study Guide.md` for dense tables. Prefer citing the **module + idea** in answers.

**LLM / agent work:** `FDE-Master.md` (course root) plus articles under `Forward Deployed Engineer/`. Cite **FDE — article — idea**. Do not copy FDE rules into this file; keep one source of truth per domain.

---

## Closing contract

You are a **reviewer and guide**, not an autopilot implementer.

- Plan first. Cite the module. Suggest the next command or design choice.
- Write code only when the user explicitly asks to implement.
- Never skip unhappy paths, tests, or the pre-deployment gate “to save time.”
- If the request would ship the wrong product, fail CI, lose data on restart, or deploy a process that exits, **stop and explain** with a citation.

*Derived from Modules 1–12 in this course. Use on any project. For models, prompts, and agents, attach `FDE-Master.md` as well.*
