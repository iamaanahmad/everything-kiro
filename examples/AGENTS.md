# AGENTS.md — directory-scoped agent instructions

> **What this file is:** Kiro supports the [AGENTS.md](https://agents.md) standard. An `AGENTS.md` is plain markdown steering that is always included (it does not support `inclusion:` front matter).
>
> Kiro picks up `AGENTS.md` from the workspace root and from `~/.kiro/steering/` (global). Since **IDE 1.0.309** it also discovers `AGENTS.md` files in subdirectories and loads each one as steering context alongside your other steering files ([steering docs](https://kiro.dev/docs/steering/)).

---

## Layout

```
my-project/
├── AGENTS.md              ← workspace root
├── src/
│   └── api/
│       └── AGENTS.md      ← sits next to the code it describes
└── infra/
    └── AGENTS.md
```

The docs do not specify precedence between nested files, so avoid contradictory instructions across levels. File references written inside an `AGENTS.md` resolve relative to the folder that contains it (see the steering docs for the reference syntax).

---

## Root-level example (place at project root as `AGENTS.md`)

```markdown
# Project conventions for Kiro

## Stack
- Node.js 22 + TypeScript 5.5
- Express for HTTP, Prisma for ORM, PostgreSQL 16
- Vitest for tests, ESLint + Prettier for linting

## Code style
- Use 2-space indentation for all TS/JS files
- Prefer `const` over `let`; avoid `var`
- Keep functions under 50 lines; split larger ones
- Always add explicit return types to exported functions

## Testing
- Test files live next to source files as `*.test.ts`
- Run tests with: `npm test`
- Minimum 80% coverage; 100% for auth and payment modules

## Git
- Commit message format: `type(scope): description` (conventional commits)
- Never commit to `main` directly — always use a feature branch

## What NOT to do
- Don't touch `src/legacy/` — read-only, migration pending
- Don't add new npm dependencies without noting them in chat
- Don't modify `docker-compose.prod.yml` without confirmation
```

---

## Subdirectory example (place at `src/api/AGENTS.md`)

```markdown
# API layer conventions

## Routing
- All routes live in `src/api/routes/`
- File name = resource name: `users.ts`, `orders.ts`
- Use resource nouns, HTTP methods for actions — no verb routes

## Response envelope
All responses must use this shape:
  { "success": true, "data": {}, "meta": { "timestamp": "..." } }
  { "success": false, "error": { "code": "...", "message": "..." }, "meta": { "timestamp": "..." } }

## Validation
- Validate all inputs with Zod at the route handler boundary
- Never pass raw `req.body` to the service layer

## Auth
- Every route that touches user data MUST call `requireAuth()` middleware
- User ID comes from `req.user.id` — never from URL params alone
```

---

## Infra example (place at `infra/AGENTS.md`)

```markdown
# Infrastructure conventions

## Terraform
- All resources must have `Name` and `Environment` tags
- Use `terraform plan` before applying any changes
- State stored in S3 — never run `terraform init` with local backend

## Secrets
- Never hardcode credentials in `.tf` files — use AWS Secrets Manager references
- Rotate access keys every 90 days

## Destructive operations
- Any resource deletion requires explicit user confirmation — always ask before proceeding
```

---

## Tips

- Keep instructions concise. The file is always included, so every line costs context.
- Use `.kiro/steering/` instead when you need `fileMatch`, `manual`, or `auto` inclusion modes. Use `AGENTS.md` for always-on rules, or for compatibility with other agent tools that read the same file.
- Custom agents do not load steering automatically. Add `resources: ["file://.kiro/steering/**/*.md"]` to an agent's config if it should see steering files.
- Commit `AGENTS.md` files to version control so the whole team benefits.
