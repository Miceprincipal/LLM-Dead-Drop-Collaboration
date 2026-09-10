# Two-Agent Collaboration and Knowledge Protocol

**Purpose:** a sterile, drop-in operating system for two AI agents working on
the same project without silently duplicating work, editing over one another,
approving their own assumptions, re-deriving knowledge the repository already
contains, repeating failed approaches, or turning a plausible result into a
completion claim.

This document is project-agnostic and publication-ready. Copy it into a
repository, replace the placeholders in **Project setup**, and point every
agent-facing instruction file (`CLAUDE.md`, `AGENTS.md`, or equivalent) to it
as the single collaboration and knowledge-management protocol.

The protocol is deliberately stricter around source changes, builds and live
tests than around read-only investigation. Its objective is not conversation or
paperwork. Its objective is to catch expensive mistakes before execution.

---

## 1. Project setup

Fill this block in when adopting the protocol:

```text
PROJECT: <name>
CANONICAL PROJECT INSTRUCTIONS: <path>
CANONICAL STATUS/ROADMAP: <path or paths>
CURRENT-TRUTH MANUAL: docs/CLEAN_MANUAL.md
FAILURE MANUAL: docs/FAILURES.md
NOTE LIBRARY INDEX: notes/INDEX.md
RAW NOTE LIBRARY: notes/
COORDINATION LOG: agentchat.md
DEAD-DROP DIRECTORY: deaddrop/

AGENT A: <name>
AGENT A NOTICE: deaddrop/<agent-a>checkagents.md

AGENT B: <name>
AGENT B NOTICE: deaddrop/<agent-b>checkagents.md

IMPLEMENTATION OWNER: <agent or per-task assignment>
REVIEW OWNER: <agent or per-task assignment>
SHARED BUILD OWNER: <agent>
```

Create this minimal layout:

```text
project-root/
├── AGENT_COLLABORATION_PROTOCOL.md
├── agentchat.md
├── deaddrop/
├── docs/
│   ├── CLEAN_MANUAL.md
│   └── FAILURES.md
├── notes/
│   ├── INDEX.md
│   └── <raw investigations, plans and handovers>
├── <canonical status/roadmap>
└── <project instruction files>
```

Add this pointer to every agent-facing project instruction file:

```text
Read AGENT_COLLABORATION_PROTOCOL.md, the configured current-truth manual and
agentchat.md before beginning work. The collaboration and knowledge protocol
is mandatory. Do not duplicate or paraphrase it here; update the protocol
itself when the process changes.
```

Keep technical truth in canonical project documentation. `agentchat.md` is a
temporary coordination channel, not a substitute for the current-truth
manual, failure manual, roadmap, handover, design record, or evidence report.

---

## 2. The three-part knowledge system

The repository uses three distinct knowledge stores. They have different jobs
and must not blur into one another.

### 2.1 Current-truth manual

`docs/CLEAN_MANUAL.md` is the canonical technical model: confirmed facts,
current architecture, live invariants, accepted interfaces, evidence policy,
current limitations, and ranked open questions. It answers: **what should a
new agent believe and build on now?**

Keep it clean:

- include the strongest current model, not a chronological diary;
- label the evidence class and scope of load-bearing claims;
- cite source files, commits, logs, documents or experiments precisely;
- distinguish observed fact, independent confirmation, inference, hypothesis
  and unknown;
- state negative scope: what the evidence does not establish;
- replace superseded conclusions with the corrected model and point to the
  failure manual for history;
- maintain a ranked open-question queue with the cheapest known discriminator;
- never silently import a claim from conversation memory or the coordination
  log.

Suggested skeleton:

```markdown
# Current-Truth Manual

**Last reconciled:** <date, commit or release>
**Purpose:** canonical current technical model

## Evidence policy and source precedence
## Current status and next bounded objective
## Architecture and invariants
## Confirmed subsystem findings
## Accepted interfaces and operational procedures
## Known limitations and negative scope
## Ranked open questions
## Canonical evidence map
```

