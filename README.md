# Everything Kiro

**A collection of Kiro IDE configurations — agents, hooks, steering files, powers, and skills — kept in sync with Kiro's actual current schemas.**

---

## What's Inside

```
everything-kiro/
├── .kiro/                        # Drop this whole folder into a workspace to use everything at once
│   ├── settings/
│   │   └── mcp.json              # MCP server configs (workspace-level)
│   ├── steering/                 # Always-active + conditional + manual context
│   │   ├── coding-standards.md   # inclusion: always
│   │   ├── security-rules.md     # inclusion: always
│   │   ├── project-patterns.md   # inclusion: always
│   │   ├── api-conventions.md    # inclusion: fileMatch (loads only for routes/controllers/api files)
│   │   └── release-checklist.md  # inclusion: manual (loads via /release-checklist)
│   └── hooks/                    # v1 JSON hooks, PascalCase triggers
│       └── *.json                # 12 hooks — see hooks/README.md for the list
│
├── agents/                       # 8 specialized agents (IDE 1.0 Markdown format)
│   ├── architect.md              # tools: read, web
│   ├── code-reviewer.md          # tools: read, web
│   ├── debug-detective.md        # tools: read, shell, web
│   ├── devops-specialist.md      # tools: read, write, shell, web
│   ├── documentation-writer.md   # tools: read, write, web
│   ├── performance-optimizer.md  # tools: read, shell, web
│   ├── security-auditor.md       # tools: read, web (shell: ask)
│   └── test-engineer.md          # tools: read, write, shell
│
├── powers/                       # Kiro Powers (POWER.md + optional mcp.json + optional steering/)
│   ├── README.md                 # Real Powers schema reference
│   ├── development-power/        # Knowledge Base Power: code review, TDD, debugging, QA
│   ├── devops-power/             # Knowledge Base Power: deploy checklist, Docker, CI/CD
│   └── database-power/           # Guided MCP Power: Postgres inspection via MCP
│
├── skills/                       # Workflow documentation, invoked by agents or read directly
│   ├── development-workflows/
│   │   ├── spec-driven-development.md   # Real .kiro/specs/ format, EARS acceptance criteria
│   │   └── tdd-cycle.md
│   ├── language-patterns/
│   │   ├── typescript-best-practices.md
│   │   ├── python-conventions.md
│   │   └── react-patterns.md
│   └── infrastructure/
│       ├── docker-workflows.md
│       ├── ci-cd-patterns.md
│       └── monitoring-setup.md
│
├── hooks/
│   └── README.md                 # Hook schema reference — the runnable hooks live in .kiro/hooks/
│
├── examples/
│   ├── example-spec/             # A real, complete .kiro/specs/password-reset/ example
│   ├── fullstack-webapp/         # Example project-level Kiro config (MCP, hooks, steering)
│   ├── AGENTS.md                 # AGENTS.md pattern: root, subdir, and infra examples
│   ├── permissions.yaml          # annotated reference — copy to ~/.kiro/settings/permissions.yaml
│   ├── sandbox.json              # local command sandbox example
│   └── workflows/                # workflow recipe skeletons
│
├── INSTALL.md
├── CONTRIBUTING.md
├── CHANGELOG.md
└── STATUS.md
```

---

## Key Concepts (as they actually work today)

### Steering files

Markdown files in `.kiro/steering/` (workspace) or `~/.kiro/steering/` (user), loaded based on frontmatter:

```markdown
---
inclusion: always        # default — every conversation
# inclusion: fileMatch
# fileMatchPattern: "**/*.api.ts"
# inclusion: manual      # loaded via slash command or #reference, not automatically
# inclusion: auto        # loaded when the request matches `description` (also needs `name`)
---
```

`manual` and `auto` steering files show up as slash commands (`/filename`). This replaced the old "manual hook trigger" concept entirely. Custom agents do **not** load steering automatically; add `resources: ["file://.kiro/steering/**/*.md"]` to an agent that needs it.

### AGENTS.md

