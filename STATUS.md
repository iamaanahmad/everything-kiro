# Everything Kiro — Status

## Current status: tracked through Kiro IDE 1.4.178 and CLI 3.3.0 (changelog entries dated up to Nov 12, 2026)

Schemas checked against kiro.dev docs and the official IDE/CLI changelogs.

## Latest pass: corrections and IDE 1.4 / CLI 3 coverage

- **Fixed `examples/permissions.yaml`**: it told users to place a workspace file at `.kiro/settings/permissions.yaml`, which Kiro doesn't read. Rules live in `~/.kiro/settings/permissions.yaml` (user) or `~/.kiro/workspace-roots/<hash>/` (workspace). Removed `curl * | sh` patterns that can't match because Kiro splits pipes before matching, and removed an allow for `.kiro/hooks/**` that always prompts anyway.
- **Fixed `examples/AGENTS.md`**: removed an unsupported claim that nested files "merge" with the more specific one winning. The docs only say each file is loaded as steering.
- **Added**: `PreCompact`/`PostCompact` hooks, `inclusion: auto` steering, Memory, Workflows, Markdown Review, Approve for me, workspace trust, and the note that custom agents don't load steering unless `resources` lists it.
- **Not changed**: the 8 agents (their frontmatter was checked against the configuration reference).

## What's included

| Component | Count | Notes |
|---|---|---|
| Agents (`agents/*.md`) | 8 | IDE 1.0 Markdown format — frontmatter includes `tools`, `welcomeMessage`, and inline `permissions` where appropriate |
| Steering (`.kiro/steering/*.md`) | 5 | 3 `always`, 1 `fileMatch` example, 1 `manual` example |
| Hooks (`.kiro/hooks/*.json`) | 12 | v1 JSON schema, PascalCase triggers — see `hooks/README.md` |
| Powers (`powers/*`) | 3 | development-power, devops-power (Knowledge Base), database-power (Guided MCP) |
| Skills (`skills/**/*.md`) | 8 | development-workflows, language-patterns, infrastructure |
| Examples (`examples/*`) | 6 | example-spec, fullstack-webapp, AGENTS.md, permissions.yaml, sandbox.json, workflows/ |
| Documentation | README, INSTALL, CONTRIBUTING, CHANGELOG, STATUS | cross-checked against actual files |

## What changed in this pass (2026-09-04)

- **Agent format**: all 8 agents updated to IDE 1.0 Markdown format. Frontmatter now includes `tools` (short-form tags: `read`, `write`, `shell`, `web`) and `welcomeMessage`. `security-auditor` gets inline `permissions` to require approval on shell commands.
- **AGENTS.md**: new `examples/AGENTS.md` showing the directory-scoped instruction pattern added in IDE 1.0.309 — root-level, API-layer, and infra examples.
- **permissions.yaml**: new `examples/permissions.yaml` with a fully annotated reference for the capability-based permissions system that replaced Trusted Commands in IDE 1.0. Covers `deny`/`ask`/`allow` effects, glob patterns for filesystem, shell, and MCP capabilities.
- **README**: new sections covering Permissions, AGENTS.md, updated Custom Agents (IDE 1.0 format), Agent Focus Mode, Cloud Sessions, Cloud Configuration Sync, Agent Plugin format for Powers, global hooks, and hooks firing on agent-driven file changes.

## What changed in the previous pass (2026-08-05)

- **Hooks**: replaced `hooks/file-watchers.json` (legacy `eventType`/`hookAction` format) with 12 files under `.kiro/hooks/` using the real v1 schema.
- **Powers**: removed `development-power/power.json` (not a real schema file). Created missing steering files. Built out `devops-power` and `database-power`.
- **Steering**: added missing `security-rules.md` and `project-patterns.md` plus `api-conventions.md` (fileMatch) and `release-checklist.md` (manual).
- **Specs**: rewrote `skills/development-workflows/spec-driven-development.md` for the real EARS format; added `examples/example-spec/`.
- **Documentation**: created `INSTALL.md` and `CONTRIBUTING.md` for real; rewrote README file tree.
- **Kiro Crew**: README compatibility note added.

## Known gaps / not attempted

- Language-specific skills beyond TypeScript/Python/React (Go, Rust, Java) — not added.
- No CI workflow validating hook JSON syntax or dead markdown links — worth adding if this grows.
- `examples/permissions.yaml` can't be installed from the repo: Kiro denies agent writes to its settings directories, and workspace rules live outside the repo. Copy it to `~/.kiro/settings/` yourself.
- The hooks docs page didn't mention compaction triggers when fetched; `PreCompact`/`PostCompact` names come from the CLI changelog and Kiro's hook tooling. Verify in the IDE's hook editor.
- No agent in `agents/` loads steering via `resources`. Add it per agent if you want that.
- Cloud Configuration Sync and Cloud Sessions are documented but no cloud-specific config is included — those are account-level features managed in Kiro Web.