Use a compact evidence vocabulary consistently:

- **VERIFIED LIVE** — observed through an accepted runtime oracle;
- **VERIFIED STATICALLY** — proved by direct source, instruction, data-flow or
  equivalent structural proof;
- **VERIFIED DATA** — established from exact shipped/generated data, hashes or
  tables;
- **INDEPENDENTLY CONFIRMED** — a second reviewer reproduced the decisive
  reasoning or measurement rather than agreeing with its summary;
- **STRONG INFERENCE** — several facts converge, but a decisive edge remains;
- **OPEN** — unresolved, disputed or deliberately unnamed.

Define source precedence for the project. Primary evidence normally outranks
conversation summaries; official documentation governs documented third-party
behavior; accepted live/static project evidence governs the project's own
behavior. “Newer” and “more confidently written” are not precedence rules.

### 2.2 Failure manual

`docs/FAILURES.md` is searchable negative knowledge: disproven theories,
failed experiments, superseded interpretations, abandoned architectures,
unreliable tools/oracles and useful negative results. It answers: **what has
already been tried, why did it fail, and what would have to change before it
is worth trying again?**

Use a stable taxonomy:

- **DISPROVEN** — direct evidence contradicts the claim;
- **FAILED EXPERIMENT** — the predeclared pass bar was not met;
- **SUPERSEDED** — a later model explains the evidence better;
- **ARCHITECTURALLY ABANDONED** — possible in isolation but incompatible with
  the chosen architecture;
- **TOOL/ORACLE FAILURE** — the instrument cannot support the judgment;
- **NEGATIVE RESULT** — a candidate was ruled out without finding the answer.

Every entry should preserve enough context to prevent accidental repetition:

```markdown
### <approach or claim> — <taxonomy status>

**Date/context:**
**Question or hypothesis:**
**Exact attempt/setup:**
**Manifest, version or environment:**
**Predeclared pass bar:**
**Observed result and retained evidence:**
**Why it failed or was rejected:**
**Replacement/current model:**
**Re-attempt condition:** <specific new condition, or NEVER under current architecture>
**Canonical references:**
```

Before retrying anything listed here, the proposing agent must state the
specific new condition that makes the old result inapplicable. “Trying it more
carefully” is not a new condition.

### 2.3 Indexed raw-note library

`notes/INDEX.md` is a catalog of raw investigations, plans, research reports,
handoffs, imported documents and historical notes. It is a discovery tool, not
a source of truth. It answers: **what detailed work already exists, how much
should it be trusted, and what supersedes it?**

Every material note in the configured raw-note tree must have one index card
(apart from `INDEX.md` itself). A note without a card is operationally
undiscoverable and should not be assumed to have been considered.

Use repository-relative links and a consistent card schema:

```markdown
### [`path/to/note.md`](path/to/note.md)

**Date:** <created or last materially revised>
**Scope/tags:** <platform, component, question, tool>
**Author/provenance:** <agent, person, imported source>
**Confidence:** HIGH | MEDIUM | LOW | MIXED | SUPERSEDED | N/A
**Evidence class:** <live, static, data, inference, hypothesis, reference>
**Summary:** <one paragraph saying what the note actually contributes>
**Key claims:** <short list if useful>
**Known errors/caveats:** <explicit, never hidden in prose>
**Canonicalized into:** <manual/roadmap/handover sections, or NOT YET>
**Superseded by:** <path/section, or NONE>
**Last reviewed:** <date and reviewer>
```

Confidence means:

- **HIGH** — extensively verified and safe to build on within its stated scope;
- **MEDIUM** — careful and likely correct, but partly static or incompletely
  reproduced; re-check before load-bearing use;
- **LOW** — thin evidence or hypothesis stage; use only as a lead;
- **MIXED** — contains both dependable and weak material; the card must say
  which is which;
- **SUPERSEDED** — useful history, but another source contains the current
  answer;
- **N/A** — reference data that does not itself make a claim.

