# CADEX TASK.md — current active task slot
# One task at a time. Write your task here before starting a session.
# The agent reads this first (after IDENTITY.md) to classify mode and load context.
# When the task is complete, the agent clears this file and writes to DIGEST.log.
#
# MODE options: bugfix | feature | refactor | design | docs
# (mode determines which BUDGET.md recipe is used for context loading)

# ─────────────────────────────────────────────────────────
# SCHEMA
# ─────────────────────────────────────────────────────────
# task:        [one sentence — what needs to be done]
# mode:        [bugfix | feature | refactor | design | docs]
# scope:       [specific files or symbols to work within — be narrow]
# context:     [any extra info the agent needs that isn't in the CADEX files]
# done_when:   [concrete, testable completion criteria]
# verify_steps:[ordered list of checks before marking complete]
# blocked_by:  [OL-NNN if an open loop must be resolved first, else "none"]

# ─────────────────────────────────────────────────────────
# CURRENT TASK — replace everything below with your task
# ─────────────────────────────────────────────────────────

task:        Add rate limiting to POST /api/agents endpoint
mode:        feature
scope:       src/api/agents.ts, src/middleware/rateLimiter.ts (create if missing)
context:     Use express-rate-limit. Limit to 100 requests per 15 minutes per IP.
             Return 429 with JSON body: {"error": "rate_limit_exceeded", "retry_after": 900}
done_when:   Middleware applied to registerAgent route. 429 returned correctly on breach.
             Unit test added for rate limit behaviour.
verify_steps:
  1. Run existing agent tests — all pass
  2. Manually verify 429 response after 100 requests
  3. Check error response matches schema above
  4. No regressions in other /api/agents routes
blocked_by:  none

# ─────────────────────────────────────────────────────────
# COMPLETED — agent writes this section when done, then clears file
# ─────────────────────────────────────────────────────────
# status:   pending | in_progress | complete
# task_id:  T-[NNN]  ← agent assigns next sequential ID
# note:     [any important note about how it was completed]

status:   pending
task_id:  T-[assign on start]
note:
