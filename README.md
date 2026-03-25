# Kai

Your AI product manager and development team. One command.

```bash
npm i -g @kraftapps-ai/kai
kai
```

## What is Kai?

Kai is a CLI that opens an interactive AI session acting as your **product manager**. Tell it what you want to build. It breaks the work into stories, then dispatches an autonomous AI developer to implement them — with a built-in reviewer that catches when the AI cuts corners.

## How it works

```
You ←→ Kai (PM)                    # brainstorm, plan, decide
              ↓
         kai.json                   # stories with acceptance criteria
              ↓
     ┌────────┴────────┐
     │  Developer AI   │            # implements one story at a time
     │  (claude -p)    │
     └────────┬────────┘
              ↓
     ┌────────┴────────┐
     │  Reviewer AI    │            # verifies each criterion independently
     │  (claude -p)    │
     └────────┬────────┘
              ↓
         Committed code             # repeat until all stories done
```

## Usage

```bash
cd your-project
kai
```

That's it. Kai will:

1. Auto-initialize if this is a new project
2. Ask what you want to build
3. Help you refine the idea and break it into stories
4. Write the stories to `kai.json`
5. When you say "go" — dispatch the AI developer loop
6. Report back when everything is done

Next time you run `kai`, it picks up where you left off — reads `kai.json` for stories and `kai-progress.txt` for what's been done.

## Example

```
$ kai
> I want to add Stripe payments to my Next.js app

Kai: Let me break that down. A few questions first:
- Checkout or subscriptions?
- Do you need a customer portal?
...

> Just one-time checkout for now. Ship it.

Kai: Created 8 stories in kai.json. Starting the dev loop...
  #1: Add Stripe SDK and env config
  #2: Create checkout API route
  #3: Add checkout button component
  ...
  All 8 stories complete!
```

## Files

```
your-project/
├── kai.json              # Stories (version control this)
├── kai-progress.txt      # Implementation log
└── .kai/
    ├── PROMPT.md          # Project context (edit this!)
    ├── loop.sh            # Implementation loop (auto-generated)
    ├── loop.log           # Loop output
    └── context.txt        # Optional: extra files to include
```

### Project context

Edit `.kai/PROMPT.md` with your tech stack, build commands, and conventions. Kai reads this on every startup and passes it to the developer AI.

## Install

```bash
npm i -g @kraftapps-ai/kai
```

Or without installing:

```bash
npx @kraftapps-ai/kai
```

**Requirements:** [Claude Code](https://claude.ai/claude-code) CLI, `jq`, `git`

## Context window

All state lives in files (`kai.json`, `kai-progress.txt`). When your session gets long, just exit and run `kai` again — it reads the files and picks up where you left off. No state is lost.

The developer loop already handles this by design — each story is a fresh `claude -p` call with a clean context window.

## The review step

After each story is implemented, a separate Claude instance reviews the code:

1. Reads the actual git diff (not the implementer's self-report)
2. Verifies each acceptance criterion
3. Is told: *"Be skeptical. The implementer may have cut corners."*

Failed reviews reset the story and feed back the failure reason for the next attempt. After 3 failures, the story is skipped.

## License

MIT
