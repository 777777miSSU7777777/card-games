# AGENTS.md — Repository Working Rules

Root instructions for AI agents and developers of the `card-games` project. They apply to all tasks and all projects; subagents and Agent Teams inherit them.

- **`docs/` is the source of truth for the product and architecture** (overview, stack, game engines, plan). This file defines the **working process**. If a rule here conflicts with `docs/`, do not choose silently: report the conflict and ask.
- Changes to this file are a separate task and require agreement.
- Rules version: v3, 2026-11-26.

## 1. Plan Before Every Task

A task is ready to start when its goal, scope, public contracts, and acceptance criteria are clear, and its dependencies are identified. If something is missing, raise it before writing code instead of guessing.

Before any file changes:

1. **Goal** — what should change in the end, in one sentence.
2. **Steps** — 3–7 items; for a large task, milestones with completion criteria.
3. **Scope of changes** — which directories/projects will be touched and what is deliberately left alone.
4. **Verification** — the commands, tests, or manual scenario that prove completion.
5. **Risks and open questions** — resolve ambiguities before coding, not after.

Show the plan to the user before starting work and update it as you go. If the plan turns out to be wrong, stop, present a new one, and continue only after agreement. For a trivial edit (a typo, a single value), a 2–3 item plan is enough, but "plan first, act second" always applies.

## 2. Commits — Only After Approval

**Workflow:**

1. Show what changes: `git status --short` and `git diff --stat` (plus key diff hunks when needed).
2. Propose the commit subject and, if needed, the body.
3. Wait for an explicit "ok" / "commit".
4. Commit exactly what was shown. If anything changed after approval, show it again and ask again.

**Format** — Conventional Commits:

```
<type>(<scope>): <subject>

<body — why, not what>
```

- `type`: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `build`, `ci`, `perf`.
- `scope`: `repo`, `docs`, `server`, `web`, `mobile`, `infra`, `ci`.
- `subject`: English, imperative mood, up to 72 characters, no trailing period.
- Examples:
  - `feat(blackjack): add split and double actions`
  - `fix(poker): return uncalled bet on showdown`
  - `docs(plan): record sprint 1 scope`
  - `chore(repo): add root AGENTS.md`

**Rules:**

- One commit — one logical change; do not mix refactoring with a feature or fix.
- Stage only task-related files (no `git add -A`); do not touch other people's uncommitted changes.
- Never commit: secrets, `.env`, `node_modules`, build artifacts, local IDE files.
- Do not add AI/agent signatures, emojis, or `Co-authored-by` trailers to commits unless explicitly requested.
- Do not commit "red": lint, typecheck, and tests for the touched code must pass — or explicitly explain why not.
- `--no-verify`, amending published commits, `force-push`, and history rewriting — only with explicit approval.
- Push, PR, and merge require separate approval (commit approval does not imply push approval).

## 3. Core Engineering Rules (Project-Agnostic)

- **Instruction priority:** system/developer > direct user instructions > this file > `docs/`. On conflict, do not choose silently — report it.
- **Simplicity (YAGNI):** the smallest solution that fits the task; no abstractions or "future infrastructure".
- **Read before you change:** existing code, tests, and docs take precedence over personal preferences; follow the local style.
- **Minimal diff:** no reformatting, renaming, or refactoring outside the task.
- **Naming:** files and directories kebab-case; React components, types, and interfaces PascalCase; functions and variables camelCase; constants UPPER_SNAKE_CASE; database tables/columns snake_case. One primary export per module where practical.
- **Comments:** comments explain why, not what. Public APIs of shared modules and published packages get a short TSDoc block. Delete commented-out code — git keeps the history.
- **Verify, don't assume:** "done" only after an actual run of lint/typecheck/tests or a manual scenario; list what you ran and the result. Never invent commands, APIs, or results.
- **Destructive operations require approval:** deletion, `git reset --hard`, `git clean`, history rewriting, `force-push`, `drop`/`truncate` — only with explicit permission; check the absolute path before deleting.
- **Do not swallow errors:** fail fast, no empty `catch`; a clear message for the user, details in the log.
- **Structured logging:** use the structured logger everywhere in server and app code; no `console.log`. Include request/table/user context ids, choose levels deliberately, and never log secrets, tokens, or personal data.
- **Performance:** measure before optimizing. Do not add caching, indexes, or abstractions without a stated requirement or a profile that proves the need.
- **Task tracking:** do not leave `TODO` or `FIXME` without an owner and a concrete next step; prefer creating a task over a comment.
- **Leave it clean:** no temp files, debug output, commented-out code, or working-tree litter after the task.
- **Meta:** a new agreement becomes a line in this file; delete outdated rules instead of accumulating them.

## 4. Task Boundaries and Scope

- Do only what the task requires; incidental findings — a separate proposal or task, not "while I'm here".
- Do not change public contracts (WS/REST protocol, DB schema, config format) without agreement.
- Respect the agreed scope: features from the "out of scope" list require a separate task and estimate.
- Do not rewrite others' changes, style, or formatting outside the task area.

