# CADEX PROTOCOL — Claude Code Entry Point
# Version: 0.1
# This file is read automatically by Claude Code at session start.
# It instructs the agent to follow the CADEX context protocol on every task.
# Do not delete or rename this file.

## What CADEX is

CADEX (Context-Aware Differential Execution) is a file-based protocol that
makes every AI coding session token-efficient by loading only what is relevant
to the current task — never the full project state.

Target: ≤ 900 tokens per task call.
Achieved by: differential loading, symbol-level indexing, structured deltas,
             and mode-based retrieval recipes.

---

## Execution protocol — follow this order on EVERY session

### Step 1 — Always load first (no exceptions)
Read `.cadex/IDENTITY.md` in full.
Read `.cadex/TASK.md` in full.
Classify task mode from TASK.md: bugfix | feature | refactor | design | docs

### Step 2 — Load digest chain
Read `.cadex/DIGEST.log` — last 7 entries only. Do not load full history.

### Step 3 — Load retrieval recipe
Read `.cadex/BUDGET.md` — find the recipe for the classified mode.
This recipe defines exactly which files and token budgets apply.

### Step 4 — Query index
Read `.cadex/INDEX.map`.
Extract keywords from TASK.md task description.
Match keywords against each file's keyword list.
Build a shortlist of matched files and matched symbols only.

### Step 5 — Load decisions (bounded)
Read `.cadex/DECISIONS.log`.
Load: last 10 entries + any entries whose keywords match the current task.
Do not load full decision history beyond this.

### Step 6 — Load open loops (mode-dependent)
Read `.cadex/OPEN_LOOPS.log`.
Load based on mode recipe from BUDGET.md.
For bugfix mode: load all open bugs first.
For design mode: load all open decisions.
For feature/refactor/docs: load only blocking loops (status: open, type: blocker).

### Step 7 — Load artifact symbols (lazy, selective)
For each file in the matched shortlist from Step 4:
  - Verify the symbol hash in INDEX.map matches the actual file content
  - If hash matches: load only the matched symbols, not the full file
  - If hash mismatches: re-read the full file, update INDEX.map, then load symbols
  - Never read a file not in the keyword-matched shortlist

### Step 8 — Verify budget
Count total tokens assembled across all loaded context.
If over the mode budget from BUDGET.md:
  - Apply escalation rules in BUDGET.md in order
  - Never exceed 900 tokens by loading more context
  - Split the task if necessary

### Step 9 — Execute task
Complete the task defined in TASK.md.
Stay within the scope defined in TASK.md scope field.
Verify completion against done_when and verify_steps fields.

### Step 10 — Write back (mandatory after every completed task)
After task is complete, the agent MUST:

  a) Append one JSON delta block to `.cadex/DIGEST.log`
     - Assign next sequential task_id (T-NNN)
     - Fill all 6 delta fields (intent, artifact, state, execution, next_pointer, tokens)

  b) Update `.cadex/INDEX.map`
     - Update hash for every symbol that was edited
     - Add new symbols that were created
     - Do not remove old symbols unless explicitly deleted

  c) If a new decision was made: append to `.cadex/DECISIONS.log`
     - Include confidence, source, last_verified_at
     - If the decision resolves a provisional entry, update that entry's confidence

  d) If a new unresolved issue was found: append to `.cadex/OPEN_LOOPS.log`
     - Assign next sequential OL-NNN id

  e) If an open loop was resolved: update its status to "resolved" in OPEN_LOOPS.log
     - Add resolved date and resolution description

  f) Clear `.cadex/TASK.md` task content
     - Set status: complete and task_id before clearing

---

## Hard rules — never violate these

1. Never read a file not matched by the current task's keywords in INDEX.map
2. Never re-read a file already loaded in this session unless hash mismatch forces it
3. Never load DIGEST.log entries older than the last 7
4. Never load DECISIONS.log entries beyond last 10 + keyword matches
5. Never exceed the mode token budget — split the task instead
6. Always write DIGEST.log entry after task completion — no exceptions
7. Never treat provisional or speculative decisions as confirmed constraints
8. Never load volatile execution_delta entries from previous sessions as current truth

---

## File reference

| File                    | Layer           | Lifetime      | Load rule                    |
|-------------------------|-----------------|---------------|------------------------------|
| .cadex/IDENTITY.md      | Session policy  | Permanent     | Always, every session        |
| .cadex/BUDGET.md        | Session policy  | Permanent     | Always, after mode classify  |
| .cadex/DECISIONS.log    | Project memory  | Durable       | Last 10 + keyword match      |
| .cadex/OPEN_LOOPS.log   | Project memory  | Until resolved| Mode-dependent               |
| .cadex/INDEX.map        | Artifact addr.  | Per-edit      | Query before any file load   |
| .cadex/DIGEST.log       | Execution mem.  | Rolling 7     | Last 7 entries only          |
| .cadex/TASK.md          | Execution mem.  | Per-task      | Always, every session        |

---

## CADEX version and attribution

Protocol:  CADEX v0.1
Spec:      https://github.com/satya928/cadex-protocol
Theory:    Agenity Research Series — Paper 2: Agent Brain Architecture
           https://zenodo.org/[paper-2-doi]
Author:    Sastry Chirravuri — https://orcid.org/0009-0005-5084-736X
License:   MIT
