# Kai

Autonomous AI Shopify expert, product manager, and developer loop for Claude Code. Published as `@kraftapps-ai/kai` on npm.

## Structure

- `kai` — the entire tool; a single bash script that bootstraps the PM agent and dev loop
- `package.json` — npm metadata and version
- `.kai/memory.md` — persistent memory across sessions (generated at runtime, not in repo)
- `.kai/loop.sh` — dev loop script (generated at runtime, not in repo)
- `.kai/loop-mcp.json` — MCP config without Playwright for the dev loop (generated at runtime)

## How it works

1. `kai` script auto-inits project files (`.kai/stories.json`, `.kai/progress.txt`, `.kai/memory.md`)
2. Auto-installs MCP servers if not already configured:
   - **Playwright** — browser testing inside Shopify Admin (PM only, excluded from dev loop)
   - **@shopify/dev-mcp** — Shopify docs, GraphQL schema introspection, Liquid/GraphQL/component validation
   - **Mantle** — Shopify app billing, subscriptions, analytics
3. Generates `.kai/loop.sh` (the autonomous dev loop) and `.kai/loop-mcp.json` (MCP config without Playwright)
4. Launches a tmux session with two panes: PM (left) and dev loop (right)
5. Falls back to non-tmux mode if tmux is not installed or `--no-tmux` is passed

## tmux integration

Kai runs inside a tmux session (`kai`) with two panes:
- **Pane 0 (left)** — The PM: interactive Claude session where you chat with Kai
- **Pane 1 (right)** — Dev loop: streams live output when the PM kicks off `.kai/loop.sh`

When you exit and run `kai` again, it reattaches to the existing tmux session. If the dev loop is still running in pane 1, it continues undisturbed — the PM respawns in pane 0 with full context from `.kai/memory.md`, `.kai/stories.json`, and `.kai/progress.txt`.

Use `kai --no-tmux` to skip tmux and run in the current terminal (original behavior).

## Session persistence

Kai uses `.kai/memory.md` to preserve context between sessions. The PM system prompt injects story status, progress log, and memory on every startup.

Use `kai --new` to force a fresh conversation (memory.md is still loaded for context).

The PM creates user stories in `.kai/stories.json`, then kicks off `.kai/loop.sh` which runs Claude in a loop — one story per iteration with self-review. The dev loop uses a Playwright-free MCP config (`.kai/loop-mcp.json`) to avoid spawning Chromium browsers. Only the PM uses Playwright for QA.

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

Run `./kai` in any project directory. It will auto-init if `.kai/stories.json` doesn't exist. Use `./kai --no-tmux` to test without tmux.
