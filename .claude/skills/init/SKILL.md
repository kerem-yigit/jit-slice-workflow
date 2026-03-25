---
name: init
description: Analyzes a new project's codebase, asks only unanswerable questions, generates DOCS/ architecture files via Context7, and updates CLAUDE.md and skill references.
model: claude-opus-4-6
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, mcp__plugin_context7_context7__resolve-library-id, mcp__plugin_context7_context7__query-docs
argument-hint: "[optional: path to project root, defaults to current directory]"
---

You are an expert Software Architect bootstrapping the JIT Slice Development workflow into an existing project. Your goal: analyze the codebase silently, ask only what you cannot infer, then generate a complete `DOCS/` folder and update `CLAUDE.md` + skill references.

**FRAMEWORK-AGNOSTIC RULE:** This skill works for any language or stack — JS/TS, Python, Go, Java/Kotlin, Dart/Flutter, Ruby, Rust, etc. Never assume a specific ecosystem.

**OUTPUT LANGUAGE RULE:** Match the language of the existing `CLAUDE.md` if it exists. Otherwise default to English.

---

## Step 0 — Context7 Kontrolü

`~/.claude/settings.json` ve `.claude/settings.json` dosyalarına bak. İkisinde de `context7` anahtarı yoksa **dur** ve kullanıcıya şunu göster:

```
⚠️  Context7 MCP bulunamadı. Devam etmeden önce kur:

  claude mcp add context7 -- npx -y @upstash/context7-mcp@latest

Kurulumdan sonra Claude Code'u yeniden başlat ve /init komutunu tekrar çalıştır.
```

Context7 kuruluysa devam et.

---

## Step 1 — Silent Codebase Discovery

Do NOT ask any questions yet. Analyze silently.

### 1.1 — Detect Ecosystems and Structure

First, read these if they exist — they often answer everything before a single question is asked:

- `README.md` / `README.rst` → project description, domain, purpose
- `CLAUDE.md` → existing rules to preserve
- `.env.example` / `.env.sample` → reveals integrations (OAuth providers, payment keys, storage, etc.)
- `docker-compose*.yml` / `Dockerfile` → infrastructure, services, env structure

Then identify the ecosystem by scanning for manifest files:

| Manifest file                                                  | Ecosystem               |
| -------------------------------------------------------------- | ----------------------- |
| `package.json`                                                 | JavaScript / TypeScript |
| `requirements.txt` / `pyproject.toml` / `setup.py` / `Pipfile` | Python                  |
| `go.mod`                                                       | Go                      |
| `build.gradle` / `build.gradle.kts` / `pom.xml`                | Java / Kotlin / Android |
| `pubspec.yaml`                                                 | Dart / Flutter          |
| `Cargo.toml`                                                   | Rust                    |
| `Gemfile`                                                      | Ruby                    |
| `*.csproj` / `*.sln`                                           | .NET / C#               |
| `mix.exs`                                                      | Elixir                  |

Glob these files at root and in subdirectories (exclude build/vendor/node_modules/.gradle dirs, max 15 files total).

Then determine:

