# Workshop: TaskFlow — Fullstack Task Manager

## Objective

Build and ship **TaskFlow**, a fullstack task management app: React frontend, NestJS API, MongoDB persistence, JWT auth — running on Bun, tested, and deployable with Docker.

This workshop uses everything from the path. There is no starter code; every decision is yours.

## Product Requirements

### Authentication

- Sign up with email + name + password (bcrypt-hashed, never stored plaintext)
- Login returns a JWT (1h expiry) and the current user
- Logout clears the client session
- All task routes sit behind `JwtAuthGuard`; users see only their own data

### Tasks

- Create a task: `title` (1–200 chars), `priority` (`low` | `high`), optional `dueDate`
- List my tasks, newest first, filterable by `?priority=`
- Complete / reopen a task
- Delete a task
- Optional stretch: tags array

### UI (React + Router + TanStack Query + Zustand)

- `/login` — login + signup forms (react-hook-form + Zod) with field-level errors
- `/` — task list: add form, filter tabs, complete/delete actions, loading and empty states via `useQuery`
- `RequireAuth` wrapper redirects unauthenticated users to `/login` (remembering the origin), sends them back after login
- Session in a Zustand store; token persisted to `localStorage`; mutations invalidate `['tasks']`

## Data Model

Design the schema yourself. Minimum shape:

```
users   { email (unique), passwordHash, name }
tasks   { title, priority, dueDate, completed, user (ref → users), timestamps }
```

Justify one embed-vs-reference decision in your README (e.g., why `user` is a reference, not embedded).

## API Contract

DTOs with class-validator — the Validation Pipe (`whitelist: true`) is required:

| Method | Path          | Auth | Notes |
|--------|---------------|------|-------|
| POST   | /auth/signup  | no   | 201 + token, 409 if email exists |
| POST   | /auth/login   | no   | 200 + token, or 401 |
| GET    | /tasks        | yes  | mine only, `?priority=` |
| POST   | /tasks        | yes  | 201, 400 on invalid |
| PATCH  | /tasks/:id    | yes  | complete/reopen; 404 if not mine |
| DELETE | /tasks/:id    | yes  | 404 if not mine |

## Architecture Expectations

- `server/` — feature modules (`auth/`, `tasks/`, `users/`) each owning controller, service, DTOs, schema, repository; `MongooseModule.forRootAsync` from `ConfigService`; config via `getOrThrow` only
- `frontend/` — `lib/api.ts` typed fetch wrapper with token injection, query keys module, features organized by domain
- Vite dev proxy for `/api` in development; nginx `/api` proxy in production — no CORS in either environment
- Tests: supertest e2e suite covering validation (400), auth (401), and ownership (user A cannot read user B's task — expect 404); at least one component test with RTL + msw

## Milestones

1. **Skeleton** — NestJS app with `/tasks` returning fixtures + Vite React app, Mongo via Compose → verify: both dev servers running, one endpoint round-trip
2. **Auth** — users module, JWT strategy + guard, login/signup forms, `RequireAuth` → verify: signup → refresh → still signed in; task routes 401 without token
3. **Tasks CRUD** — schema, repository, service, controller, list UI with TanStack Query → verify: create/complete/delete in the UI, data survives server restart
4. **Hardening** — global ValidationPipe, error states in the UI, invalidation on every mutation → verify: invalid input shows field errors; API returns clean 400s
5. **Testing** — e2e suite with a fresh database per run → verify: suite green, including the ownership test
6. **Ship** — Dockerfiles for both apps, nginx with `/api` proxy and history fallback, root Compose file → verify: `docker compose up --build` gives a working app on `:80`

## Deliverables

- Repo (or monorepo) with `server/` + `frontend/`, each starting with one command
- Passing tests (`bunx vitest run` and the supertest suite)
- `docker compose up --build` serving the full app
- README: setup, env vars (`.env.example` committed), your embed-vs-reference justification, what you would build next

## Definition of Done

- [ ] Two users, isolated data — proven by test, not by hope
- [ ] No secret in code; no `any` on the API boundary (DTOs in, response shapes out)
- [ ] Mutations invalidate queries — the UI reflects writes without manual refresh
- [ ] Fresh clone → `docker compose up --build` → working app
