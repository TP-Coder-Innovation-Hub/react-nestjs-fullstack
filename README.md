# React + NestJS Fullstack Developer

A directory-per-topic learning path for building fullstack web applications with React, NestJS, TypeScript, and MongoDB — running on Bun.

## Learning Objectives

By the end of this path you can:

- Build React frontends with hooks, React Router, and TanStack Query
- Structure NestJS backends with modules, dependency injection, guards, and pipes
- Model and query MongoDB data with Mongoose
- Ship a fullstack app with auth, testing, and Docker

## Prerequisites

- Basic JavaScript syntax
- Familiarity with HTML/CSS
- Completed the JS/TS foundations (Bun setup, TypeScript basics) in [nodejs-typescript-backend](https://github.com/TP-Coder-Innovation-Hub/nodejs-typescript-backend)

## Structure

```
README.md                              — This file
01-react-fundamentals/                 — Components, JSX, hooks
  01-why-react.md
  02-jsx-components.md
  03-hooks-state.md
  04-hooks-effects-data.md
02-react-app-building/                 — Router, state, forms
  01-react-router.md
  02-state-context-zustand.md
  03-forms-validation.md
03-nestjs-backend/                     — Modules, DI, request pipeline
  01-nestjs-architecture-di.md
  02-controllers-providers.md
  03-pipes-guards-interceptors.md
04-mongodb-nestjs/                     — Persistence with Mongoose
  01-nestjs-mongoose-module.md
  02-schemas-repositories.md
  03-population-transactions.md
05-fullstack-integration/              — Frontend meets backend
  01-tanstack-query-client.md
  02-auth-flow-jwt-guards.md
  03-cors-env-config.md
06-production/                         — Testing and deployment
  01-testing-vitest-rtl.md
  02-deployment-docker.md
99-workshop/                           — Full project
  01-workshop-spec.md
```

## The Stack

| Layer     | Technology                                  |
|-----------|---------------------------------------------|
| Runtime   | Bun (package manager, test runner, dev)     |
| Language  | TypeScript everywhere                       |
| Frontend  | React 18 (hooks) + Vite                     |
| Backend   | NestJS                                      |
| Database  | MongoDB + Mongoose (@nestjs/mongoose)       |
| Data      | TanStack Query (server state on the client) |
