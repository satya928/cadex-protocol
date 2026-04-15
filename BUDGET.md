# CADEX BUDGET — retrieval recipes per task mode
# This file is the enforcement layer. Every session must stay within these budgets.
# The agent classifies task mode first, then loads ONLY what the recipe allows.

global_budget: 900 tokens total per task call

# ─────────────────────────────────────────────
# MODE: bugfix
# Use when: fixing a failing test, resolving an error, patching a regression
# ─────────────────────────────────────────────
mode: bugfix
  identity:       100   # IDENTITY.md — always
  open_loops:     150   # unresolved bugs from OPEN_LOOPS.log — first priority
  recent_deltas:  200   # last 3 execution deltas from DIGEST.log
  artifact:       300   # failing symbol + associated test only (from INDEX.map)
  decisions:       50   # max 2 relevant decisions from DECISIONS.log
  skip:           [design context, full file contents, digest history > 3 entries,
                   unrelated decisions, feature context]

# ─────────────────────────────────────────────
# MODE: feature
# Use when: building new functionality, adding an endpoint, creating a component
# ─────────────────────────────────────────────
mode: feature
  identity:       100
  decisions:      200   # last 5 + keyword-matched decisions
  recent_deltas:  150   # what changed in last 3 tasks
  artifact:       350   # interface definitions + symbols directly affected
  open_loops:      50   # blockers only, not all open loops
  skip:           [test output, execution history > 3 tasks, unrelated bug context]

# ─────────────────────────────────────────────
# MODE: refactor
# Use when: restructuring code without changing behaviour
# ─────────────────────────────────────────────
mode: refactor
  identity:       100
  decisions:      250   # conventions are critical here — load more decisions
  artifact:       400   # all symbols in refactor scope
  recent_deltas:  100   # only last task delta
  open_loops:       0   # skip unless a loop directly blocks refactor
  skip:           [open loops, test output, execution history, feature context]

# ─────────────────────────────────────────────
# MODE: design
# Use when: making architectural decisions, defining interfaces, planning approach
# ─────────────────────────────────────────────
mode: design
  identity:       100
  decisions:      300   # full decision context — most important in this mode
  open_loops:     200   # unresolved questions are design inputs
  artifact:       200   # interface map only — no implementation details
  recent_deltas:    0   # skip unless directly relevant
  skip:           [test output, execution deltas, file contents, bug history]

# ─────────────────────────────────────────────
# MODE: docs
# Use when: writing documentation, updating README, generating specs
# ─────────────────────────────────────────────
mode: docs
  identity:       100
  decisions:      200   # what was decided = what docs should reflect
  artifact:       300   # public API symbols only
  open_loops:      50   # document known limitations
  recent_deltas:    0   # skip
  skip:           [open loops (internal), execution deltas, test output, bug history]

# ─────────────────────────────────────────────
# ESCALATION RULES
# If a task cannot be completed within budget, follow these rules in order:
# ─────────────────────────────────────────────
escalation:
  1. Trim artifact load — load only the single most relevant symbol, not all matches
  2. Compress decisions — summarise multiple decisions into one block
  3. Skip open_loops entirely if not blocking current task
  4. If still over budget: split the task into two TASK.md entries and do them sequentially
  5. Never exceed 900 tokens by loading more context — split instead