The confidence belongs to the index review, not to the note's self-confidence.
An impressive tone is not evidence.

---

## 3. Mandatory lookup workflow

For every new question, use this order before fresh research or
implementation:

```text
new question
  -> notes/INDEX.md
  -> relevant indexed notes and supersession pointers
  -> current-truth manual and failure manual
  -> first-party project source/data
  -> official third-party documentation/source where applicable
  -> only then new research, tracing, disassembly or experimentation
```

This is a literal gate, not advice. Finding a promising filename is not the
same as reading it. Following a citation is not complete until the cited source
has actually been opened and checked.

At session recovery or handoff, use a broader startup order:

1. canonical project instructions;
2. current-truth manual's status, invariants and relevant subsystem sections;
3. canonical roadmap or active handover;
4. version-control state and exact checkpoint;
5. `agentchat.md` and any claimed dead drop;
6. the per-question lookup pipeline above.

Before proposing a fix, search the failure manual by subsystem, symbol,
symptom, tool and mechanism. Before trusting a raw note, read its index card
for corrections and supersession. Before guessing about a dependency, read
its official manual, README or source.

Pasteable instruction block for an agent-facing file:

```text
MANDATORY KNOWLEDGE ORDER

Before starting any new question, search the configured note index, open every
relevant indexed note and follow its supersession pointers, then check the
current-truth and failure manuals, then project source/data and official
third-party documentation. Only after those sources are exhausted may you
start fresh research or implementation. Conversation memory and the
coordination log are not canonical evidence.

Write confirmed current facts and open questions to the current-truth manual;
write rejected, failed and superseded paths to the failure manual; add or
update an index card for every material raw note. Do not revive a recorded
failure without naming the concrete new condition that invalidates the old
result.
```

---

## 4. Knowledge promotion and reconciliation

Raw work becomes canonical through review, not by existing in a file.

```text
investigation or imported source
  -> raw note + index card
  -> peer review / independent verification
  -> current fact or open question in CLEAN_MANUAL.md
     and/or failed path in FAILURES.md
  -> roadmap/status update when project state changes
```

After a material result:

1. preserve the raw evidence and exact provenance;
2. update or create its index card;
3. promote only the supported claim into the current-truth manual;
4. record disproven or rejected paths in the failure manual;
5. update roadmap/status only when the milestone or next action truly changed;
6. add supersession pointers in both directions where practical;
7. request documentation review before commit.

Do not copy the same narrative into every document. Canonical manuals should
state conclusions and boundaries; raw notes should retain method and detail;
the index should summarize and route; the roadmap should state progress and
next actions; the failure manual should preserve negative knowledge.

When sources conflict, do not silently overwrite either claim. Record both,
mark the status contested, identify the cheapest discriminator, and preserve
the superseded path after resolution. A correction should make old search
terms lead to the new answer.

Structural manual or index edits follow the same ownership discipline as code:
one editor at a time, fresh reread immediately before writing, exact diff
review afterward. The reviewer must check not only prose accuracy but also
whether the claim landed in the correct store.

If migrating from a monolithic manual, freeze it rather than deleting it. Mark
it prominently as superseded, index it as historical, move current conclusions
into the clean manual, move negative history into the failure manual, and point
old search terms toward their replacements. Never leave two documents both
claiming to be the canonical manual.

---

## 5. Roles and authority

For each task, name two roles explicitly:

- **Implementer:** investigates, proposes a design, edits within the approved
  scope, and produces evidence.
- **Reviewer:** independently checks the design, exact diff and evidence. The
  reviewer grants or refuses the two keys described below.

The roles may swap between tasks. They must not collapse into one role for a
risky change.

The reviewer is a technical peer, not an authority to obey blindly. The
implementer is not entitled to approval because it has already spent time on a
solution. Both agents must distinguish:

- observed facts;
- independently confirmed facts;
- strong inferences;
- hypotheses;
- unknowns.

No agent may broaden its own ownership, approve its own exemption, or silently
change the pass condition after seeing a result.

