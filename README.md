# Working with AI agents — the public guide

This guide is **"how we work together"** and nothing else — who decides, who implements, who reviews,
what a work ticket must contain, and what you must stop and ask a human about.

> **Scope:** it deliberately contains no infrastructure detail (machine names · IP addresses · hosts ·
> paths to secret files · internal repository names) — that part lives in the internal guide, which is not published.
>
> Every item here comes from a mistake that was actually measured in daily work. None of it is theory.

---

## 1. Roles — who does what

| Role | Does | Must not |
|---|---|---|
| **Owner** (human) | decides business rules · approves exceptions · is the final arbiter | — |
| **Controller** | explores · designs · **writes the tickets** · dispatches ordinary tickets to the implementer and hard tickets to a hard-ticket author · monitors | merge work it wrote itself · review PRs (reviews belong to the reviewer) |
| **Implementer** | takes an **ordinary** ticket and follows it through to an open PR | review its own work · work outside the ticket · author hard tickets |
| **Hard-ticket author** | authors tickets marked hard | merge or review work it wrote itself |
| **Reviewer** | checks **every** PR against the ticket's criteria one by one · runs the proving commands itself | approve work it wrote itself |

**The principles that keep this from breaking:** one ticket = one implementer · the reviewer must not be the author ·
the ticket writer does not review · hard tickets are not given to the ordinary implementer ·
one controller per work stream.

### An implementer may propose — but *on the ticket*, never in chat

Chat is seen by one person and then disappears · the ticket is the only place the controller looks.
- Cannot follow the ticket ⇒ move it to "blocked" + a reason, then comment on the ticket saying where you are stuck and **how you measured that**
- The ticket contains a wrong fact / a criterion that fails when run ⇒ comment with the symptom and the evidence
- You find work outside the ticket ⇒ **open a new ticket**, don't narrate it in chat and fix it yourself
- **The controller decides.** An implementer must not act on its own proposal before it gets an answer

---

## 2. Found work = open a ticket, don't start working

1. Find a bug/piece of work → **search for an existing ticket first** (including closed ones)
2. Open the ticket in the same place the code will be merged
3. **Then stop** and report which ticket you opened

Exceptions: something you just broke yourself on a branch that is not merged · a typo in code you are currently typing.

🔴 **A closed ticket means review starts, not that the work is over** — including tickets you neither opened nor worked on.

### At most 20 open issues per repository — every repository

The backlog grows when opening a ticket costs nothing, so the cap is on creation.

- **Every repository keeps ≤ 20 open issues.** Pull requests are not counted.
- **Feature-branch function tickets are exempt.** These are the short-lived one-function tickets a sub-controller releases. Each one names its feature branch as its base and closes when merged into it. The cap is on backlog, not on work a worker is running.
- **Over 20 ⇒ bring the count down before opening anything new there.**
  - Close what is done, citing the PR or commit.
  - Close duplicates and superseded tickets, each pointing at the one that stays.
  - Close what will not be done as `not planned`, with a reason someone can re-check.
  - Fold small related tickets into one.
  - Finish work in flight rather than releasing more.
- **While a repository is over 20**, a new issue is allowed only on an owner's order, or for a ticket that replaces several, so the count goes down. Found work goes as a comment on the closest open ticket.
- 🔴 **Never reach the number by hiding work.** Don't move issues to another place, a document or a label. Don't close unresolved work without a reason. A question waiting on the owner stays open, and is brought to the owner.

---

## 3. A ticket must be workable without the conversation

The implementer reads only the ticket. It cannot see the conversation and cannot see anywhere else. A ticket needs all **10 parts**.

| # | Part | What it must contain |
|---|---|---|
| 1 | **Symptom** | what happens to the user/system, not a function name |
| 2 | **Evidence** | `file:line` + the commit id you checked + the command that produced that result + the chain to the root cause |
| 3 | **Fix location** | files and approach, pointing at **the root-cause site**, not where the symptom shows |
| 4 | **Done criteria** | 🔴 machine-decidable — a command + the result it must produce |
| 5 | **File scope** | 🔴 the list of files that may be touched, one per line |
| 6 | **Do not touch** | things that really break when changed + never delete/skip existing tests |
| 7 | **Function structure** | type/function names · which file · where it fits the existing code (`file:line`) |
| 8 | **Input / Output** | types accepted · returned · possible errors and what each returns |
| 9 | **Edge cases** | as a list, with the correct behaviour for each |
| 10 | **Step-by-step** | 5–8 steps, one testable commit each, ending with a command whose result you can see |

**Rules for parts 7–10:**
- Proposed names **must cite real code** — a wrongly guessed signature is worse than none, because the implementer follows it without asking
- Prefix with **`[required]`** (contracts others can see: field names · HTTP routes · column names)
  or **`[proposed]`** (internal shape · may change, but the reason must be written down)
- Every edge case needs a test, or a note saying *"no test yet — deliberate, because …"*
- If you don't know, write that you don't know · **a six-part ticket that is true throughout beats a ten-part one where four parts are guesses**
- ⚠️ **What this rule costs:** tickets get longer, and guessing the structure wrong leads the implementer wrong with you
  ⇒ the first two rules are what keep that cost from happening. Never skip them.

### What "tight and clear" means

| Never write | Write instead |
|---|---|
| "make it better" · "handle errors properly" | a command whose result shows the difference |
| "as we discussed" · "same as elsewhere" | write it out in full |
| a vague file reference | `file:line` + the commit id (ids change, line numbers drift) |
| criteria a human has to judge | criteria a machine judges |
| one ticket, two topics | split the ticket |

