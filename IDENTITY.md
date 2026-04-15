# CADEX IDENTITY — always loaded, hard cap: 120 tokens
# Fill once. Never exceed this file size. Every session reads this first.

project:     [your-project-name]
stack:       [languages, frameworks, key libraries, infra]
goal:        [single sentence — current sprint or milestone objective]
conventions: [code style, naming rules, test policy, branch strategy]
do_not:      [hard constraints — things the agent must never do]
output:      [preferred response format — e.g. "code only, no prose" or "JSON"]

# --- EXAMPLE (delete and replace with your own) ---
# project:     Agenity — persistent AI agent identity platform
# stack:       TypeScript, FastAPI, Supabase, Pinecone, Polygon
# goal:        Ship agent registry MVP with REST API and reputation score
# conventions: snake_case functions, camelCase vars, tests required for API routes
# do_not:      Never modify migration files directly. Never push to main without tests passing.
# output:      Code blocks first, explanation after. No preamble.