Kiro supports the [AGENTS.md](https://agents.md) standard: always-included markdown steering with no inclusion modes. It is picked up from the workspace root and `~/.kiro/steering/`, and (since **IDE 1.0.309**) from subdirectories, so a file can sit next to the code it describes. The docs don't define precedence between nested files, so keep them consistent. See [`examples/AGENTS.md`](examples/AGENTS.md).

### Hooks

Hooks are versioned JSON files at `.kiro/hooks/*.json` (workspace) or `~/.kiro/hooks/*.json` (user):

```json
{
  "version": "v1",
  "hooks": [{
    "name": "lint-on-save",
    "trigger": "PostFileSave",
    "matcher": "\\.ts$",
    "action": { "type": "command", "command": "npm run lint" },
    "timeout": 30
  }]
}
```

Triggers: `SessionStart`, `Stop`, `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PreTaskExec`, `PostTaskExec`, `PostFileCreate`, `PostFileSave`, `PostFileDelete`, `PreCompact`, `PostCompact`. `PreToolUse`, `UserPromptSubmit`, `PreTaskExec`, and `PreCompact` can **block** — a command action exiting with code 2 stops the operation. See [`hooks/README.md`](hooks/README.md) for the full reference.

Global (user-level) hooks apply across every workspace (**IDE 1.0.182**). Hooks also fire on agent-driven file changes (**IDE 1.0.116**). `PreCompact` and `PostCompact` triggers (**IDE 1.4.121**, CLI 3.1) run around conversation compaction; a pre-compaction hook can cancel it by exiting with code 2, and neither takes a matcher.

### Permissions

A capability-based system that controls what the agent can read, write, execute, and call. Replaces the old Trusted Commands / Command Denylist from 0.x. Rules live in YAML files:

- `~/.kiro/settings/permissions.yaml` — user scope, all projects
- `~/.kiro/workspace-roots/<hash>/permissions.yaml` — workspace scope, one project

There is no in-repo permissions file. Workspace permissions are stored *outside* the repo (per-user) so a cloned repo cannot inject rules, and the agent cannot write to either location. Workspaces are untrusted until you trust them; until then Kiro doesn't load the workspace's custom agents, steering, MCP config, or skills.

Since **IDE 1.4.154**, the Autopilot toggle can become a permission mode menu (Supervised, Accept file changes, Approve for me). In Approve for me, a model judges only the actions your deny/ask/allow rules don't settle.

```yaml
rules:
  - capability: shell
    match: ["git *", "npm *"]
    effect: allow
  - capability: fs_write
    match: ["**/.env", "**/*.key"]
    effect: deny
```

Capabilities: `fs_read`, `fs_write`, `shell`, `web_fetch`, `web_search`, `mcp`, `subagent`, `skill`, `power`, `context`, `diagnostics`, `sandbox_network`. Priority: `deny > ask > allow > evaluate`; the most restrictive match wins across all scopes. Shell commands are split on `;`, `&&`, `||`, `|` before matching. See [`examples/permissions.yaml`](examples/permissions.yaml) for a full annotated reference.

### Custom Agents (IDE 1.0 format)

Agents are Markdown files in `.kiro/agents/` (workspace) or `~/.kiro/agents/` (user). The IDE 1.0 format adds `tools` tags, inline `permissions`, `welcomeMessage`, and more in the frontmatter:

```markdown
---
name: my-agent
description: What this agent does
tools: [read, write, shell, web]
welcomeMessage: "Ready. What are we working on?"
permissions:
  rules:
    - capability: shell
      match: ["npm *", "git *"]
      effect: allow
---

Your system prompt here...
```

Short-form `tools` tags: `read` = `fs_read`, `write` = `fs_write`, `shell`, `web` = web tools, `*` = all built-in tools. Switch agents mid-session without losing conversation history. The 8 agents in this repo each have an appropriate tools scope.

### Specs

Kiro's structured feature workflow: three gated files under `.kiro/specs/{feature_name}/`.

1. **requirements.md** — EARS-format acceptance criteria (`WHEN [event] THEN [system] SHALL [response]`)
2. **design.md** — architecture, interfaces, data models
3. **tasks.md** — checkbox list of coding-only tasks

Each phase requires explicit user approval before the next begins. **Quick Spec** mode generates all three in one pass for smaller features. See [`skills/development-workflows/spec-driven-development.md`](skills/development-workflows/spec-driven-development.md) and [`examples/example-spec/`](examples/example-spec/).

### Powers

Powers package documentation (and optionally an MCP server) into something Kiro activates on demand:

- **Knowledge Base Power** — `POWER.md` only (`development-power`, `devops-power`)
- **Guided MCP Power** — `POWER.md` + `mcp.json` (`database-power`)

Powers can also be packaged in the open **Agent Plugin format** (bundled skills + MCP), installable from a local folder or GitHub URL. Added in **IDE 1.0.288**.

No `power.json` — all metadata lives in `POWER.md` frontmatter. See [`powers/README.md`](powers/README.md) and the [official Powers registry](https://github.com/kirodotdev/powers).

### Agent Focus Mode

An experimental chat-first layout for directing multiple parallel agent sessions. Launch independent sessions, watch file changes as inline diffs, and use structured workflows (Spec, Plan, Bug Fix, Quick Spec) or freeform chat. Toggle from the top-right corner. Added in **IDE 1.0**; Cloud Sessions in Agent Focus added in **IDE 1.0.293**.

### Cloud Sessions

Run a session in the cloud alongside local ones. Start from Agent Focus Mode, pick the repositories it works against, and keep working after closing your laptop. Added in **IDE 1.0.293**.

### Memory

Kiro can carry repository conventions, corrections, and ruled-out approaches between sessions (**IDE 1.4.0**). The setting defaults to Off since 1.4.10; browse what Kiro remembers in the Memory view. Sub-agents get read-only access to memories (1.4.121).

### Workflows

A workflow runs a graph of agents in background sessions while you keep working in the parent chat. Each step has its own context, a reviewer doesn't inherit the coder's reasoning, and you can inspect, pause, or steer any step while it runs.

- **Recipes** live in `.kiro/workflows/*.workflow.{json,yaml,yml}` (project), `~/.kiro/workflows/` (user), or bundled by Kiro. Describe the outcome in chat and Kiro proposes a graph; save it as a recipe to reuse.
- **Nodes**: `step` (run an agent), `sequence` (ordered steps), `repeat` (loop until a condition or cap), `parallel` (independent branches), `watch` (poll an external system without model turns).
- **Built-in agents**: `wf-planner`, `wf-coder`, `wf-design`, `wf-design-reviewer`, `wf-review-aggregator`, `wf-pr-submitter`, `wf-pr-responder`, `wf-auto-researcher`, `semantic_reviewer`. Tool access comes from each agent's definition.
- **Use workflows**: controls how readily Kiro starts one: *Only when asked*, *Only for large tasks*, *As much as possible* (IDE 1.4.26).

Example shapes: investigate (read-only report), feature-pipeline (requirements → design/review → code/review → validate), troubleshoot-until-verified (bounded loop), publish-and-respond (PR watch loop). See [`examples/workflows/`](examples/workflows/) for an adaptable skeleton and the [workflow docs](https://kiro.dev/docs/workflows/).

### Local command sandbox

Limits what shell commands and stdio MCP servers can reach. The harness stays outside; only the command and its children are sandboxed. Enable in `~/.kiro/settings/sandbox.json` or `<workspace>/.kiro/settings/sandbox.json`:

```json
{
  "enabled": true,
  "networkAccess": "DEFAULT_BLOCKED",
  "mcpSandboxing": true
}
```

`networkAccess` modes: `DEFAULT_BLOCKED` (only `sandbox_network` allow rules), `COMMON_DEPENDENCIES` (common dev hosts allowed), `DEFAULT_ALLOWED` (no host filter). Host rules under `sandbox_network` in permissions.yaml are first-match. MCP sandboxing wraps stdio servers; HTTP/SSE servers are not affected.

Platform: macOS uses Seatbelt via `sandbox-exec`; Linux needs `bwrap`; Windows needs a one-time admin setup. See [`examples/sandbox.json`](examples/sandbox.json) and the [sandbox docs](https://kiro.dev/docs/sandbox/).

### Markdown Review

### Cloud Configuration Sync

Steering files, custom agents, skills, powers, and hooks can be synced to your Kiro account and accessed across the IDE and Agent Focus. Cloud-managed items show a cloud indicator and open as read-only with an "Edit in web" action. Added in **IDE 1.0.437**.

### MCP

External tool servers configured in `.kiro/settings/mcp.json` (workspace) or `~/.kiro/settings/mcp.json` (user). Supports the latest MCP protocol revision including OAuth for servers that require sign-in. Keep enabled servers to what a project actually needs — every enabled MCP server's tools count against context budget.

### Kiro Crew

[Kiro Crew](https://kiro.dev/blog/introducing-kiro-crew/) runs on Kiro CLI and reads existing `.kiro` configuration. This repository's steering, hooks, skills, and custom-agent patterns carry over after installation without a separate migration.

This is compatibility documentation only: Everything Kiro does **not** ship a Crew manifest, schedules, Apps, integrations, or autonomous orchestration workflows.

---

## Installation

See [INSTALL.md](INSTALL.md) for the full walkthrough. Short version:

```bash
git clone https://github.com/iamaanahmad/everything-kiro.git
cp -r everything-kiro/.kiro your-project/.kiro
cp -r everything-kiro/agents your-project/.kiro/agents
```

Then edit `.kiro/settings/mcp.json` and replace the placeholder tokens with real credentials — never commit real secrets.

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Match the real schema (check kiro.dev/docs if unsure), keep claims in README/STATUS/CHANGELOG in sync with what's actually on disk, and don't add a component without also updating the file tree above.

---

## Resources

- [Kiro IDE](https://kiro.dev)
- [Kiro Docs](https://kiro.dev/docs)
- [Kiro Changelog](https://kiro.dev/changelog/) — check here before assuming a schema is current
- [Official Powers registry](https://github.com/kirodotdev/powers)
- [Model Context Protocol](https://modelcontextprotocol.io)

---

## License

MIT — use freely, modify as needed, contribute back if you can.