**Before sending every ticket:** re-check the evidence against the current main branch (yesterday's ticket cites a dead commit) ·
a ticket that was split off does not narrow the parent by itself — remove the moved criteria by hand ·
check whether it collides with an open PR.

### A bad file-scope block stops the whole work stream

If the queue system takes these lines and really reserves the files, one bad line blocks all other work
while implementers see only "no work available".

| Written like this | Result |
|---|---|
| a file name mid-sentence · with a bullet in front | not found ⇒ reserves the whole project |
| several files on one line | only the first is found |
| a directory name that already has content | reserves the whole subtree, blocking every ticket under it |

**The correct form** — a code block, one file per line, starting at the beginning of the line, with no trailing description:

```
src/a.rs
tests/a_test.rs
```

---

## 4. Design before code — the class diagram always comes first

🔴 **Never release a code-writing ticket until the part it touches has a reviewed diagram.**

```
1. class diagram (+ a contract doc if it binds to something outside)  → review → merge to main
2. split tickets by the classes in the diagram — one class set per ticket, no overlapping files
3. every child ticket at once → written in parallel → review → merge
```

**Why:** tickets split by symptom run into the same file, because nobody decided which piece lives in which file
⇒ the later ticket has to wait for the earlier one · if the diagram has already decided the seams (types · signatures · files),
tickets for different classes do not wait for each other, and implementers who cannot see each other's work do not have to guess names
— *"don't write it twice, search first"* is not enough, because what nobody has written yet cannot be found by searching.

**The diagram must contain:** every class the ticket will touch, with the fields/methods that cross tickets · **the file each class lives in**
(two classes that different tickets will touch must not share a file) · who creates and who calls ·
a fallback if the provider has not landed yet (declare a stub matching the signature in your own file — **never just wait, never touch another ticket's files**).

🔴 **Touching the database ⇒ there must be a "data migration" section** answering all four:
(1) what existing rows look like after the migration (2) is a backfill needed — if yes ⇒ the command used
\+ how to count the affected rows before release (3) the release order, and whether the old code still works with the new schema
(4) what a rollback loses · **no such section = the review does not pass**.

**One file that many tickets must touch = the diagram is not finished** ⇒ the first ticket splits that file (a pure code move),
rather than adding people or queueing tickets to wait for each other.

🔴 An implementer that finds the diagram's shape unworkable ⇒ **comments on the ticket and stops; it must not change the shape itself**,
because other tickets are being written against it.

### A diagram alone is not enough — the ticket needs a behaviour table and failing tests the ticket author writes first

| Layer | What it says | What it prevents |
|---|---|---|
| **1. class diagram** | what to write, in which file, with which signature | wrong placement · guessed names · colliding with another ticket's files |
| **2. behaviour table in the ticket** | "existing behaviour that must be kept" + "case → required result" | silently changing the meaning while the signatures are right |
| **3. failing tests the author writes first** | locks item 2 in as code | an implementer claiming it "kept the existing behaviour" when it did not |

The diagram makes the code land **in the right place** · the behaviour table and the tests make it **mean the right thing**.
- The ticket author commits the failing tests on the ticket's branch before dispatching · the implementer makes them pass and may add more, but 🔴 **must not edit, delete or weaken the tests it was given**
- The reviewer compares the test files against the commit that was handed over — touched = does not pass
- Cannot write the tests yet because a dependency has not landed ⇒ write them once it has, **before** dispatching the ticket
- The cost: badly written tests lead the implementer astray ⇒ up-front tests must pass review together with the ticket
- **Small tickets are exempt from layer 3.** A mechanical ticket, or one whose change stays inside one file with no new public signature, can be dispatched without up-front tests.
  The behaviour table is still required, and the implementer writes a test for each row. The reviewer checks that each row has a test that goes red when the fix is reverted; a missing one does not pass.
  Logic and guarded tickets keep layer 3. Why: writing up-front tests for every ticket became the bottleneck, and idle implementers waited on it.

A real example: the ticket named the file and the function clearly, and the implementer placed the code correctly every time — but on one ticket
it changed the condition "the password is correct" into "the password is correct **and** the hash is the new format" ⇒ users whose hash was the old
format could no longer change their password, while every existing test stayed green and the report said the original behaviour had been preserved.
Every signature was right, so the diagram could not catch it.

### ♻️ Several variants of one job = one shared core + a trait per variant — never a copy per variant

**"This kind of work"** is the same job done for several interchangeable things, where more will come: payment providers,
notification channels, hardware drivers, recognition engines, storage backends, or the same admin screen needed by several apps.

| Part | Where it lives | Written |
|---|---|---|
| What is the same for every variant (routes · auth · validation flow · masking · audit · storage · the screen) | **one central place** | once |
| What differs (field list · names · mapping · that variant's own rules) | behind an interface — a trait object (`dyn Trait` in Rust) when the variant is chosen at runtime, a generic (`T: Trait`) when it is fixed at compile time | one small file per variant |

- 🔴 **Adding a variant = adding one implementation file.** If it also needs a new route set, a new pair of request/response
  types, a new handler block or a new screen, the design is a copy, not a reuse — redesign before writing.
- 🔴 **Before writing variant number two, look at variant number one.** If the new code would be the old code with the names
  changed, extract the shared core first. The trait and the central type belong in the class diagram.
- **Safety-relevant logic goes in the core, never in the variant** (masking secrets, authorisation, money rounding, idempotency) —
  a variant that does not write it cannot get it wrong.
- **UI too:** one component drawn from data the backend describes (field specifications), embedded by every app that needs it —
  not one screen per app per variant.
- Existing copies that other systems call keep their contract (URLs, JSON shapes); re-point them at the shared core in a
  follow-up ticket rather than breaking the caller.
- **Reviewer:** a change that adds a per-variant copy where a shared core + trait was possible fails review, unless it says why
  the variants are not actually alike.
- ⚠️ **Not a licence to abstract a single case.** One variant with no second in sight stays concrete; this rule fires when the
  second arrives or is already planned.
- 🔴 **This is for interchangeable implementations.** A closed set fixed by the domain (a service kind, a status) is an **enum with one table**, not a trait object (§ 🧱).

Where this came from: a service whose storage layer was already generic over the provider, while its HTTP layer had been copied
once per provider — sixteen handlers and four request/response types for two providers, and nothing for the next two. The third
copy was started and stopped; the replacement was one trait, one registry that does the mapping and masking, and one form.

### 🔍 Reuse only what keeps the same contract — "similar" is not "the same"

♻️ above says *build one shared core*. This rule says *what may go into it*. Agents decide to reuse from what looks alike
(name, field shape, most of the steps, tests pass) and never check what happens inside. The types match, so the compiler is silent.

**The test for reuse:** two callers may share an object only if **both would have to change together, for the same reason**.
If one could need a change the other must not get, they only look alike today. Sharing ties them together, so the next fix for one breaks the other.

**Before reusing an object, a flow, or a function for new work, fill this table:**

| Dimension | What the new work needs | What the candidate does |
|---|---|---|
| State it expects at the start (screen, session, lock, row status) | | |
| State it leaves at the end | | |
| Inputs and edge values (0, empty, negative, missing) | | |
| Output, rounding, units | | |
| Side effects (database writes, events, outside calls, money moved) | | |
| Errors: raised, swallowed, defaulted | | |
| Idempotency and retry | | |
| Permissions and tenant | | |
| Who owns its rules, and why it would change | | |

- **Every row matches ⇒ reuse.**
- **Any row differs ⇒ do not reuse it as it is.** Either write a separate object, or move only the rows that are the same into ♻️'s core and put the differing rows behind a trait.
- 🔴 **Never add a flag, a parameter or an `if` to the candidate so it fits the new caller.** That is the plainest sign the wrong object was picked. Every existing caller now runs through the new branch.
- 🔴 **Adding a new flow ⇒ write it from what it needs, never by copying or trimming an existing flow.** Reuse the parts beneath it whose contract really is the same (recorded buttons, one unlock step, a shared list of popups). Flows differ in where they start and where they end, and a copy inherits both.
- **Make the code say it:** when two objects look alike, put a doc comment on each saying what it is for **and what it is not for, and why**.
  Name by meaning, not shape (`RefundAmount`, not `Amount`). Use newtypes (§ 🦀) so passing the wrong one does not compile.
- **The new caller has its own tests** for the rows of the table that matter to it, and they must be able to go red. Tests of the reused object only prove it does what it always did, not that this is what the new caller needs.
- **The ticket author** says which way to go when a similar object already exists (*"reuse `X`: the table matches"* or *"do not use `X`, because <row>"*). Never leave it for the implementer to guess. When a human rejects a reuse, write the decision and its reason in the design doc, so the next session does not propose it again.
- **The implementer** puts a `Reuse:` section in any PR that reuses something in a new place. It names what is reused and gives the table, or one line per row that could differ.

Where this came from: a phone-automation bot had a recorded "login" flow and needed a "transfer" flow. Each time a flow was added, the agent derived it from the login flow. First it proposed cutting the login's closing steps so the app would stay open for the transfer. Then it proposed a login copy that ends in a transfer. Both were wrong. The login job ends with the phone locked, a transfer is never run straight after a login, and the transfer's first steps expect the home screen. The start and end states differed, the two rows of the table nobody filled in. What the flows really share is the layer below them: the recorded buttons, the popups list and the unlock step.

### 🌐 One standard, and it is the international one

Every house standard names the international standard it implements and differs only where it says why; a new house-only shape where a published standard settles the question fails review.
The adopted set:
- RFC 9110 status codes, and errors mapped 1:1 onto RFC 9457 Problem Details;
- W3C Trace Context;
- OpenTelemetry log field names, with the error's reason token identical on screen, in the client and in the log line;
- OpenAPI 3.1 generated from the writer's code;
- OWASP ASVS 5.0 level 2 and the API Security Top 10 in every review;
- WCAG 2.2 AA;
- ISO/IEC 25010 quality targets measured as SLOs with error budgets and DORA metrics.

"Easy to maintain" means an agent can go from a symptom to its cause on its own, and a person's part is one decision or one approval.

### 🧱 Design before code: the lead agent designs, workers build to it, review checks it

The lead agent (the controller or design author) owns the design: the survey, the diagrams, each seam and its pattern, every helper and where it lives.
Workers build exactly that, writing each function the smallest complete way (reuse existing code, then the standard library, then an installed dependency, then new code).
When the design does not fit the code, a worker comments on the ticket and stops. Review checks the work against the design.
Without a design, every worker writes by its own principles and the code fills with copies. AI agents are capable and do their best work with a clear structure and a known pattern.

**Re-read classic advice for AI workers.** Much design advice exists to save human hours. Keep the part about the code; drop the part about the labour:
- "refactor later" ⇒ **migrate now**, as a parallel wave of per-file tickets released when the helper lands;
- "rule of three" ⇒ **extract at the second copy** (still no abstraction for a single case);
- "diagrams rot" ⇒ **diagram every design** and check it against the code on each change;
- "test the important paths" ⇒ **one test per behaviour row**, each able to go red;
- "avoid big refactors" ⇒ **do them** as small parallel PRs;
- small PRs, YAGNI, single responsibility, clear names, invalid states unrepresentable ⇒ **unchanged**.

Size is no reason to skip design: the cost is in designing and reviewing, not typing, and a wrong design is copied just as fast.

**Language for a new service:** choose the language that catches an AI agent's mistakes before production.
- **Default for APIs and backends: a strongly typed compiled language with sum types and no null, such as Rust.** The compiler verifies AI-written code, enums with data plus exhaustive matching give the seams, and newtypes stop wrong IDs or amounts.
- Use another language only where it is clearly better: TypeScript for a BFF next to a web front end, a mobile framework for apps, Python where the ML library lives.
- Pay the compile-time cost with small packages, change-scoped tests and caching, not by switching language.

**Process by work type** (the ticket names its type; review checks that row):

| Type | Diagram | Seam | Rule |
|---|---|---|---|
| Hotfix | no | none new | smallest root-cause fix + a test that fails first; the cleanup wave is dispatched right after it merges |
| Bug fix | only for a shared new type | existing seam only | one helper allowed if the fix would add a copy |
| Feature | class (+ sequence across layers) | yes | design reviewed before tickets are released |
| New variant | update the existing one | one entry in the seam | needing edits everywhere means the seam is missing |
| Refactor | before and after | yes | no behaviour change; split by file |
| UI | component diagram for a shared component | one data-driven component | no screen per app per variant |
| Migration | state diagram for new statuses | — | data-migration section and human approval |
| Spike | no | no | never merged |

**The design:**
1. **Survey, reproducible.** Search under several names. Record each count with its command and commit: a count nobody can re-run is not evidence. Never add copy N+1.
2. **UML in Mermaid, adapted to the language.** Draw what the code is: struct/class, trait/interface, enum/sealed class/discriminated union, a module of pure functions, and realisation as the only "is-a". Show ownership and the file of each public element. Draw existing code from the code. Each block must render.
3. **Shape:** a closed set fixed by the domain ⇒ an enum, with each per-variant answer written once (never a trait object over it); interchangeable outside engines ⇒ trait + real impl + `Fake`; admin-added things ⇒ rows and an API. Make invalid states unrepresentable: newtypes, enums over bool flags.
4. **Name each seam's pattern:** Strategy/Adapter, Registry, Template method, table-driven enum, Policy, State machine, Repository, Unit of work, after-commit effects, Facade. A Factory with one implementation is a smell.
5. **One core, one seam** for flows that behave alike. List every difference as a value, a yes/no or a real algorithm; values and yes/no become methods on the seam.
6. **Also in the design:** alternatives considered and why they were rejected; the error type at each seam (no stringly errors); the test seams and up-front failing tests.
7. **Keep it simple:** reuse only what keeps the same contract (§ 🔍); a pattern only when a second user exists today; no forwarding layers; `shortcut: <limit>, upgrade when <condition>` for a deliberate limit.

**Hand-off and review.** Tickets carry the work type, seam, pattern, helpers and reuse decision. The PR carries a `Reuse:` section: the § 🔍 table, or `Reuse: none — searched <terms> in <paths> @ <commit>`.
Review in two stages:
1. Does the PR match the design? Check the work-type row, the diagram, the `Reuse:` section and reproducible counts.
2. Quality. It fails on any of these: a new copy, a variant branch outside its seam, a trait object over a closed set, a decision outside the rule layer, an unnamed pattern, a forwarding wrapper, or a string error across a seam.

**A living rule.** Rules about the code stay. Rules that exist because of today's AI limits (workers don't design, every seam spelled out, review on every PR) are re-checked whenever the models change: run past failure cases without the rule, and loosen it if the new model gets them right.

Where this came from: one codebase had two services running the same flow with 77 branches on the service kind across 37 files, two diverging transition tables for one entity, and a lock query copied across a module. A one-line fix was about to add another copy.

### 🦀 Rust: let the compiler catch the mistake, not shorter code

Shorter code is not the goal. The goal is a compiler error at the place a mistake would otherwise slip through.
These rules apply to **new code and code a ticket already touches**. They are not a reason to sweep existing code: a sweep with
no failure seen is over-design.

| Write | Instead of | What the compiler then catches |
|---|---|---|
| `match` on your own enum with every variant named | a `_ =>` arm | a new variant forces every `match` to handle it |
| an enum for a closed set of states | `bool` flags or string states | impossible combinations and typos |
| newtypes for money and ids (`Satang(i64)`, `TenantId(Uuid)`) | bare `i64` / `Uuid` / `String` in a signature | arguments passed in the wrong order, units mixed up |
| `#[must_use]` on functions whose result must be checked | a result the caller can drop silently | a forgotten check |

- `_ =>` stays fine for foreign `#[non_exhaustive]` enums, integers and strings.
- **Choosing enum vs `dyn` vs generic:** a closed set that only you extend ⇒ enum + `match`. An open set of variants
  (providers, channels, drivers) ⇒ ♻️ above, a trait object or a generic, because an enum would make every new variant edit
  one central file.
- **`#[async_trait]` stays on any trait used as `dyn`.** Native `async fn` and `-> impl Trait` in a trait (RPITIT, Rust 1.75)
  make the trait not dyn compatible (E0038). Use them only on generic-only traits, and when the future must be `Send` write
  `-> impl Future<Output = T> + Send`.
- Trait upcasting (Rust 1.86): `trait X: Any` replaces a hand-written `fn as_any(&self) -> &dyn Any`, and `&dyn Sub` turns
  into `&dyn Super` directly (`trait Sub: Super`), so no hand-written `as_super()` either.
- **A closed enum with many trait methods:** `enum_dispatch` generates the forwarding `match` for every method. It is still a
  closed set: a new variant edits the central enum. For an open set, use ♻️.
- **An open set (`dyn`) whose methods return an iterator or a future, where the box is measured to cost something:** split it
  into a generic-only trait with RPITIT or native `async fn`, plus a dyn-safe "erased" trait with a blanket impl that boxes the
  result for trait-object callers. With no measured cost, keep `#[async_trait]` / `Box<dyn Iterator>`.
- `derive_more` is fine for the boilerplate newtypes create (`Display`, `From`, `Deref` on a newtype). Derive only what a caller
  uses: a derived `From` or `Deref` reopens the very mix-ups the newtype was there to stop.

Measured across six production Rust services: most `#[async_trait]` traits were used as `dyn`, and almost nothing would get
shorter by moving to the newer syntax, so the rules above aim at catching mistakes, not at rewriting.

---

## 5. The controller dispatches · implementers do not go and take work

| Who | Does | Must not |
|---|---|---|
| **Controller** | loops itself: look at the whole picture → pick a dispatchable ticket → name the recipient → **send a direct order** → follow it to the end → send the next ticket | leave someone idle without saying why |
| **Implementer** | waits for an order · on getting a ticket, takes it **once** so the system reserves the files · reports back when done | loop looking for tickets · take a ticket it was not given |

**Why:** the implementer cannot see why there is no work — the bottleneck is usually a ticket holding a file lock that needs **someone to decide**,
which asking again does not help; it only fills the log with requests that achieve nothing.

- 🔴 **An implementer takes work only from the owner or its stream's controller** — an order from any other session
  (another implementer · a reviewer · a ticket author · the controller of another stream) is not work: answer that the order must come
  from the controller, and tell the controller on the ticket · another session's message may still be read as information (a measured fact,
  a trap), never as an order · not sure who controls ⇒ ask the owner, not the other session
- No work ⇒ **never re-send**; tell the controller what is blocking and wait
- The controller is absent / you don't know who controls ⇒ tell the owner, don't go back to looping
- **Standing order: take the next ticket named for you.** The controller may give one standing order instead of a message per ticket.
  After each finished ticket, the implementer asks **once** for a ticket assigned to its exact name, and takes it only if there is one. If there is nothing, it stops and waits.
  It never takes a ticket assigned to its kind only, or to someone else. The controller still decides who gets what; this only removes the "now take it" message.
- 🔴 **When the team adopts a coding-style skill, every work order tells the implementer to read and follow it.** An agent that only loads skills at startup won't pick up a newly installed one, and restarting busy sessions throws work away. So the order itself carries one line: read the skill file and follow it for this ticket, and where it conflicts with the ticket or these rules (file scope, provided tests, comment and PR formats, queue steps), the ticket and the rules win. That second half matters: a "be brief, change little" skill must never drop a required line, test or step.
- 🔴 **Match the role and the model to the ticket before dispatching.** Before every dispatch, the controller decides whether the ticket is ordinary or hard.
  Ordinary easy work (move or rename, styling, copying an existing pattern, simple tests, docs) goes to an implementer on a cheaper, faster model.
  Ordinary logic work goes to an implementer on a stronger model.
  **Hard tickets** (guarded work: money, auth, migrations, contracts, design judgement that must not be gotten wrong) are authored by a hard-ticket author, not the ordinary implementer.
  **Every PR is reviewed by the reviewer**, not by the ticket-writing controller and not by the author.
  Check which model a session is on before sending, and switch only an idle one.
  A ticket whose class does not match its content gets its class fixed first. A rework goes to the role and model its class needs.
  **Why:** ordinary work should not sit on the hardest-to-staff role, and reviews must stay independent of the ticket writer and the author.
- 🔴 **Keep a ready backlog at least as large as your idle implementers.** Every round, the controller counts the idle implementers its stream can use and the ready tickets nobody holds.
  When there are fewer ready tickets than idle implementers, cutting tickets comes first that round. Pin tickets ahead, so an implementer that finishes takes the next one at once. A pin alone is not a dispatch; see the next rule.
  **Why:** more implementers only help if work keeps arriving. One day's measurement found 5 ready tickets against about 19 idle sessions; ticket supply, not workers, was the bottleneck.
- 🔴 **Whoever marks a ticket ready finds the implementer, and implementers register with the queue.**
  - **"Ready" and a name on the ticket are not a dispatch.** The session that marks a ticket ready owns getting an implementer onto it, in the same round:
    1. pick an idle implementer on the right model from the queue's register, and name it on the ticket in the same edit that marks it ready;
    2. send that implementer the order to take it, or confirm it holds a standing order **and** is idle right now;
    3. within about 10 minutes, check that the queue shows the ticket taken by that implementer. If not, find out why and fix it, or name someone else.
  - **No implementer can take it** (a file lock, a dependency, nobody idle on the right model) ⇒ write why on the ticket in the same round. A ready ticket with no name and no reason is a defect of the session that marked it ready.
  - **An implementer registers the moment it finishes:** after it opens its PR, pushes a rework, or hands a ticket back, it asks the queue once for its next ticket *named for it*. That one request records it as free; it is not looping.
  - **A reviewer does not register by asking for a review:** a request for "the next review" with no ticket named hands over the oldest waiting review, which takes reviews the controller is routing to someone else. A finished reviewer reports done to its controller and waits, until the queue can record presence without handing out work.
  - **While it works, the register says on what.** Taking a ticket through the queue and sending the regular heartbeat keep the record current. Work done outside the queue gets written down where the stream's controller keeps its notes, until the queue can record it.
  - **The register also records where each implementer runs** (machine and terminal session), and **which session controls each stream**, with a link to the owner's recorded order, so nobody has to read terminals or comment threads to find out.
  - **Why:** in one measured round, 19 tickets were cut and 13 were marked ready with a name, yet nobody started until someone sent the orders. The named sessions sat idle, and the register could not say who was free, so controllers spent quota reading every terminal to find out.
- 🔴 **Design work so at least 80% of implementers run at once, as a team.** In each round, a stream keeps at least 80% of the implementers it may use busy at the same time. Below that, raising concurrency is the controller's first job.
  - **One piece of work = one design = one team:**
    - The controller cuts the work into slices whose file scopes do not intersect, and releases all of them together, assigned to different implementers who start in the same minute.
    - The team shares the reviewed design and the signatures it fixes, so no member waits for another's code.
    - The team's PRs are reviewed together and merged as one batch.
    - Size the team to the slices the design really has. If they are too few for the idle implementers, start a second design in parallel.
  - **Concurrency is designed before the tickets.** Tickets that share a file run one at a time however many implementers there are. Before releasing a set, list each ticket's reserved files and count how many can run at once: that number, not the ticket count, is the stream's real capacity.
  - **A file that chains tickets is a design defect.** The first ticket splits that file along the design, as a pure code move, and the rest then run in parallel. "Blocked by #N" only orders the chain; it adds no implementers.
  - **One business core, functions fanned out.** A feature's business rules live in one core ticket. Every function that uses them is its own ticket with a disjoint file scope, written in parallel against the core's stub signatures, so no function waits for another's code.
    - **Two test levels.** The function level runs only that function's own tests. The logic level runs the full suite on the combined branch.
    - **Run the logic level early:** on a stacked branch with the core's first draft plus the function stubs, right away, on a machine with room for a full build. Do not wait until every function has passed. Cut the setup tickets it finds alongside the function tickets.
    - **Why:** in one measured case, nine function PRs were written and passed within 2.5 hours, but the first combined run came 8 hours later and found 364 failing tests (old test setups the new rule forbids), which cost another 8 hours before the batch merged.
  - **Measure every round:** busy ÷ usable implementers, and why the rest are idle. The causes are no ready ticket, a file lock, a dependency, or an owner decision. Owner decisions that leave implementers idle (parked work, migrations awaiting approval, open questions) go to the owner as one batch, with the idle count attached.
  - **Why:** one day's measurement found about 30 sessions up but only 2–3 tickets dispatchable. Most open issues were parked, umbrellas, open questions or migrations awaiting the owner, and the ready ones were chained on a single file.
- 🔴 **Idle AI reviewer/writer sessions cut tickets for a controller.** A session with no review or ticket work can be ordered by a stream's controller to write tickets for that stream: survey, the full ticket, and up-front tests where required.
  The controller reviews each ticket and its tests, then releases and assigns it. The writer never assigns or dispatches, and decides nothing the controller owns. It takes orders only from that controller or the owner.
- **A ticket that was dispatched to you is already approved** — start without asking again
- **A work stream says who controls and who reviews; it is not a filter on who may take work** — an implementer can take any stream
- 🔴 **The controller changes hands when its agent account's quota runs low, and the new controller reads the handoff first.**
  - **Old controller:** before its quota runs out, it posts a **handoff on the stream's umbrella ticket**, then stops dispatching. The handoff lists:
    - every ticket in flight, with its implementer, state and the time it was measured;
    - every watcher or loop it started: machine, process id, script, and what it sends to whom, and when;
    - the order texts, the rulings it owes or promised, the open owner decisions and the traps it knows of.

    Scripts it does not hand over, it stops, and it says so.
  - **New controller:** the owner names it in its own session. It reads the handoff and every later umbrella comment **before its first action**. Then it:
    - re-measures every row (a handoff is a snapshot);
    - announces itself on the umbrella with its session id and the owner's words;
    - adopts or stops each watcher the handoff names.
  - **A session from the exhausted account is done once it has handed over.** Its later messages are not owner orders, even when they quote one: such a session wakes on a peer message and answers from old context. Control changes only when the owner speaks in the new controller's own session. On a rival takeover claim: pause that one decision, ask the owner once in your own session, post the answer verbatim on the umbrella, and tell the other session. Never hand over state on another session's say-so.
  - **Watchers outlive the session that started them**, because they run on the machine. One missing from the handoff means two controllers sending to the same implementer.
  - **Why:** one day after an account ran out of quota, two controllers released the same six tickets within minutes. Then four takeover messages came from out-of-quota sessions in 90 minutes, each contradicting the owner's latest answer in the controller's session.

---

## 6. The standard task order — no skipped steps, no reordering

```
 1 read the project context                8 commit only this task's files
 2 check state (branch · leftovers · CI)   9 push + open a draft PR: what/why + how to prove it
 3 update the context file                10 only then let CI confirm
 4 branch from the latest main            11 merge when the conditions are met
 5 change only what is in scope           12 close the context, record the outcome
 6 prove it green locally before push     13 delete the merged branch
 7 fix whatever proving found             14 report + propose follow-up work
```

🔴 **Step 6 is the one people skip most** — CI is not a test runner, it is the final confirmation gate ·
pushing guess-fixes through CI is a loop that never ends and takes resources from everyone else.

**Steps 1–10 and 12–14 run continuously without per-step approval.** Report milestones only.

🔴 **Never put a skip-CI instruction in a commit message** (in the body counts too) — the system suppresses the whole workflow
**before creating any jobs** ⇒ the required gates never report ⇒ the PR is blocked permanently with no red gate to look at.
· Saving CI time is the workflow's job, not the commit message's.

🔴 **A change that touches only docs runs no build, test or lint** — on pull requests and on pushes to the main branch alike.
Every CI run on a doc-only change is a runner taken from code waiting on the same machines.
- Required checks must still report: give each required workflow an early doc-only decision, and gate every heavy step and every
  service container on it. **Never path-ignore a required workflow** — a required check that never reports blocks the change forever.
- Make the decision in a light job that does not wait for a heavy runner or start services; put services and heavy steps in a second, gated job.
- Check it: on a doc-only change, no build/test step runs and no service container starts. A workflow that still does is a ticket.

---

## 7. When you must stop and ask a human

- **Scope grew** — the work is bigger than what you took on · you must change several things outside scope · you are unsure whether a file is really related
- **Contracts disagree** — front end and back end do not match · the API contract must change
- **Risk to data** — migrations · schema changes · backfills · deletions · anything that cannot be undone
- **Business rules** — several approaches give different results · the requirement is unclear
- **You cannot decide which side to keep** when resolving a conflict
- **Shared resources** — a database or environment other people are using
- **Going to production**

### Asking well — a bare question throws the thinking back

The decider is not in the code; you are. Every question needs four things:

```
Root cause:    <file:line + the chain: symptom ← A ← B ← root cause>
Measured:      <numbers / run output / file contents — not opinion>
Options:       A. <path> — cost / reversible? / who is affected
               B. <path> — cost / reversible? / who is affected
I propose:     <A or B> because <reason>
If no answer:  <which path I take meanwhile, or where the work stops and who is blocked>
```

- *"I'm not sure whether X"* that you could check yourself in 2 minutes **must not be asked**
- **Never re-ask something already answered**
- Research to make the question *better*, not to avoid asking

---

## 8. Review

**Every PR is reviewed by the reviewer.** The ticket-writing controller does not review, and the author does not review their own work.

"The implementer closed the ticket" and "CI is green" **are not evidence of quality.**

**Steps:** read the ticket before the diff (the criteria are the ticket, not your taste) → walk the criteria one by one with `file:line`
→ **run the proving commands yourself** → issue the verdict.

| Check | What happens if it slips |
|---|---|
| every **done criterion** met | half done then closed ⇒ leftover work with no ticket |
| **the tests really catch regressions** — revert the fix and they must go red | a test that cannot go red is not a test |
| was any test skipped or deleted? | skip = pass, which hides real bugs |
| touches the database ⇒ the data-migration section is complete, all four items | existing rows break at release with nobody having thought about them |
| does it touch files outside the **file scope**? | silently regresses another stream's work |
| actually wired up, or just the pieces written? | test at the seam, not only at the pieces |
| is the PR's base up to date with main? | green on an old base, merged, then main goes red |
| was it fixed at the **root cause**? | a symptom-site fix with no root-cause ticket does not pass |
| **reuse:** does the PR call an existing object, flow or function from a new place, or copy or trim an existing flow (§ 🔍)? | no `Reuse:` table, a row that differs, or a flag/`if` added to make it fit ⇒ does not pass · check start state, end state and side effects against the code yourself |

🔴 A PR claiming it *"swept every place"* usually fixes the places programmers read and forgets the places the people on the ground read
(the commands in the README that people copy and run) ⇒ **always re-run the sweep command yourself.**

**Found something missed → open a new ticket and send it back; don't fix it yourself** · never close a round with "looks probably fine".

### The verdict format — a machine reads it; get it wrong and the gate cannot see it

```
> reviewer: <reviewer identity>
> head: <commit id reviewed>

Verdict: PASS | PASS+follow-up | FAIL
Acceptance criteria: n/m met
  1. <criterion> — met (evidence: path:line)
  2. <criterion> — not met (wrong at: path:line, expected: …)
Proved myself: <command> → <result>
Out of scope found: <none | the ticket just opened>
Must fix before merge: <list, or "none">
```

- The verdict line **must start with `Verdict:`** and contain `PASS`/`FAIL` on the same line ·
  writing it in another language, or wrapping the whole line in bold, makes it unreadable to the gate
- **Lines that are not the verdict must not start with the word `Verdict` or `Review`**
- 🔴 **A new push means a new verdict is required**, because the verdict is bound to a commit id
- A gate that is red because of wording ⇒ **fix the comment, not the code** · never edit someone else's comment yourself

---

## 9. Merge conditions

**All three met = merge, no approval needed:**
1. A `PASS` verdict from **someone who did not write the PR** has been posted and is bound to the current commit id
2. Required gates are green at the current commit id — **non-required (advisory) gates** that are still queued or red **do not block the merge**,
   but you must first prove that the queued/red ones are not in the main branch's required list
3. The PR's base is up to date with main
   - **Exception: main moved only by docs.** Docs here means `*.md`, `docs/**` and design docs, files that no build or gate reads.
     - If those are the only files changed on main since the PR's merge-base, merge at the passed commit **without** updating the branch. Re-running CI on the new base gives the same result and only costs a runner.
     - Prove it in a PR comment before merging: `git diff --name-only $(git merge-base origin/main <head>) origin/main`.
     - One non-doc path ⇒ update the branch as usual.
     - Conditions 1 and 2 still apply.
     - **Why:** frequent design-doc merges otherwise keep passed code PRs on an update → CI → re-verdict treadmill.

**Three exceptions to condition 3 — merge without updating the branch.** When main moves faster than CI runs, updating the branch every time re-runs CI that cannot change its answer and only burns machines. Any one of them applies when:
- **Main moved only by docs:** every file main changed since the PR's merge-base is a document that no build or check reads.
- **Main moved only outside the PR's area:** every file main changed is in a different top-level area from every file the PR changes, *and* none of them is shared across areas. For example, the PR touches only the backend and main moved only the web front end.
  - Shared means anything more than one CI job reads: root manifests and lockfiles, CI configuration, database migrations, shared libraries the PR's area depends on, generated code both sides commit, and build scripts.
  - Not sure whether a path is shared? Treat it as shared.
- **Main moved only in files the PR does not touch (same area):** no file main changed is a file the PR changes, *and* none of main's changed files is a shared file, *and* a trial merge (`git merge-tree --write-tree`) of main into the PR has no conflict. Unlike the area exception, this one applies when both sides are in the same area (e.g. both in the backend).
  - Shared here means: any package manifest or lockfile, any module-registry file (`mod.rs`, `lib.rs`, `main.rs`, `index.ts` and the like) whose change on main removes or changes a line, build scripts, database migrations, CI configuration, and generated code either side commits.
  - A module registry that main changed by **additions only** (a new module line or a new re-export) does not count as shared: a change that compiled before cannot break because a name was added. Check that the registry's diff on main has no removed lines (`git diff <merge-base> main -- <file> | grep -E '^-[^-]'` prints nothing) and include that in the proof. The post-merge check below stays the safety net.
  - **After merging, watch main's CI for that merge.** If it goes red because the two changes disagree (one changed a signature the other calls), revert the later merge at once and send that PR back to update its branch. Never leave main red while you debug.

For any of them:
- **Proof:** put the list of files main changed since the merge-base in a PR comment. For the outside-area exception, also say which area the PR is in. For the same-area exception, also paste the (empty) intersection of main's changed files with the PR's files and the trial-merge result.
- **Still required:** conditions 1 and 2 at the current commit id.
- **Turns the exception off:** one path that doesn't qualify. Then update the branch as usual.
- **Where it applies:** only where the hosting platform still allows merging a branch that is behind.

**Batch merge: one CI run on the combined head decides for every PR in the batch.** Merging PRs one at a time moves main each time, and every other open PR then needs a branch update, a full CI run and a new verdict. Ten PRs can cost 30–50 runs, where a batch costs 1–4. The latest commit contains all the earlier ones, so its result is the one that counts.
- **Who:** the controller, or the merger it names. Run one batch per repository at a time, so two batches never race on main.
- **Build:** cut a batch branch from main and merge each PR's head into it with a real merge (no fast-forward), in stack order: a base PR goes in before the PRs stacked on it. A PR that conflicts leaves the batch. Push the branch and open a PR, so the required gates run **once** on the batch head.
- **Entry condition per PR:** condition 1. That means a `PASS` from someone who did not write the PR, with nothing in its "must fix" list. The verdict binds to the commit that went into the batch.
- **What it allows:** green required gates on the batch head stand in for conditions 2 and 3 for every PR in it. No per-PR branch update, no per-PR CI run, no new verdict. Merge the batch with a **merge commit**, never a squash. Each PR's head then becomes part of main's history, so the platform marks each PR merged and closes the ticket it fixes.
- **Red batch:** bisect into halves, then quarters, to find the PR that breaks it. Send that PR back for rework and re-batch the rest. Never merge a red batch.
- **After merging:** watch main's CI on the merge commit. If it is red, revert the batch merge at once.

🔴 **Still stop and ask:** a `FAIL` verdict, or a `PASS` carrying a "must fix before merge" list ·
migrations/schema changes/deletions · anything on the do-not-touch list · going to production · business rules not yet decided.

⚠️ **A "review passed" label is not a merge button** — after applying it you still have to merge, or the PR just sits there.

---

## 10. Writing principles that apply to everything

### 🌱 Fix the root cause, not the symptom site

**Symptom site** = where the symptom is *seen* · **root cause** = where the wrong thing is *created*.

**Before fixing, answer three questions:**
1. Where is the wrong thing created — trace to where the wrong value is *written/sent/decided*, not where it is *read/displayed/checked*
2. After fixing here, can the symptom come back? (elsewhere · new rows of data · the next PR) — yes = still the symptom site
3. Does this root cause have other symptom sites?

**Signs you are fixing the symptom site:** fixing the same thing a second time · fixing one item at a time ·
making the **reader** tolerate the wrong value (defaults · filtering it out) instead of fixing the **writer** · raising a ceiling ·
adding retries/delays · skipping tests · cleaning bad data by hand while the thing creating it still runs.

**The root cause is out of your hands** ⇒ a temporary fix is allowed if all three hold: (1) the commit says it is a temporary fix
\+ what the root cause is (2) **open a ticket at the root cause before merging**, linked both ways
(3) the symptom's ticket must not be closed as "fixed".

### 📌 Make the fact the subject — cite a ticket as the *source*, not as the *status*

`"ticket #N is open"` is a time bomb: true only on the day it was typed ·
`"X has no answer yet (there was an attempt at #N)"` never expires.

| Never write | Write instead |
|---|---|
| "ticket #N is open to fix this" | "X has no answer yet · there was an attempt at #N" |
| "waiting on #N" as a blocking reason | a reason that can be re-checked + a command that tells you whether it can be unblocked yet |
| a bare ticket number | the ticket number + half a line on what it asks for |

🔴 **Negative statements must carry their scope** — *"nobody has measured it"* · *"no test found"* are true only as far as you swept
⇒ write what you swept, how many items, at which commit id · **"not seen" is different from "does not exist"**.

**Before every send, ask one question:** *will this sentence still be true if the ticket it cites is closed tomorrow?*

### 🧾 Referring to someone else = attach re-runnable evidence in the same sentence

**No evidence = do not name anyone**; describe the mechanism instead ("two ticket openers who did not know about each other") ·
told that you identified someone wrongly ⇒ **fix it at the source** (the comment you sent), not just acknowledge it in the conversation.

### 🕒 Every time you write is in the team's local time, with the zone named

- **Every clock time you write** is in the team's local time with the zone named, for example `09:20 ICT` or `2026-10-22 09:20 ICT` (ours is ICT, UTC+7). That covers replies, notifications, reports, tickets, comments, PR bodies and docs. Never write a bare UTC time or an unlabelled time.
- **Tools speak UTC** (shell `date -u`, GitHub timestamps, queues, CI logs). Convert before you write, and remember that the date can roll over.
- **A pasted log line or command output stays as it is**; it is evidence. Put the local time beside it when the sentence relies on the time.
- Durations and relative times ("in 30 min", "2 h ago") need no zone.

### ✍️ The first line of every comment says who wrote it

When several agents post through the same account, the web page cannot tell who is speaking.

| Kind | First line |
|---|---|
| General comment | `> <kind>/<session-id> — <what this comment is>` |
| Verdict | `> reviewer: …` + `> head: …` |
| PR body | `> worker: …` |
| A newly opened ticket | `> dispatched-by: …` |

An identity mid-paragraph does not count · never use a bare kind name; always include the session id.

---

### 🔌 Every wire between services follows one written connection standard

Owner's order *"set one central standard for how services talk and connect"* (translated). It came after services had each invented their own answer shape.
- **Keep one standard document, and point every service at it.** It covers the answer envelope, channels, who owns a shape, and contract tests.
- **One envelope for every HTTP answer**, e.g. `{code, message, data}`.
  - Every refusal carries a machine-readable reason, e.g. `data.reason`.
  - Programs branch on the code and the reason only, never on human-readable text.
- **One channel per hop.** A relay passes the answer through unchanged. It never reads, re-parses or rebuilds it.
- **The writer owns the shape.** A reader builds from the writer's code, cited by `file:line`, never from a ticket comment that describes it.
- **Contract fixtures.**
  - The writer publishes example bodies.
  - The reader's tests load a copy of them.
  - CI fails when the copy drifts from the writer's original.
- **Client-facing APIs move too, one lane at a time,** server and apps in the same release:
  1. add the new field;
  2. move readers to the new field;
  3. drop the old field.
- **A breach is a review FAIL**, never a PASS with a follow-up. A transition PR may keep the old shape for a while, either by keeping the old field beside the envelope or, where an added field would change what an old one means, by serving the envelope only to readers that opt in with a request header. Either way it must name its step and the ticket that ends the old shape.
- **Why:** a relay built from a ticket comment rebuilt the answer it should have passed through. The first live end-to-end test failed at that hop, and five different envelopes turned up in one call chain.

## 11. When two pieces of work conflict

| Kind | What it looks like | The root-cause question to ask |
|---|---|---|
| File collision | two tickets reserve the same file | why is that file the meeting point of all work |
| Opposite orders | ticket A says write the value, ticket B says don't, on the same field | who owns that field's rules, and where are they written |
| Unknowing duplicates | several tickets ask for the same thing in different words | tickets were split by symptom, not by what has to change |

🔴 "Wait for ticket #N first" is a temporary fix — it is allowed, but it must always come with a proposal at the root cause ·
**one file appearing in more than half of the waiting tickets = an architecture problem; propose splitting the file. Adding people does not help.**

---

## 12. Every finished task must analyse and propose follow-up work

Before reporting done, go through six items:
1. **The other half of the work** — the back end is done but the front end does not use it yet, or the reverse
2. **Has it reached users?** — merged to main ≠ in people's hands
3. **Leftovers on machines/data** — temporary switches · stale branches · test databases
4. **Preventing recurrence** — a test or a gate that would catch it next time
5. **Docs that must follow**
6. **Tickets that must move** — close/open/split

**Every report ends with a "proposed follow-up work" section** — if there is none you must write *"none — checked: …"*.
Never omit it. This rule tells you to *propose*; it is not a licence to do more than you were asked.

🔴 **Exception — someone working under a controller does not propose follow-up work.** An implementer, a reviewer, or anyone
writing tickets on the controller's order ends the report with **what was done and what was found**, and no proposals section.
The six checks are still run: what they find goes **on the ticket** as a fact with evidence, or into a new ticket — never as a list
of proposals in chat or in a message to the controller. Deciding what comes next is the controller's. The section stays mandatory
for the controller and for anyone working with no controller.

🔴 Never close a ticket as done without the commit that did the work — not doing it / a duplicate ⇒ close it with the reason ·
a multi-sided ticket must not be closed until every side has a commit.

---

## 13. Traps that keep coming back

### An endpoint called on a cycle must send only what changed

**Never assemble the whole response on every request.** It must have: a version that only increases · the caller carrying its own cursor ·
unchanged ⇒ a short answer at constant cost · changed ⇒ only the difference · and the caller redraws only what changed.

**Signs of a violation:** reading a whole table and filtering out more than half · counting by walking history instead of using a count command ·
setting a refresh interval without looking at how often the data changes · fixing it by adding machines instead of doing less work per request.

### Several streams at once — reserving a ticket is not enough, guard at merge time

Different tickets can touch the same file ⇒
- **Catch up with main both when opening the PR and before merging**, not once
- **Never resolve a conflict by taking one side wholesale** — read what the other side changed and **why** first
- **After catching up with main, your earlier run results are void** — prove two things: your own tests can still go red on the new base
  **and** the tests of the side you merged over are still green

### Every app must be able to say which version it is

When a customer calls, you must be able to say which build they are on.
- Always report four fields: version · commit id · build time · release channel
- 🔴 **The front end reports the version of the back end it is talking to**, otherwise "it's broken" cannot be placed on a side
- 🔴 **A version nobody reads is always wrong without anyone knowing** ⇒ it must be emitted at run time,
  and CI must fail if it does not match what was declared
- Confirm a release by **the build's identity**, not by HTTP 200 (a 200 can come from the old one)
- **Never change the version number in a feature PR** — move it only in the release commit

### Always clean up what you created, before reporting done — never leave junk behind

Whoever finishes a piece of work cleans up. This applies to every role.
Several implementers on one machine create build outputs, test databases and containers, and nobody cleans them up
⇒ the disk fills faster than anyone expects, and the machine slows down because things nobody uses are still running.
The principle: **anything holding one ticket's data = use once and discard · anything expensive to recreate that holds nobody's data = reuse.**

| Thing | What to do |
|---|---|
| A database/cache container started to test one ticket | run it so it deletes itself on stop (`docker run --rm`) + label it with its owner and ticket · never set it to restart by itself |
| A database server for tests | use the project's standing one · let the tests create and drop their own sub-databases |
| The build-output folder of each worktree | delete it together with the worktree at merge or when the ticket is abandoned · 🔴 **never let two worktrees of the same project share one build-output folder** — some build tools (cargo, for one) do not put the path into their hash ⇒ the second worktree runs the other one's build output with no warning |

**Before reporting done, these must be empty:** containers carrying your label · the ticket's worktree along with its build output.

A rule on its own gets forgotten ⇒ there must also be a **periodic sweeper** as a second gate, and the sweeper must guard "in use" two ways:
(1) no file **at any depth** was modified within the time threshold (2) no process has a file **inside** open
— checking only the top level, or only the directory itself, deletes the build output of work that is currently running (both have happened).

**Merged or closed ⇒ whoever merges or closes it removes the worktrees.** The implementer must keep its worktree while the PR is open (a failed review sends it back),
and the merge happens later in another session, often on another machine ⇒ unless the merge step owns the cleanup, nobody does, and worktrees pile up by the hundred.
Never remove one a process is inside, or one with uncommitted changes or commits that are not on the remote.

The sweeper covers worktrees and scratch clones too, as a backstop. It removes one only when **all** hold:
no process inside it · nothing outside the git metadata changed within the threshold · the working tree is clean ·
every commit is on the remote, or its PR is merged or closed and that PR's head contains the local commit (an open PR ⇒ kept) ·
it is not locked or on a keep list · for a clone, also no stash, no live linked worktree, and every local branch safe.
It never removes unmerged work by age alone, and it logs the reason for every folder it keeps, so "why is this still here" always has an answer.

### Handing work across rounds

When you are near full, or before a large piece of work, record into a context file: what the task is · the branch and its state ·
**what has been done** · **the next steps in order** · key files/tests · what is still waiting on a decision · how to prove it.

Starting a new round: read that file first → check you are on the right branch → **continue from the next step immediately, without re-asking**
→ clear out the parts that are finished.

---

## Source

Distilled from an internal guide that is used every day, with all infrastructure detail removed.

If you find an item that does not hold up in practice, or want to propose an addition, open an issue.
