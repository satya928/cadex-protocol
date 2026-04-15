# CADEX — Context-Aware Differential Execution

**A model-agnostic, file-based context operating system for token-efficient AI coding sessions.**

> Every Claude Code / Cursor / Windsurf session re-ingests your full project from scratch.  
> A 3-hour coding session can consume 30,000–80,000 tokens before writing a single useful line.  
> CADEX fixes this architecturally — not by writing better prompts, but by making context waste impossible.

---

## The problem

When you use Claude Code, Cursor, or any AI coding tool, every task call reconstructs the entire context window:

- Full file contents re-sent on every task
- Conversation history re-ingested every turn  
- Unchanged architecture decisions re-explained every session
- The AI re-discovers the same bugs you already found yesterday

This is not a prompting problem. It is a **session architecture problem**.

## The solution

CADEX introduces differential context loading — the same principle as Git commits. Git doesn't re-store your entire codebase on every commit. It stores only what changed. CADEX applies this to LLM context:

- Only load what changed since the last session
- Only load the symbols being touched, not whole files  
- Only load decisions relevant to the current task
- Compress history into structured deltas — never raw conversation

**Result: 600–900 tokens per task call instead of 8,000–40,000.**

---

## How it works

CADEX is 8 plain text files in a `.cadex/` folder at your project root. Any AI tool that reads files follows the protocol automatically.

### The 4 layers

| Layer | Files | Purpose |
|---|---|---|
| Session policy | `IDENTITY.md`, `BUDGET.md` | Who you are, how much to load |
| Project memory | `DECISIONS.log`, `OPEN_LOOPS.log` | What was decided, what's unresolved |
| Artifact addressing | `INDEX.map` | Symbol-level file index with stale detection |
| Execution memory | `DIGEST.log`, `TASK.md` | What changed, what to do now |

The entry point (`CLAUDE.md` / `.cursorrules` / `.windsurfrules`) tells your AI tool to follow the protocol on every session.

### The execution loop

```
1. Read IDENTITY.md          → who is this project (120 tokens, always)
2. Read TASK.md              → what is the task, classify mode
3. Read DIGEST.log (last 7)  → what happened recently
4. Apply BUDGET.md recipe    → load only what this mode needs
5. Query INDEX.map           → find matched symbols by keyword
6. Load DECISIONS.log        → last 10 + keyword match only
7. Load matched symbols only → not whole files
8. Execute task
9. Write DIGEST.log delta    → structured JSON, not prose
10. Update INDEX.map hashes  → stale detection for next session
```

### 5 task modes — each with its own retrieval recipe

| Mode | Load priority | Skip |
|---|---|---|
| `bugfix` | Failing symbol + open bugs + recent deltas | Design context, full files |
| `feature` | Decisions + interfaces + affected symbols | Test noise, old history |
| `refactor` | Conventions + all symbols in scope | Open loops, test output |
| `design` | Full decision log + open questions | Execution deltas, file contents |
| `docs` | Public API symbols + decisions | Bugs, execution history |

---

## Install

**Option 1 — Copy the folder**
```bash
# Clone and copy .cadex/ into your project
git clone https://github.com/satya928/cadex-protocol
cp -r cadex-protocol/.cadex /your-project/.cadex
```

**Option 2 — npx (coming in v0.2)**
```bash
npx cadex-init
```

Then:
1. Fill in `.cadex/IDENTITY.md` with your project details (keep it under 120 tokens)
2. Copy `.cadex/CLAUDE.md` to your project root as `CLAUDE.md` (for Claude Code)  
   Or rename to `.cursorrules` for Cursor, `.windsurfrules` for Windsurf
3. Start writing tasks in `.cadex/TASK.md`

---

## The 8 files

### `.cadex/IDENTITY.md`
Your project's always-loaded core. Hard cap: 120 tokens. Contains: project name, stack, current goal, conventions, hard constraints, output format. Written once. Never changes mid-sprint.

