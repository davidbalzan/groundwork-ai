---
title: "Architectural Decisions"
tags: [groundwork/reference]
aliases: ["DECISIONS", "ADR", "Decision Log"]
---

# Architectural Decision Records (ADRs)

> **Document the "why" behind significant technical decisions.**

ADRs capture context that's easy to forget: why we chose X over Y, what constraints existed, and what trade-offs we accepted. Future team members (and AI assistants) will thank you.

---

## Decision Log

| ID      | Decision                                                                              | Status                | Date       |
| ------- | ------------------------------------------------------------------------------------- | --------------------- | ---------- |
| ADR-001 | [Monorepo with Turborepo](#adr-001-monorepo-with-turborepo)                           | Accepted              | 2026-02-20 |
| ADR-002 | [React + Vite for Frontend](#adr-002-react--vite-for-frontend)                        | Accepted              | 2026-02-20 |
| ADR-003 | [Node.js + Hono for Backend](#adr-003-nodejs--hono-for-backend)                       | Accepted              | 2026-02-20 |
| ADR-004 | [ESLint 9 pinned (flat config)](#adr-004-eslint-9-pinned-flat-config)                 | Superseded by ADR-005 | 2026-06-24 |
| ADR-005 | [ESLint 10 via explicit React version](#adr-005-eslint-10-via-explicit-react-version) | Accepted              | 2026-06-29 |

---

## ADR-001: Monorepo with Turborepo

**Status**: Accepted
**Date**: 2026-02-20

### Context

We need to manage frontend, backend, and shared packages in a cohesive way across the project.

### Decision

Use Turborepo with pnpm workspaces for monorepo management.

### Consequences

**Positive:**

- Shared TypeScript types between frontend and backend
- Parallel builds and caching speed up CI
- Single repository simplifies dependency management
- pnpm provides efficient disk usage with symlinks

**Negative:**

- Need to configure Turborepo pipeline
- All team members work in same repo
- Shared packages require careful versioning

### Alternatives Considered

| Alternative    | Pros                               | Cons                            | Why Not                    |
| -------------- | ---------------------------------- | ------------------------------- | -------------------------- |
| Nx             | More features, powerful generators | Steeper learning curve, heavier | Overkill for most projects |
| Lerna          | Familiar, established              | Legacy, less active development | Outdated patterns          |
| Separate repos | Independent deployments            | Friction for shared code        | Coordination overhead      |

---

## ADR-002: React + Vite for Frontend

**Status**: Accepted
**Date**: 2026-02-20

### Context

Need a frontend framework with fast iteration and a mature ecosystem.

### Decision

Use React 19 with Vite as the build tool and Tailwind CSS 4 for styling.

### Consequences

**Positive:**

- React's component model and ecosystem maturity
- Vite provides fast HMR essential for UI development
- Tailwind 4 with CSS-first config and design tokens
- Wide library support and team familiarity

**Negative:**

- Bundle size consideration for production
- Tailwind's CSS-first config (no `tailwind.config.js`) differs from v3 habits

### Alternatives Considered

| Alternative | Pros                      | Cons                                   | Why Not                            |
| ----------- | ------------------------- | -------------------------------------- | ---------------------------------- |
| Next.js     | Full-stack, SSR           | SSR not always needed, adds complexity | Over-engineered for many use cases |
| Vue + Vite  | Great DX, smaller bundle  | Smaller ecosystem                      | Fewer libraries available          |
| Svelte      | Compiled, minimal runtime | Less mature ecosystem                  | Library ecosystem not ready        |

---

## ADR-003: Node.js + Hono for Backend

**Status**: Accepted
**Date**: 2026-02-20

### Context

Need a fast, lightweight backend framework with excellent TypeScript support.

### Decision

Use Node.js runtime with Hono web framework.

### Consequences

**Positive:**

- Hono is lightweight, Web Standards-based (~14kb)
- TypeScript-first with excellent types
- Built-in middleware (CORS, logger, Zod validation)
- Portable across runtimes (Node, Bun, Deno, Cloudflare Workers)

**Negative:**

- Smaller ecosystem than Express
- Team needs to learn Hono patterns

### Alternatives Considered

| Alternative | Pros                     | Cons                                  | Why Not                            |
| ----------- | ------------------------ | ------------------------------------- | ---------------------------------- |
| Express     | Huge ecosystem, familiar | Legacy patterns, no native TypeScript | Dated patterns                     |
| Fastify     | Fast, good TS support    | More complex plugin system            | Heavier than needed                |
| Bun + Hono  | Better performance       | Bun still evolving, edge cases        | Node.js more stable for production |

---

## ADR-004: ESLint 9 pinned (flat config)

**Status**: Superseded by [ADR-005](#adr-005-eslint-10-via-explicit-react-version)
**Date**: 2026-06-24

### Context

The monorepo lints with ESLint's flat config (`eslint.config.js`) and the React plugins (`eslint-plugin-react`, `eslint-plugin-react-hooks`). When refreshing dependencies, ESLint 10 is the latest major, but the latest published `eslint-plugin-react` (7.37.5) still calls the `context.getFilename()` API that ESLint 10 removed — it crashes at lint time with `TypeError: contextOrFilename.getFilename is not a function`. The plugin's declared peer range stops at ESLint `9.7`.

### Decision

Pin `eslint` and `@eslint/js` to the `9.x` line (currently 9.39.x) until `eslint-plugin-react` ships an ESLint 10-compatible release. All other tooling (TypeScript 6, Vite 8, Vitest 4, Zod 4) is on its latest major.

### Consequences

**Positive:**

- Lint runs clean across all four workspace projects
- Flat config (`eslint.config.js`) is already the modern format, so the migration to ESLint 10 will be a version bump, not a rewrite

**Negative:**

- One major version behind on ESLint until the React plugin catches up
- A reminder/check is needed before bumping ESLint to 10 (verify `eslint-plugin-react` peer range first)

### Alternatives Considered

| Alternative                            | Pros                       | Cons                                          | Why Not                                           |
| -------------------------------------- | -------------------------- | --------------------------------------------- | ------------------------------------------------- |
| ESLint 10 + drop `eslint-plugin-react` | Latest ESLint              | Lose React-specific rules (JSX, display-name) | Too much rule coverage lost                       |
| ESLint 10 + patch the plugin           | Latest ESLint, keep rules  | Maintain a fork/patch, fragile                | Not worth the upkeep for a starter template       |
| Pin ESLint 9 (chosen)                  | Clean lint, no maintenance | One major behind                              | Lowest risk; trivial to revisit when plugin ships |

---

## ADR-005: ESLint 10 via explicit React version

**Status**: Accepted
**Date**: 2026-06-29
**Supersedes**: [ADR-004](#adr-004-eslint-9-pinned-flat-config)

### Context

ADR-004 pinned ESLint to 9.x because `eslint-plugin-react` (7.37.5) crashes on ESLint 10
during React-version **auto-detection** (`contextOrFilename.getFilename is not a function`).
ADR-004 only weighed dropping the plugin or patching it. A simpler option was missed: the
crash fires _only_ in the auto-detect code path, which is skipped when the React version is
set explicitly.

### Decision

Move to **ESLint 10** (`eslint` ^10.6.0, `@eslint/js` ^10.0.1) and set
`settings.react.version` explicitly (to the installed React version) in `eslint.config.js`
instead of `"detect"`. This sidesteps the broken path without dropping or patching the
plugin. Verified green: lint, typecheck, and build all pass across all workspaces.

### Consequences

**Positive:**

- On the latest ESLint major; parity with `groundwork-ai-lite`.
- Keeps all React lint rules; no fork/patch to maintain.
- One-line config change, easy to remove once the plugin declares ESLint 10 support.

**Negative:**

- `eslint-plugin-react`'s declared peer range still stops below 10, so a harmless peer
  warning remains until it ships official support.
- `settings.react.version` must be kept roughly in step with the installed React major.

### Alternatives Considered

| Alternative                | Pros          | Cons                               | Why Not                                 |
| -------------------------- | ------------- | ---------------------------------- | --------------------------------------- |
| Stay on ESLint 9 (ADR-004) | Zero changes  | A major behind; diverges from lite | Superseded — the workaround is trivial  |
| Drop `eslint-plugin-react` | Latest ESLint | Lose React rules                   | Unnecessary given the workaround        |
| Patch/fork the plugin      | Keep rules    | Fragile upkeep                     | The explicit-version setting is simpler |

---

## ADR Template

```markdown
## ADR-XXX: [Title]

**Status**: Proposed | Accepted | Rejected | Superseded by ADR-XXX
**Date**: YYYY-MM-DD

### Context

What is the issue that we're seeing that is motivating this decision?

### Decision

What is the change that we're proposing and/or doing?

### Consequences

**Positive:**

- Benefit 1

**Negative:**

- Trade-off 1

### Alternatives Considered

| Alternative | Pros | Cons | Why Not |
| ----------- | ---- | ---- | ------- |
| Option A    | ...  | ...  | ...     |
```