### Ownership rules

Before substantial work, post:

```text
OWNER: <agent>
TASK: <bounded objective>
FILES/SYSTEMS: <owned scope>
OTHER AGENT: <review or independent investigation>
```

Do not edit the same implementation files concurrently. This also applies to
structural edits in shared canonical documents. Re-read a shared section
immediately before changing it.

Ownership transfers must be explicit in `agentchat.md`. A previous session's
ownership assumption is not permission to edit today.

---

## 6. The two-key rule

Risky work requires two separate approvals.

### Key 1 — DESIGN ACK

Required before editing source, configuration, build scripts, generated inputs,
public interfaces, persistent data, or live-system behavior.

The implementer posts a **DESIGN REVIEW REQUEST** containing:

- the exact problem and evidence;
- the proposed mechanism, not merely the desired outcome;
- files and systems allowed to change;
- files and systems forbidden to change;
- invariants that must remain true;
- predeclared pass, fail and stop conditions;
- expected risks and cheaper alternatives considered;
- how the change can be reverted.

The reviewer independently checks relevant documentation and source, then
responds with one of:

```text
DESIGN ACK — approved exactly as scoped.
REVISE — specific defect or missing evidence.
BLOCK — design is unsafe or premise is unsupported.
```

Approval is not transferable to a materially different design.

### Key 2 — DIFF ACK

Required after editing and before any build, generator, test, live emulator,
deployment, benchmark, migration, or execution that could mutate state.

The implementer posts a **PRE-BUILD DIFF REVIEW** containing:

- the complete diff;
- an externally computed exact-byte manifest;
- every changed file;
- confirmation that no out-of-scope file changed;
- static checks performed;
- a return/error-path audit where relevant;
- unresolved concerns;
- an explicit statement that nothing has been built or run.

The reviewer independently reads the actual files and recomputes the manifest.
It must not approve from the implementer's summary alone.

The reviewer responds with:

```text
DIFF ACK — build/run only the reviewed manifest using the authorized commands.
REVISE — edit, then return with a new manifest; prior approval is void.
BLOCK — do not execute.
```

Any byte change after DIFF ACK invalidates the approval. Re-review is required.

### What needs both keys

Use both keys for:

- source or configuration changes;
- build-system changes;
- tests against live emulators, devices, services or production-like data;
- generators that can rewrite source or checked-in artifacts;
- interface/ABI/schema changes;
- changes to safety gates, approval hooks or their configuration;
- destructive or difficult-to-reverse operations.

Read-only inspection, log parsing and documentation research do not require
both keys, but findings still require peer review before becoming load-bearing.

Documentation-only changes normally require one post-edit review before commit.
Canonical status changes must cite the evidence that justifies them.

---

## 7. Exact manifest and approval provenance

The reviewed party must not be the sole authority for what was reviewed.

Use an external script or reviewer-owned command to hash the actual files. A
typical manifest contains:

```text
<sha256>  path/to/file-a
<sha256>  path/to/file-b
```

The reviewer independently recomputes it before issuing DIFF ACK. Recompute it:

1. immediately before the build or run;
2. immediately after the build;
3. after the final authorized run;
4. before commit.

Stop on any mismatch.

### Do not normalize away changes

When implementer and reviewer share a machine, hash exact bytes. Do not
normalize CRLF/LF, symlinks, or other representation details during approval:
normalization can make two genuinely different worktree states appear equal.

Track executable mode or other relevant metadata separately when the project
depends on it, for example with the version-control index or a manifest field.
If a cross-platform project deliberately uses canonical serialization, define
that policy before work begins; do not invent normalization after a mismatch.

### Approval tokens

If hooks use DESIGN ACK or DIFF ACK tokens, their hashes and tokens must be
computed and stored outside the implementer's writable trust boundary. A model
asserting “the hash is X” is not provenance.