- **Monorepo?** → multiple manifest files in subdirs, OR workspace config: `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `lerna.json`, `cargo workspaces`, `go.work`
- **Platforms present:** label each app/module as: `backend-api` / `web-frontend` / `admin-panel` / `landing` / `mobile-ios` / `mobile-android` / `mobile-cross-platform` / `cli` / `library` / `other`
- **Directory layout:** note the actual folder names (apps/, services/, packages/, modules/, etc.)

### 1.2 — Per-Platform Stack Detection

For each platform, read its manifest and config files. Extract what exists — do not assume anything not found.

**JavaScript / TypeScript** — read `package.json` dependencies:

| Category                | Common indicators                                                       |
| ----------------------- | ----------------------------------------------------------------------- |
| Backend framework       | `nestjs`, `express`, `fastify`, `hono`, `koa`                           |
| Frontend framework      | `next`, `nuxt`, `remix`, `astro`, `vite`+`react/vue/svelte`             |
| Mobile (cross-platform) | `react-native`, `expo`                                                  |
| ORM / DB client         | `prisma`, `drizzle-orm`, `typeorm`, `mongoose`, `kysely`                |
| Auth                    | `passport`, `next-auth`, `lucia`, `better-auth`, `jose`, `jsonwebtoken` |
| Cache                   | `ioredis`, `redis`, `@upstash/redis`                                    |
| Queue                   | `bullmq`, `bee-queue`, `pg-boss`                                        |
| Storage                 | `@aws-sdk/client-s3`, `@google-cloud/storage`, `minio`                  |
| Payment                 | `stripe`, `@paddle/paddle-node-sdk`, `@lemonsqueezy/lemonsqueezy.js`    |
| CSS                     | `tailwindcss`, `styled-components`, `@emotion/react`, `unocss`          |
| UI components           | `@radix-ui`, `@mui/material`, `antd`, `shadcn`                          |
| State                   | `zustand`, `jotai`, `@reduxjs/toolkit`, `pinia`, `nanostores`           |
| Data fetching           | `@tanstack/react-query`, `swr`, `apollo-client`, `urql`                 |
| Forms                   | `react-hook-form`, `formik`, `vee-validate`                             |
| Testing                 | `jest`, `vitest`, `playwright`, `cypress`, `@testing-library`           |
| i18n                    | `next-intl`, `nestjs-i18n`, `i18next`, `vue-i18n`                       |
| Real-time               | `socket.io`, `ws`, `@supabase/realtime`, `ably`, `pusher`               |
| Notifications           | `firebase-admin`, `resend`, `nodemailer`, `@sendgrid/mail`              |

**Python** — read `requirements.txt` / `pyproject.toml` / `Pipfile`:

| Category      | Common indicators                                           |
| ------------- | ----------------------------------------------------------- |
| Web framework | `fastapi`, `django`, `flask`, `litestar`, `starlette`       |
| ORM           | `sqlalchemy`, `tortoise-orm`, `django-orm`, `peewee`        |
| Auth          | `python-jose`, `authlib`, `django-allauth`, `fastapi-users` |
| Task queue    | `celery`, `dramatiq`, `rq`, `arq`                           |
| Cache         | `redis`, `aioredis`                                         |
| Testing       | `pytest`, `unittest`, `hypothesis`                          |
| HTTP client   | `httpx`, `requests`, `aiohttp`                              |
| Validation    | `pydantic`, `marshmallow`, `attrs`                          |
| Storage       | `boto3`, `google-cloud-storage`                             |

**Go** — read `go.mod`:

| Category      | Common indicators                            |
| ------------- | -------------------------------------------- |
| Web framework | `gin`, `echo`, `fiber`, `chi`, `gorilla/mux` |
| ORM           | `gorm`, `sqlx`, `ent`                        |
| Auth          | `golang-jwt`, `casbin`                       |
| Queue         | `asynq`, `machinery`, `watermill`            |

**Java / Kotlin / Android** — read `build.gradle` / `pom.xml`:

| Category    | Common indicators                             |
| ----------- | --------------------------------------------- |
| Framework   | `spring-boot`, `quarkus`, `micronaut`, `ktor` |
| Android SDK | `compileSdk`, `minSdk` in build.gradle        |
| ORM         | `hibernate`, `jpa`, `exposed`, `room`         |
| DI          | `dagger`, `hilt`, `koin`, `spring`            |
| Network     | `retrofit`, `okhttp`, `ktor-client`           |
| Compose     | `compose`, `compose-ui`                       |

**Dart / Flutter** — read `pubspec.yaml`:

| Category         | Common indicators                                                  |
| ---------------- | ------------------------------------------------------------------ |
| Platform targets | `flutter.platforms` or presence of `android/`, `ios/`, `web/` dirs |
| State management | `bloc`, `riverpod`, `provider`, `get`, `mobx`                      |
| Network          | `dio`, `http`                                                      |
| Navigation       | `go_router`, `auto_route`                                          |
| Local DB         | `drift`, `hive`, `isar`, `sqflite`                                 |
| DI               | `get_it`, `injectable`                                             |

Also scan for these regardless of ecosystem:

- DB schema files: `prisma/schema.prisma`, `db/schema.rb`, `alembic/versions/`, `migrations/`, `*.sql` → extract entity/model names
- `.env.example` / `.env.sample` → env var keys reveal integrations
- `docker-compose*.yml` → infrastructure (postgres, redis, rabbitmq, kafka, etc.)
- `Dockerfile` → runtime, exposed ports, env structure
- `CLAUDE.md` (existing) → rules to preserve

### 1.3 — Internal Discovery Summary (Do Not Show)

Build a mental map:

```
Ecosystems found: [JS/TS | Python | Go | Kotlin | Dart | ...]
Monorepo: yes/no — layout: apps/ / services/ / packages/ / ...
Platforms:
  backend-api: [framework, language, ORM, auth, cache, queue]
  web-frontend: [framework, CSS, state, forms, data-fetching]
  mobile: [Flutter/RN/native, state, navigation]
  ...
