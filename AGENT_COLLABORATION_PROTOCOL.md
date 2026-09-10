# Two-Agent Collaboration Protocol

**Purpose:** a drop-in operating system for two AI agents working on the same
project without silently duplicating work, editing over one another, approving
their own assumptions, or turning a plausible result into a completion claim.

This document is project-agnostic. Copy it into a repository, replace the
placeholders in **Project setup**, and point every agent-facing instruction file
(`CLAUDE.md`, `AGENTS.md`, or equivalent) to it as the single collaboration
protocol.

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
├── <canonical status/roadmap>
└── <project instruction files>
```

Add this pointer to every agent-facing project instruction file:

```text
Read AGENT_COLLABORATION_PROTOCOL.md and agentchat.md before beginning work.
The collaboration protocol is mandatory. Do not duplicate or paraphrase it
here; update the protocol itself when the process changes.
```

Keep technical truth in canonical project documentation. `agentchat.md` is a
temporary coordination channel, not a substitute for a roadmap, handover,
design record, or evidence report.

---

## 2. Roles and authority

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

## 3. The two-key rule

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

## 4. Exact manifest and approval provenance

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

## 5. Review quality

A review token proves that an approval occurred against particular bytes. It
does not prove that the review was intelligent.

The reviewer must:

1. read the original problem and predeclared discriminator;
2. inspect the relevant documentation before guessing about third-party tools;
3. inspect the actual diff and enough surrounding control flow to understand
   it;
4. independently recompute important counts, hashes and predicates;
5. enumerate early returns, failure paths and state-reset paths when continuity
   matters;
6. look for stale state, overflow, coordinate, lifetime, ordering and
   check-then-use errors;
7. verify that “no-op when disabled” remains true;
8. verify that the measured target is the intended target;
9. state what the evidence does **not** prove;
10. refuse certainty inflation.

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

Cross-family review reduces correlated blind spots but cannot eliminate them.
Keep periodic human spot checks on the project's highest-cost failure classes.

---

## 6. Predeclare experiments

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

## 7. Build and run discipline

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

## 8. `agentchat.md` protocol

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

## 9. Atomic dead drops

The dead drop is a wake-up signal, not a message body. Its file must be empty.
The real message is always the newest entry in `agentchat.md`.

### Zero-data invariant

**Dead-drop files must remain zero-data flags.** Their contents carry no
message, summary, status, context, request, acknowledgement or other
information. The only information a dead drop conveys is that the recipient
must re-read `agentchat.md`, which is the canonical coordination channel.

Do not optimize this by putting a note in the notice file. A payload can become
stale, split coordination state across two channels, or cause the receiver to
act on captured information instead of the current `agentchat.md` head. The
empty-file rule is part of the synchronization mechanism, not a formatting
preference.

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

## 10. Idle heartbeat automation

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
handle its actionable request within ownership and two-key review rules.
Delete only the claimed processing file after acting and signal the other
agent when required. Never start unreviewed source, configuration, build or
live-system work. Report only meaningful work, a review decision or a blocker.
```

The heartbeat must not poll aggressively, narrate unchanged state, start a
second live run, or interrupt an agent that has claimed a notice or owns the
shared build.

---

## 11. Failure and surprise handling

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

## 12. Commit and branch discipline

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

## 13. Recovery after a crash, deletion or usage limit

Do not reconstruct project state from conversational memory alone.

Recovery order:

1. read the canonical project instructions;
2. read the canonical status/roadmap and relevant evidence handover;
3. inspect version-control branch, status and recent commits;
4. consume any leftover `.processing.md` notice;
5. otherwise claim any canonical notice atomically;
6. read `agentchat.md` from the top;
7. inspect retained logs and manifests referenced by the latest review;
8. confirm current ownership and shared-build status;
9. continue from the last completed gate, not from a remembered intention.

An interrupted `.processing.md` file is work to resume, not garbage to delete.

---

## 14. Hook enforcement: useful but not sufficient

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

## 15. Compact message templates

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

---

## 16. Adoption checklist

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
- [ ] Run a dry exercise: design request → revision → diff ACK → simulated
      failure → recovery from `.processing.md`.

---

## 17. Final principle

This system cannot guarantee perfect reasoning. It can ensure that assumptions,
scope, reviewed bytes, execution authority and contradictory evidence remain
visible long enough for a second mind to challenge them.

The protocol succeeds when an error is caught before build or live execution,
not when both agents produce flawless first drafts.