The hook definition, verifier, approval store and token store are themselves
security-sensitive. If the constrained process can edit `.claude/settings.json`,
the hook script, or the approval store under the same OS identity, it can alter
the fence. Put real enforcement outside that writable area where practical.

---

## 8. Review quality

A review token proves that an approval occurred against particular bytes. It
does not prove that the review was intelligent.

The reviewer must:

1. read the original problem and predeclared discriminator;
2. follow the mandatory lookup workflow, including the note index and failure
   manual, before accepting a supposedly new approach;
3. inspect the relevant official documentation before guessing about
   third-party tools;
4. inspect the actual diff and enough surrounding control flow to understand
   it;
5. independently recompute important counts, hashes and predicates;
6. enumerate early returns, failure paths and state-reset paths when continuity
   matters;
7. look for stale state, overflow, coordinate, lifetime, ordering and
   check-then-use errors;
8. verify that “no-op when disabled” remains true;
9. verify that the measured target is the intended target;
10. state what the evidence does **not** prove;
11. check that new knowledge was placed in the correct canonical/index store;
12. refuse certainty inflation.

Useful adversarial questions include:

- Did the experiment measure the correct file, buffer, screen rectangle,
  coordinate system, time window, device, branch and binary?
- Is the denominator complete, or is `0/N` being reported over a partial `N`?
- Does a “last seen” value differ from “last successfully consumed”?
- Can overflow, stale publication or a repeated serial launder invalid state?
- Does an early return bypass cleanup, state advancement or an end-of-frame
  drain?
- Is a supposedly harmless build actually invoking a generator?
- Is a green run distinguishable from a lucky non-reproduction?
- Did the change fix the mechanism, or merely weaken the gate?
- Is this actually new, or is an indexed note/failure entry being rediscovered?
- Does the cited note still carry its claimed confidence, or was it superseded?

Cross-family review reduces correlated blind spots but cannot eliminate them.
Keep periodic human spot checks on the project's highest-cost failure classes.

---

## 9. Predeclare experiments

Before an expensive or live experiment, write down:

```text
HYPOTHESIS:
DISCRIMINATOR:
CONTROL:
EXPECTED IF TRUE:
EXPECTED IF FALSE:
FAIL-CLOSED CONDITIONS:
EXACT TARGET/REGION/TIME WINDOW:
RUN COUNT:
STOP CONDITIONS:
LOG PATHS:
```

Do not rewrite these after seeing the result. If the result exposes a defective
predicate, record that defect and design a new experiment; do not reinterpret
the old run into success.

For visual or spatial validation, every result must report the actual measured
rectangle/band/coordinate space from the run itself. “The screen looked right”
or “the code uses the expected coordinates” is not evidence that the captured
pixels came from the intended region.

Pin random seeds when reproducibility is valuable, but do not confuse a fixed
seed with full determinism. Scheduling, I/O, timers, thread interleaving and
external state may still vary. Preserve the logged seed with every result.

One clean run is not enough when the unfixed system sometimes passes by luck.
Use a mechanism-specific oracle and/or replicated runs with a predeclared count.

---

## 10. Build and run discipline

DIFF ACK authorizes only the stated sequence.

A safe default sequence is:

1. recompute the reviewed manifest;
2. verify the expected branch and worktree state;
3. run only the approved build command;
4. stop if generation occurs unexpectedly;
5. recompute the manifest after build;
6. verify the built artifact is newer than its reviewed sources;
7. run exactly the approved test or lifecycle count;
8. preserve complete logs;
9. recompute the manifest again;
10. post a POST-RUN REVIEW REQUEST and yield.

Do not improvise a fix during an authorized build/run. A failure returns the
work to design or diff review.

Compilation is not automatically exempt. Some builds run generators or mutate
sources. A compile-only exemption is acceptable only when the command is proven
non-generating, restricted to reviewed inputs, and followed by manifest
verification.

Any exemption proposed by the constrained agent receives **stricter** scrutiny,
even when it sounds reasonable. “Let me relax the mechanism constraining me” is
adversarial-shaped regardless of intent.

---