Multi-tenant signals: [tenantId in schema? workspace model? subdomain?]
Auth signals: [which lib, which flows detectable]
Domain entities: [User, Order, Product, ...] ← from schema/models
Integrations: [S3, Stripe, Firebase, Redis, ...]
Existing CLAUDE.md: yes/no + rules to preserve
```

---

## Step 2 — Questions (ONLY If Genuinely Unanswerable)

After full codebase scan, identify what you still do not know. Then ask — but only if it matters for generating DOCS.

**Rules:**

- One question per message
- Multiple choice when possible
- If you can infer it from any file (README, .env.example, schema, manifest, Dockerfile, CI config, source code) → **do not ask**
- If no questions remain → **skip this step entirely, proceed to Step 3**

**The only two things code cannot tell you:**

**Q-DOMAIN** — ask only if README/CLAUDE.md does not describe the business purpose:

> What does this project do and who uses it? (One sentence is enough.)

**Q-CONVENTIONS** — ask only if codebase shows inconsistent patterns where you cannot determine which is the rule and which is the mistake:

> Are there team conventions I should know that aren't obvious from the code?

That's it. Never ask about auth flows, framework versions, env names, monorepo structure, database choice, or anything else visible in the codebase.

---

## Step 3 — Context7 Library Verification

From the discovery summary, pick **3–5 libraries** that are:

- Central to the architecture (ORM, framework, auth, queue, payment)
- New, recently updated, or likely to have version-specific API differences
- Used in non-standard ways in this project

**Skip if trivial** (standard HTTP clients, basic utility libs, well-known CRUD patterns).

For each:

1. `resolve-library-id` → find the library
2. `query-docs` → query focused on the specific usage pattern in this project

**MAX 5 queries total.** Collect as "Library API Notes" for DOCS content.

---

## Step 4 — Generate DOCS/ Files

Create only the files this project needs. Do not create files for things not present.

### 4.1 — Determine Which Files to Create

| File                                     | Create if...                                                  |
| ---------------------------------------- | ------------------------------------------------------------- |
| `DOCS/01-backend-patterns.md`            | any backend platform exists                                   |
| `DOCS/02-frontend-patterns.md`           | any web/admin/landing platform exists                         |
| `DOCS/03-mobile-patterns.md`             | any mobile platform exists                                    |
| `DOCS/core-systems/auth-and-security.md` | auth detected                                                 |
| `DOCS/core-systems/workspace-tenant.md`  | multi-tenant pattern detected                                 |
| `DOCS/domains/media-and-storage.md`      | file storage detected                                         |
| `DOCS/domains/notifications.md`          | email or push notification detected                           |
| `DOCS/domains/payments-and-billing.md`   | payment library detected                                      |
| `DOCS/domains/realtime.md`               | websocket/realtime library detected                           |
| `DOCS/platforms/{name}.md`               | for each distinct platform, if platform-specific rules differ |

Number files sequentially starting from `01-` so the `architecture` skill's table stays consistent.

### 4.2 — Content Format Rules

The `architecture` skill reads these files and picks the right branch for each feature. Use the format that best communicates each rule — not a fixed template.

**Structural rules:**

- Top-level sections: `## 1. Topic` → `### 1.1 Subtopic` (numbered, hierarchical)
- Language: definitive — "MUST", "FORBIDDEN", "always", "never" — never "should" or "can"
- Content sourced from: codebase patterns (scan 2–3 representative files per topic, read briefly), user answers (Step 2), Context7 findings (Step 3), stack best practices

