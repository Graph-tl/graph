# Graph

[![npm version](https://img.shields.io/npm/v/@graph-tl/graph)](https://www.npmjs.com/package/@graph-tl/graph)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![npm downloads](https://img.shields.io/npm/dm/@graph-tl/graph)](https://www.npmjs.com/package/@graph-tl/graph)

**Your agent forgets everything between sessions.** Every new chat starts from zero — re-reading files, re-discovering decisions, re-planning work that was already planned. You lose minutes (and tokens) re-explaining what happened last time.

Graph fixes that. It's an MCP server that gives your agent persistent memory: a dependency tree of tasks, evidence of what was done, and automatic handoff to the next session.

## Install

```bash
npx -y @graph-tl/graph init
```

Restart Claude Code. That's it.

## See it work

Tell your agent:

> "Use graph to plan building a REST API with auth and tests."

The agent will:
1. Create a project and interview you about scope
2. Decompose work into a dependency tree — nested tasks, priorities, blocking relationships
3. Claim and complete tasks one by one, recording what it did after each
4. When you start a new session, the next agent picks up exactly where the last left off

No copy-pasting context. No re-explaining what was done. The graph carries it forward.

## Before and after

**Without Graph** — every session starts cold:
```
You: "Continue working on the API"
Agent: "I don't see any prior context. What have you built so far?
        What's left? What decisions were made?"
You: *spends 5 minutes re-explaining everything*
```

**With Graph** — the agent calls `graph_onboard` and knows immediately:
```
Agent: "I see the project. 3 of 8 tasks are done. Auth module and
        API spec were completed last session using JWT with RS256.
        Routes and Database layer are ready to work on. I'll pick
        up Routes — it's highest priority. Claiming it now."
```

One call. Full context. Zero re-explanation.

## What you get

- **Session-to-session memory** — agents pick up exactly where the last one left off
- **Dependency engine** — knows what's blocked, what's ready, and what to work on next
- **Evidence trail** — every task records commits, decisions, and file changes so nothing is lost
- **Knowledge base** — persistent project knowledge (conventions, architecture decisions) auto-surfaced to agents
- **Local and private** — single SQLite file on your machine, no cloud, no telemetry

## How it works

```
graph_onboard   → "What's the state of this project?"
graph_next      → "What should I work on?" (claims it)
   ... agent does the work ...
graph_update    → "Done. Here's what I did." (resolves with evidence)
   → engine returns newly unblocked tasks
graph_next      → "What's next?"
```

### Planning

The agent calls `graph_plan` to build a dependency tree:

```
Build REST API
├── Design
│   └── Write API spec
├── Implementation
│   ├── Auth module          (depends on: Write API spec)
│   ├── Routes               (depends on: Write API spec)
│   └── Database layer
└── Testing
    ├── Unit tests           (depends on: Auth, Routes, Database)
    └── Integration tests    (depends on: Unit tests)
```

The engine immediately knows: "Write API spec" and "Database layer" are actionable. Everything else is blocked. When a task resolves, dependents unblock automatically.

### Handoff

Session 1 ends after completing 3 tasks. Session 2 starts:

```
→ graph_onboard("my-project")

← goal: "Build REST API"
  hint: "2 actionable task(s) ready. 3 resolved recently."
  recently_resolved: Auth module, API spec, Database layer
  knowledge: "JWT with RS256, keys in /config"
  actionable: Routes (priority 8), Integration tests (priority 7)
  continuity_confidence: 85/100
```

The new agent knows what was built, what decisions were made, and what to do next. The continuity confidence score tells it how much to trust the existing state.

## Tools

Graph exposes 22 MCP tools. Here are the ones agents use most:

| Tool | What it does |
|---|---|
| `graph_onboard` | Full project context in one call — summary, tree, evidence, knowledge, actionable tasks |
| `graph_plan` | Batch-create a task tree with dependencies. Atomic |
| `graph_next` | Get the highest-priority actionable task. Optional claim |
| `graph_update` | Resolve tasks with evidence. Returns newly unblocked tasks |
| `graph_resolve` | One-call resolve — auto-collects git commits and modified files |
| `graph_status` | Formatted project dashboard with progress and integrity checks |
| `graph_roadmap` | Release-pipeline view grouped by horizon (now / next / later / paused) |

<details>
<summary>All tools</summary>

**Core workflow:** `graph_open`, `graph_plan`, `graph_next`, `graph_update`, `graph_resolve`

**Navigation:** `graph_onboard`, `graph_context`, `graph_tree`, `graph_query`, `graph_history`

**Structure:** `graph_connect` (dependency edges with cycle detection), `graph_restructure` (move, merge, drop, delete tasks), `graph_roadmap`

**Quality:** `graph_status`, `graph_retro` (structured retrospectives with drift detection), `graph_agent_config`

**Knowledge:** `graph_knowledge_write`, `graph_knowledge_write_batch`, `graph_knowledge_read`, `graph_knowledge_search`, `graph_knowledge_delete`, `graph_knowledge_audit`

</details>

## Configuration

```bash
npx -y @graph-tl/graph init    # Auto-configures everything
```

Or add manually to `.mcp.json`:

```json
{
  "mcpServers": {
    "graph": {
      "command": "npx",
      "args": ["-y", "@graph-tl/graph@latest"],
      "env": {
        "GRAPH_AGENT": "claude-code"
      }
    }
  }
}
```

<details>
<summary>Environment variables</summary>

| Variable | Default | Description |
|---|---|---|
| `GRAPH_AGENT` | `default-agent` | Agent identity for audit trail |
| `GRAPH_DB` | `~/.graph/db/<hash>/graph.db` | Database path (per-project, outside your repo) |
| `GRAPH_CLAIM_TTL` | `60` | Soft claim expiry in minutes |

</details>

<details>
<summary>CLI commands</summary>

```bash
graph init           # Set up graph in the current project
graph update         # Clear npx cache and re-run init to get the latest version
graph ship           # Build, test, bump, commit, push, and create GitHub release
graph doctor         # Run integrity checks on all projects
graph backup         # List, create, or restore database backups
graph ui             # Start the web UI
graph --version      # Print version
graph --help         # Print usage summary
```

</details>

### Updating

Graph checks npm for newer versions on startup. To update:

```bash
npx @graph-tl/graph update
```

## Data & security

Your data stays on your machine.

- **Single SQLite file** in `~/.graph/db/` — outside your repo, nothing to gitignore
- **Local-first** — stdio MCP server, no telemetry, no cloud sync
- **No secrets stored** — task summaries, evidence notes, and file path references only
- **You own your data** — back it up, delete it, move it between machines

## License

MIT — free and open source.