## 11. `agentchat.md` protocol

Use one newest-first log. Put a single permanent protocol header at the top,
then a separator and newest entries immediately below it:

```markdown
# AGENTCHAT

<short pointer to this collaboration protocol>

---

### 2030-01-01 — Agent B: newest message

...

---

### 2030-01-01 — Agent A: older message
```

Post before substantial work, at every review gate, when evidence surprises
you, when a claim becomes contested, and before declaring a milestone complete.

Do not silently rewrite a disagreement. Record:

```text
CLAIM A:
EVIDENCE:

CLAIM B:
EVIDENCE:

STATUS: CONTESTED
CHEAPEST DISCRIMINATOR:
```

Once resolved, update canonical documentation. Preserve useful rejected paths
in a failure/history record rather than erasing them from memory.

---

## 12. Atomic dead drops

The dead drop is a wake-up signal, not a message body. Its file must be empty.
The real message is always the newest entry in `agentchat.md`.

Agent A signals Agent B by creating Agent B's empty notice file, and vice
versa. Always signal after:

- answering a review request;
- creating a new review request;
- newly blocking or unblocking the other agent;
- posting information that should change the other's current work.

### Receiver algorithm

For recipient `<name>`:

1. If `deaddrop/<name>checkagents.processing.md` exists, resume that claimed
   notice first.
2. Otherwise, if `deaddrop/<name>checkagents.md` does not exist, there is no
   notice. End quietly.
3. Atomically rename the notice to
   `deaddrop/<name>checkagents.processing.md`.
4. Read `agentchat.md` from the top.
5. Act on the newest actionable request within current ownership and review
   rules.
6. Delete only the claimed `.processing.md` file after acting.
7. If replying or unblocking the sender, create the sender's empty notice.

The rename and delete must be separate from the sender's canonical filename.
Never read the canonical notice and then delete it: a new signal created in
that gap could be destroyed accidentally.

Atomic rename assumes both names live on the same filesystem. Do not put the
processing file in a separate temporary mount.

Check your own notice at the start and end of each user-prompted turn. The end
check catches a review message posted while you were working.

Do not put summaries inside notice files. Summaries become stale; an empty
signal forces the receiver to read the current log.

---

## 13. Idle heartbeat automation

An optional heartbeat can keep asynchronous work moving. It must be
non-interfering and quiet when nothing changed.

Suggested automation prompt:

```text
Run only when this project task is idle. If an assistant turn, command, live
run or review is active, end quietly. Check the recipient dead drop using the
atomic inbox protocol in AGENT_COLLABORATION_PROTOCOL.md. Consume a leftover
.processing file first; otherwise atomically rename the canonical empty notice
before reading agentchat.md from the top. Treat the notice as a signal only.
If no notice exists, create no files and report nothing. If a notice exists,
handle its actionable request within ownership and two-key review rules. Before
investigating any new question, follow the protocol's mandatory knowledge
order: note index, relevant indexed notes and supersession pointers,
current-truth and failure manuals, project source/data, then official
third-party documentation. Never treat agentchat.md or conversation memory as
canonical evidence. Preserve raw evidence and route any material result through
the note index and the correct manual; do not edit canonical knowledge without
the required ownership and documentation review.
Delete only the claimed processing file after acting and signal the other
agent when required. Never start unreviewed source, configuration, build or
live-system work. Report only meaningful work, a review decision or a blocker.
```

The heartbeat must not poll aggressively, narrate unchanged state, start a
second live run, or interrupt an agent that has claimed a notice or owns the
shared build.

---

## 14. Failure and surprise handling

When a run fails or evidence contradicts the theory:

1. stop the authorized sequence;
2. preserve the full log and exact binary/source hashes;
3. state the smallest claim the evidence supports;
4. do not apply a quick workaround;
5. do not weaken the gate, crop the sample, change the denominator, or exclude
   the failure after seeing it;
6. post the contradictory evidence immediately;
7. design the cheapest discriminating measurement;
8. return to DESIGN ACK before behavior changes.