## 5. Stack and Dependencies

- TypeScript strict on the server, web, and mobile; each project is standalone, shared code is distributed as versioned packages instead of being imported across projects.
- Agreed stack: React 19 + Vite 8 + Mantine 9 + Redux Toolkit/RTK Query (web); Expo SDK 58 + RN Paper (mobile); NestJS 12 + Fastify + `ws` (server); PostgreSQL 18 + Drizzle; Zod; Vitest + fast-check + Testcontainers + Playwright; Biome; pnpm.
- New dependencies and technologies — only with justification (what they replace, why they are unavoidable) and agreement. No "just in case" libraries.
- Do not introduce alternatives without a decision: Redis, brokers, Kubernetes, microservices, GraphQL, Next.js, Tailwind, Zustand, TanStack Query, etc. (see `docs/02-stack.md`).
- Package manager — pnpm only; versions are pinned by each project's lockfile.

## 6. Repository Structure and Project Boundaries

- `server/`, `web/`, `mobile/` — three standalone projects: composition, platform code, and the modules each project needs.
- Each project is an independent package with its own `package.json`, `tsconfig.json`, `node_modules`, and lockfile; install and run commands inside the project directory.
- Projects never import each other's source. Shared code (rule engines, protocol, client layer, i18n) lives inside the project that needs it, is duplicated, or is published as a versioned package (private registry); there are no workspace links.
- Rule engines live inside `server/`: no `pg`, `fs`, `process`, network, or environment.
- The server is the source of truth: client actions are validated on the server; never trust client state.
- Projections: other players' cards never reach the client. Logs and errors contain no cards, tokens, or personal data.

## 7. Tests and Checks

- Every fix comes with a regression test that failed before the fix.
- Tests must be deterministic: no real network calls, no `sleep`, no dependence on wall-clock time or test order. Inject clocks and randomness; seed everything.
- Rule engines: unit + property (`fast-check`) + golden replay; cross-cutting invariants — `docs/03-game-engines.md`, §4.
- A new or changed rule = bump `rulesVersion` + new golden files; never edit existing ones.
- Before committing: inside each affected project run `pnpm lint`, `pnpm typecheck`, `pnpm test`; before a PR — the full run in every project.
- While a project's scaffold does not exist, do not invent commands. Once its `package.json` exists, use only its scripts; if a script is missing, report it.
- The repository must not contain: `.only`/`.skip` without justification, commented-out code, debug `console.log`, dead code, or test branches in production code.

## 8. Data, Money, and Security

- Secrets — only in `.env` (mode 600) and GitHub Secrets; keep `.env.example` in sync. Secrets never appear in code, the client, logs, or fixtures.
- SQL — only through Drizzle; migrations are forward-only, never edit applied migrations.
- Chips and pots — integers, transactional and idempotent; `float` for money is forbidden.
- Dangerous actions (data deletion, migrations on a real DB, CI/deploy/permission changes, sending data outside) — only after explicit approval.

## 9. UI and i18n

- All user-facing strings live in the project's locale resources (RU + EN); hardcoded strings in components are forbidden.
- Web — Mantine 9 and CSS modules; mobile — RN Paper. No custom design system.
- Errors: a clear message for the user, technical details in the log.
- Web and mobile keep their own protocol/client layer; sharing is only through versioned packages or deliberate duplication, never source imports.

## 10. Documentation

- `docs/` is the source of truth: when behavior or architecture changes, update the relevant document.
- A significant technical decision — an ADR in `docs/02-stack.md` (or a new ADR by agreement).
- Update README and instructions when commands, operations, or behavior change.

## 11. Communication and Reporting

- Reply in the user's language; code, identifiers, and branches in English.
- Do not present assumptions as facts: if you did not verify it, say so. Never fabricate test results, logs, or runs.
- When blocked — state exactly what you need (access, decision, clarification) and do not look for destructive workarounds.
- Address every review comment or decline it with a reason. Do not resolve a thread without a change or an answer, and re-request review after fixes.
- Batch non-blocking questions and ask them once; ask blocking questions immediately. Do not send status pings — report at milestones or when blocked.
- Task summary: what was done, how it was verified, what remains; name the remaining work explicitly if incomplete.

## 12. Definition of Done

A feature is done when (aligned with `docs/04-plan.md`, §9):

- [ ] It works in web and, for game screens, in mobile.
- [ ] Rules are covered by unit + property + at least one golden replay.
- [ ] Errors are handled and logged; the user sees a clear message.
- [ ] Other players' cards never reach the client; no secrets in the client.
- [ ] All strings are in resources; RU/EN switching works.
- [ ] README/instructions are updated if new operations appeared.
- [ ] Lint, typecheck, and tests are green.

## 13. Explicitly Forbidden

- Committing or pushing without approval.
- Bypassing checks: `--no-verify`, disabling lint, editing CI to get "green".
- Secrets in code, logs, tests, or fixtures.
- New dependencies or technologies "for the future" without agreement.
- Mass refactoring or reformatting of files outside the task.
- Implementing features from the "out of scope" list without a separate task and estimate.