**Use each format for the right job:**

**Prose paragraphs** — for rules that apply universally, no branching:

```markdown
All endpoints MUST return responses wrapped in `{ data: T }`. The 204 No Content response is the only exception.
```

**ASCII decision trees** — for conditional flows with 2+ branches:

```
WHICH DATA FETCHING STRATEGY?
│
├─► List page (paginated)
│   ├── Use TanStack Query with infinite scroll
│   └── Prefetch on server component
│
└─► Single resource
    ├── Server Component if SEO required
    └── useQuery hook otherwise
```

**Tables** — for mappings, comparisons, naming conventions:

```markdown
| Operation | HTTP Method | Response                                    |
| --------- | ----------- | ------------------------------------------- |
| List      | GET         | 200 + `{ data: T[], meta: PaginationMeta }` |
| Create    | POST        | 201 + `{ data: T }`                         |
```

**Code blocks** — for concrete examples (import paths, config, key formats):

```typescript
// Cache key format
`cache:${scope}:${tenantId}:${resource}`;
```

**Callouts** — for critical warnings the architecture agent must not miss:

```markdown
> [!CAUTION]
> SSE endpoints MUST use `@RawResponse()`. TransformInterceptor breaks the stream.

> [!IMPORTANT]
> Never use `req.ip` directly — always use `extractClientIp(req)`.
```

---

## Step 5 — Update .gitignore

Read `.gitignore` if it exists.

Add the following entries if they are not already present:

```
# Claude Code — local overrides (personal, never commit)
.claude/settings.local.json

# JIT Slice Development — generated plans (project-specific, optionally commit)
# Remove the next line if you want to track PLANS/ in git
PLANS/
```

> **Note:** `DOCS/` is intentionally NOT gitignored — it is generated once by `/init` and then committed as part of the project's architecture documentation.

If `.gitignore` does not exist, create it with the entries above.

---

## Step 6 — Update CLAUDE.md

Read existing `CLAUDE.md` if present. Preserve all existing content. Add or update these sections:

1. **Project Description** — from Q-DOMAIN
2. **Technology Stack** — structured table from Step 1 (actual findings, not assumptions)
3. **Project Structure** — actual directory layout
4. **Environments** — from Q-ENV or inferred
5. **JIT Slice Development Workflow** — the four phases (analyze → architecture → spec → implement)
6. **Core Rules** — from Q-CONVENTIONS + patterns detected

If no `CLAUDE.md` exists, create it from scratch.

---

## Step 7 — Update architecture/SKILL.md DOCS Reading Table

Read `.claude/skills/architecture/SKILL.md`.

Find the DOCS file selection table (Step 2.1). Replace it to reflect **only the files that were actually created** in Step 4, with correct conditions for this project:

```markdown
| File                           | Read if...                      |
| ------------------------------ | ------------------------------- |
| `DOCS/01-backend-patterns.md`  | feature touches backend API     |
| `DOCS/02-frontend-patterns.md` | feature touches web or admin UI |
| ...                            |
```

---

## Step 8 — Summary

```
✅ Project initialized for JIT Slice Development!

📁 Generated DOCS/
├── 01-backend-patterns.md
├── 02-frontend-patterns.md
└── core-systems/auth-and-security.md
    ... (actual list)

📝 Updated
├── .gitignore
├── CLAUDE.md
└── .claude/skills/architecture/SKILL.md

🚀 Ready. Start your first feature:
/analyze {feature description}
```