A failure to reproduce is not a refutation. A passing control is not proof of
the mechanism. A green end-to-end result may still be a lucky run. Say which
one occurred.

Fail closed on incomplete evidence such as overflow, unknown ordering, stale
publication, partial denominators or unverified target identity. If fail-closed
behavior causes a red gate, that is evidence incompleteness—not permission to
pretend the evidence was complete.

---

## 15. Commit and branch discipline

Use a dedicated branch for risky work. Do not mix unrelated dirty-worktree
changes into the checkpoint.

Before commit:

- complete post-run review;
- update canonical documentation;
- recompute the reviewed manifest;
- inspect staged files explicitly;
- run the staged diff check;
- confirm no unrelated files are staged;
- record retained logs and evidence paths in the handover;
- record the exact tested target/region where relevant.

If implementation lives in a nested repository, commit it there. Do not sweep
an untracked parent directory or unrelated historical work into a convenient
mega-commit. Record both nested and parent commits when both are intentionally
changed.

---

## 16. Recovery after a crash, deletion or usage limit

Do not reconstruct project state from conversational memory alone.

Recovery order:

1. read the canonical project instructions;
2. read the current-truth manual's current status, invariants and relevant
   subsystem sections;
3. read the canonical status/roadmap and relevant evidence handover;
4. inspect version-control branch, status and recent commits;
5. consume any leftover `.processing.md` notice;
6. otherwise claim any canonical notice atomically;
7. read `agentchat.md` from the top;
8. search the note index, follow relevant supersession pointers and search the
   failure manual before resuming the question;
9. inspect retained logs and manifests referenced by the latest review;
10. confirm current ownership and shared-build status;
11. continue from the last completed gate, not from a remembered intention.

An interrupted `.processing.md` file is work to resume, not garbage to delete.

---

## 17. Hook enforcement: useful but not sufficient

Prompt rules are conventions. Hooks are harder gates, but they still need a
real trust boundary.

For Claude Code-style hooks:

- blocking requires exit status **2**, not merely any non-zero status;
- structured `permissionDecision: "deny"` is an alternative;
- gate `Edit`, `Write`, risky `Bash`, build and live-run commands;
- gate subagent dispatch or disable it if subagent tool calls bypass parent
  hooks;
- treat hook errors and silent hook failures as security events;
- keep verifier and approval data outside agent-writable paths where possible.

A “Read/Grep occurred” hook only proves access, not comprehension. It can be a
useful first gate but must not replace DESIGN ACK or DIFF ACK.

A strong enforcement design checks externally computed state such as:

```text
approved design token exists
AND current exact-byte manifest matches reviewed manifest
AND requested command belongs to the approved command class
AND approval has not already been consumed where single-use matters
```

Hooks cannot measure reviewer engagement depth. Cross-agent review, explicit
predicates, retained evidence and periodic human inspection remain necessary.

---

## 18. Compact message templates

### Work claim

```text
OWNER:
TASK:
SCOPE:
READ-ONLY OR MUTATING:
EXPECTED OUTPUT:
```

### Design review request

```text
DESIGN REVIEW REQUEST
PROBLEM:
EVIDENCE:
PROPOSED MECHANISM:
ALLOWED FILES/SYSTEMS:
FORBIDDEN CHANGES:
INVARIANTS:
PASS/FAIL/STOP CONDITIONS:
RISKS/UNKNOWNS:
ROLLBACK:
NOT STARTED:
```

### Design decision

```text
DESIGN ACK | REVISE | BLOCK
INDEPENDENT CHECKS:
DECISION:
EXACT AUTHORIZED SCOPE:
REQUIRED PRE-BUILD EVIDENCE:
```

### Pre-build diff review

```text
PRE-BUILD DIFF REVIEW
DESIGN ACK REFERENCE:
CHANGED FILES:
EXACT-BYTE MANIFEST:
DIFF SUMMARY:
RETURN/ERROR-PATH AUDIT:
OUT-OF-SCOPE CHECK:
UNRESOLVED CONCERNS:
NOT BUILT OR RUN:
```

