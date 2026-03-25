# Kai

Autonomous AI Shopify expert, product manager, and developer loop for Claude Code. Published as `@kraftapps-ai/kai` on npm.

## Structure

- `kai` — the entire tool; a single bash script that bootstraps the PM agent and dev loop
- `package.json` — npm metadata and version
- `.kai/PROMPT.md` — per-project context (generated at runtime, not in repo)
- `.kai/memory.md` — persistent memory across sessions (generated at runtime, not in repo)
- `.kai/loop.sh` — dev loop script (generated at runtime, not in repo)

## How it works

1. `kai` script auto-inits project files (`kai.json`, `kai-progress.txt`, `.kai/PROMPT.md`, `.kai/memory.md`)
2. Auto-installs MCP servers if not already configured:
   - **Playwright** — browser testing inside Shopify Admin
   - **@shopify/dev-mcp** — Shopify docs, GraphQL schema introspection, Liquid/GraphQL/component validation
3. Generates `.kai/loop.sh` (the autonomous dev loop)
4. Launches Claude with `--continue` to resume the last conversation (per-repo)
5. Falls back to `.kai/memory.md` for context if conversation history is unavailable

## Session persistence

Kai uses a two-layer persistence strategy:
- **`claude --continue`** — resumes the exact last conversation in this directory (primary)
- **`.kai/memory.md`** — Kai-maintained memory file with key decisions, current focus, and session log (fallback)

Use `kai --new` to force a fresh conversation (memory.md is still loaded for context).

The PM creates user stories in `kai.json`, then kicks off `.kai/loop.sh` which runs Claude in a loop — one story per iteration with self-review. Both the PM and the dev loop worker use Shopify Dev MCP tools to introspect APIs, search docs, and validate code.

## MCP servers

- `@shopify/dev-mcp` — Official Shopify Dev MCP. Provides: `learn_shopify_api`, `search_docs_chunks`, `fetch_full_docs`, `introspect_graphql_schema`, `validate_graphql_codeblocks`, `validate_component_codeblocks`, `validate_theme_codeblocks`, `validate_theme`. No auth required.
- `@playwright/mcp` — Browser automation for testing embedded apps in Shopify Admin.
- Shopify Storefront MCP (remote, per-store) — `https://{shop}.myshopify.com/api/mcp`. Product search, cart management. No auth.

## Publishing

```bash
npm publish --access public
```

Version lives in `package.json` only.

## Testing changes

Run `./kai` in any project directory. It will auto-init if `kai.json` doesn't exist.
