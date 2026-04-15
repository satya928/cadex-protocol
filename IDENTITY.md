# CADEX IDENTITY
# Hard cap: 120 tokens. Never exceed this.
# This file loads on EVERY session before anything else.
# Only update goal: when sprint changes. All other fields are stable.

project:     Agenity — persistent AI agent identity + economy platform
stack:       TypeScript, FastAPI, Supabase, Pinecone, Polygon, Next.js
goal:        Publish Paper 2 on Zenodo + launch cadex-protocol repo on GitHub
conventions: snake_case functions, camelCase vars, typed schemas always, tests required for API routes, append-only logs never edited
do_not:      Never modify migration files directly. Never push to main without tests passing. Never load full file when symbol slice suffices. Never skip DIGEST entry after a task.
output:      Code blocks first, explanation after. JSON deltas for task completions. Academic tone for paper drafts. No preamble.