### Diff decision

```text
DIFF ACK | REVISE | BLOCK
RECOMPUTED MANIFEST:
STATIC REVIEW:
AUTHORIZED BUILD COMMAND:
AUTHORIZED RUN COMMAND/COUNT:
REQUIRED OUTPUT:
STOP CONDITIONS:
```

### Post-run review request

```text
POST-RUN REVIEW REQUEST
SOURCE/BINARY MANIFEST:
EXACT COMMANDS:
LOG PATHS:
PREDECLARED PREDICATE RESULTS:
CONTROLS:
TARGET/REGION/TIME WINDOW ACTUALLY MEASURED:
FAILURES/OVERFLOW/UNKNOWN STATE:
SUPPORTED CLAIM:
NOT PROVED:
NO FURTHER WORK STARTED:
```

### Review rejection

```text
REVISE
DEFECT:
DIRECT EVIDENCE:
WHY IT MATTERS:
ONLY AUTHORIZED CORRECTION:
DO NOT BUILD/RUN:
```

### Knowledge reconciliation request

```text
KNOWLEDGE REVIEW REQUEST
NEW OR CORRECTED CLAIM:
EVIDENCE CLASS AND SCOPE:
RAW NOTE / RETAINED EVIDENCE:
INDEX CARD CREATED OR UPDATED:
CURRENT-TRUTH MANUAL CHANGE:
FAILURE MANUAL CHANGE:
ROADMAP/STATUS CHANGE:
SUPERSEDES / SUPERSEDED BY:
NEGATIVE SCOPE:
UNRESOLVED QUESTIONS:
```

### Documentation decision

```text
DOC ACK | REVISE | BLOCK
EXACT DOCUMENT HASHES:
EVIDENCE CROSS-CHECKED:
CONFIDENCE/SCOPE CHECK:
SUPERSESSION LINKS CHECKED:
PLACEMENT IN CORRECT KNOWLEDGE STORE:
UNSUPPORTED OR INFLATED CLAIMS:
```

---

## 19. Adoption checklist

- [ ] Fill every path and role in **Project setup**.
- [ ] Create `docs/CLEAN_MANUAL.md`, `docs/FAILURES.md` and `notes/INDEX.md`.
- [ ] Define evidence classes, source precedence and confidence meanings.
- [ ] Inventory existing notes and give every material note an index card.
- [ ] Mark legacy or monolithic manuals superseded; do not leave two sources
      both claiming canonical authority.
- [ ] Reconcile current facts into the clean manual and prior failed paths into
      the failure manual before relying on the system.
- [ ] Name the two agents and their notice files.
- [ ] Create `agentchat.md` and `deaddrop/`.
- [ ] Point all project instruction files to this protocol.
- [ ] Name canonical status and evidence documents.
- [ ] Assign implementation, review and shared-build ownership.
- [ ] Define which changes require both keys.
- [ ] Define the exact-byte manifest command.
- [ ] Put enforcement hooks and approval storage outside agent-writable scope
      where practical.
- [ ] Decide whether subagents are disabled or equally gated.
- [ ] Install an idle heartbeat only if it can remain non-interfering.
- [ ] Define high-cost human spot-check categories.
- [ ] Test the knowledge path: new question → index → relevant note → manuals →
      official source → reviewed canonical update.
- [ ] Run a dry exercise: design request → revision → diff ACK → simulated
      failure → recovery from `.processing.md`.

---

## 20. Final principle

This system cannot guarantee perfect reasoning. It can ensure that assumptions,
scope, reviewed bytes, execution authority, prior failures, source provenance
and contradictory evidence remain visible long enough for a second mind to
challenge them.

The protocol succeeds when an error is caught before build or live execution,
when existing knowledge prevents needless re-derivation, and when a corrected
claim becomes easier to find than the mistake it replaces—not when both agents
produce flawless first drafts.