### `.cadex/BUDGET.md`
Token budgets and retrieval recipes for each of the 5 task modes. This is the enforcement layer — without it, context loading is random. With it, every session has a defined maximum and a defined loading order.

### `.cadex/DECISIONS.log`
Append-only architectural decision record. Each entry carries: what was decided, why, what was rejected, which files it affects, and a confidence grade (`confirmed` / `provisional` / `speculative`). The agent loads only the last 10 + keyword-matched entries — never the full history.

### `.cadex/OPEN_LOOPS.log`
Unresolved bugs, pending decisions, risky assumptions, and external blockers. Prevents token-heavy re-discovery — if something is unresolved, it's here, not re-found in every session.

### `.cadex/INDEX.map`
Symbol-level file index. Maps every function, class, and logical section to its file path, content hash, and keywords. The agent queries this before reading any file. Hash mismatch = stale index = re-read before loading. Prevents loading 400-line files when only 20 lines are relevant.

### `.cadex/DIGEST.log`
Structured JSON delta chain. Every completed task emits one delta block with 4 typed fields: `intent_delta` (what happened), `artifact_delta` (what changed), `state_delta` (what was decided), `execution_delta` (what was observed). Replaces prose summaries entirely. Rolling window of 7 entries loaded per session.

### `.cadex/TASK.md`
The current active task. One task at a time. Contains: task description, mode, scope, done_when criteria, verify_steps, and any blocking open loops. The agent reads this to classify mode and load the right context. Cleared after each completed task.

### `.cadex/CLAUDE.md`
The entry point. Instructs Claude Code to follow the full CADEX protocol on every session. Contains the complete 10-step execution loop, hard rules, and file reference table. Rename to `.cursorrules` or `.windsurfrules` for other tools.

---

## Works with any AI tool

CADEX is entirely file-based. The underlying protocol is identical across all tools — only the entry point filename changes.

| Tool | Entry point file |
|---|---|
| Claude Code | `CLAUDE.md` |
| Cursor | `.cursorrules` |
| Windsurf | `.windsurfrules` |
| Aider | System prompt injection |
| Any LLM API | Inject `.cadex/IDENTITY.md` as system prompt header |
| Claude.ai Projects | Paste IDENTITY.md into project instructions |

---

## Theoretical foundation

CADEX is the reference implementation of the layered cognitive architecture proposed in:

**Agenity Research Series — Paper 2: Agent Brain Architecture**  
Sastry Chirravuri · [Zenodo DOI — add when published]

The paper proposes that LLMs should act as stateless reasoning engines, with memory, routing, and planning operating as independent external components under strict token budgets. CADEX demonstrates that this architecture is deployable today, in real projects, with measurable token reduction.

The architectural mapping:

| Agenity concept | CADEX implementation |
|---|---|
| Identity core (always loaded) | `IDENTITY.md` |
| Episodic memory | `DIGEST.log` volatile entries |
| Semantic memory | `DECISIONS.log` durable entries |
| Skill routing | Task mode classification |
| Retrieval budget | `BUDGET.md` recipes |
| Lite agent scoping | Mode-specific skip rules |

---

## Roadmap

- **v0.1** — Protocol spec + 8 file templates (current)
- **v0.2** — `npx cadex-init` installer, auto-detects tool and generates entry point
- **v0.3** — Hash validation CLI (`cadex verify`), stale index detection
- **v0.4** — `cadex pack --mode bugfix` assembles the context packet for inspection
- **v1.0** — Full CLI + adapters for Claude Code, Cursor, Aider, generic API mode

---

## Author

**Sastry Chirravuri** — Independent AI Researcher  
ORCID: [0009-0005-5084-736X](https://orcid.org/0009-0005-5084-736X)  
GitHub: [satya928](https://github.com/satya928)  
Agenity Research: [Zenodo](https://zenodo.org) | [Medium](https://medium.com)

---

## License

MIT — use freely, attribution appreciated.

---

*CADEX v0.1 — April 2026*
