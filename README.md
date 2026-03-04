# Graph

[![npm version](https://img.shields.io/npm/v/@graph-tl/graph)](https://www.npmjs.com/package/@graph-tl/graph)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![npm downloads](https://img.shields.io/npm/dm/@graph-tl/graph)](https://www.npmjs.com/package/@graph-tl/graph)

**Your agent forgets everything between sessions.** Your architecture decisions, naming conventions, business rules, project vision — gone. Every new chat starts from zero. You re-explain the same context, re-make the same decisions, and watch the agent drift from patterns you already established.

Graph fixes that. It's an MCP server that gives your agent a persistent project brain: vision and goals, architecture decisions, conventions, structured roadmaps, and automatic handoff between sessions.

## Install

```bash
npx -y @graph-tl/graph init
```

Restart Claude Code. That's it.

## See it work

Tell your agent:

> "Use graph to build out the project vision, architecture, and a roadmap for my app."

The agent will:
1. Interview you about goals, business rules, principles, and constraints
2. Document everything as persistent knowledge — architecture decisions, naming conventions, specs
3. Build a roadmap with prioritized, dependency-aware tasks across milestones
4. Start executing — claiming tasks, recording evidence, unblocking the next piece

Next session, a fresh agent picks up exactly where the last left off. It already knows your vision, your architecture, your naming conventions, and what was done yesterday.

## Before and after

**Without Graph** — every session starts cold:
```
You: "Continue working on the project"
Agent: "I don't see any prior context. What's the architecture?
        What conventions are you using? What's been built?"
You: *spends 5 minutes re-explaining everything*
```

**With Graph** — the agent calls `graph_onboard` and knows immediately:
```
Agent: "I see the project. Vision: multi-tenant SaaS platform.
        12 of 30 tasks done. Auth and data layer shipped last week.
        Conventions: kebab-case files, Zod validation at boundaries.
        3 tasks are actionable — I'll pick up the billing integration,
        it's highest priority and its dependencies are resolved."
```

One call. Full context. Zero re-explanation.

## What you get

- **Persistent project brain** — vision, goals, architecture, business rules, conventions — all survive between sessions
- **Structured roadmaps** — dependency-aware task trees with priorities, milestones, and automatic unblocking
- **Knowledge that sticks** — decisions, specs, nomenclature recorded once, auto-surfaced to every future agent
- **Evidence trail** — every task records commits, decisions, and file changes so nothing is lost
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

The agent calls `graph_plan` to build a dependency tree. This isn't a flat todo list — it's a structured breakdown with blocking relationships:

```
SaaS Platform
├── Foundation
│   ├── Document architecture decisions
│   ├── Define naming conventions & code principles
│   └── Write API spec
├── Core
│   ├── Auth & tenancy          (depends on: architecture, API spec)
│   ├── Data layer              (depends on: architecture)
│   └── Billing integration     (depends on: Auth & tenancy)
├── Features
│   ├── User management         (depends on: Auth & tenancy)
│   └── Dashboard               (depends on: Data layer, User management)
└── Release
    ├── E2E tests               (depends on: all Features)
    └── Deploy pipeline          (depends on: E2E tests)
```

The engine knows: architecture docs, naming conventions, and API spec are actionable now. Everything else is blocked. When a task resolves, dependents unblock automatically.

### Knowledge

As the agent works, it records decisions and conventions as persistent knowledge — not buried in chat history, but stored and auto-surfaced in future sessions:

```
knowledge: "architecture"  → "Event-driven, PostgreSQL, Redis for cache"
knowledge: "convention"    → "kebab-case files, Zod at boundaries, no default exports"
knowledge: "decision"      → "Stripe for billing — evaluated Paddle, chose Stripe for metered billing support"
knowledge: "api-contract"  → "REST, versioned /v1/, snake_case JSON fields"
```

Convention and architecture entries are automatically included in relevant tool responses. The agent follows your patterns without being told every session.

### Handoff

Session 1 ends after completing 5 tasks. Session 2 starts:

```
→ graph_onboard("my-project")

← goal: "SaaS Platform"
  summary: 5 of 14 resolved, 3 actionable
  recently_resolved: Architecture, Conventions, API spec, Auth, Data layer
  knowledge: 6 entries (architecture, conventions, 2 decisions, API contract, env setup)
  actionable: Billing integration (priority 9), User management (priority 8), Dashboard (priority 7)
  continuity_confidence: 92/100
```

The new agent knows the vision, the architecture, the conventions, what was built, and what to do next. The continuity confidence score tells it how much to trust the existing state.

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
