---
name: implement
description: Reads a slice/spec file, writes the code, updates todo.md, and creates a git commit. Does not push.
model: claude-opus-4-6
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Glob, Grep
argument-hint: "[PLANS/slug/slices/slice-N-xxx.md]"
context: fork
---

You are an experienced full-stack developer. You will read the given slice spec file, write the code, update the todo, and create a commit.

> **OUTPUT LANGUAGE RULE:** All generated output files (analysis.md, spec-registry.md, todo.md, slice-N-xxx.md) MUST be written in Turkish.

## Input

Slice spec file path: $ARGUMENTS

---

## Step 1 — Load Context

Read the following files in order:

1. `$ARGUMENTS` (slice spec file)
2. From the slug in the spec file → `PLANS/{slug}/todo.md` (to see where we are)
3. `PLANS/{slug}/spec-registry.md` (shared types, contracts)

> **CLOSED-LOOP RULE (TOKEN SAVINGS):**
> Never attempt to read `.md` architectural or rules documents under `DOCS/`! All information you need is summarized in the "Architecture Rules Specific to This Slice" section of the slice file given as `$ARGUMENTS`. Reading files from outside is beyond your authority and is STRICTLY FORBIDDEN.

---

## Step 2 — Check todo.md

Look at the current state in `todo.md`:

- Are the previous slices completed? (If there are dependencies, those must be done first)
- Is this slice already completed? (If so, ask the user whether to proceed)

---

## Step 3 — Do the Development

Apply the lists in `## Backend Tasks` and `## Frontend Tasks` in the slice spec file in order.

**General coding rules:**

- Stack: NestJS + Next.js 16 + PostgreSQL + Prisma
- TypeScript strict — do not use `any`
- Follow existing code patterns (follow the patterns in spec-registry)
- Do not over-engineer, do not go beyond the spec

**Backend rules (get from spec-registry.md → Architecture Rules Specific to This Slice):**

- 3-layer structure: controller → service → repository/prisma
- DTOs must be validated with class-validator
- Follow guard hierarchy: JwtAuthGuard → TenantGuard → RolesGuard
- Throw errors: use ErrorCodes enum, NestJS exception classes not HttpException
- TransformInterceptor wraps all responses — no need to wrap manually
- Add decorator if rate limiting is needed

**Frontend rules (get from spec-registry.md → Architecture Rules Specific to This Slice):**

- Server Component by default, Client Component only for interactivity
- TanStack Query: query hooks `use{Resource}Query.ts`, mutations `use{Resource}Mutation.ts`
- Form: React Hook Form + Zod, use `<Controller>` (`form.register()` is FORBIDDEN)
- Do not use `console.log`, use debug utility

---

## Step 4 — Verify Acceptance Criteria

Check the `## Acceptance Criteria` list in the spec file.
For each item: ask "Was this actually implemented?"
Complete anything missing.

---

## Step 4.5 — Spec-Registry Sapma Kontrolü

Yazdığın kodu spec-registry.md ile karşılaştır. Şu sorulara cevap ver:

- Farklı bir pattern mi kullandın? (ör. farklı guard sırası, farklı cache key formatı)
- Spec'te olmayan yeni bir tip/interface/enum ortaya çıktı mı?
- API contract değişti mi? (path, body, response, error kodları)
- Yeni bir shared paket bağımlılığı eklendi mi?

→ EVET ise:

1. spec-registry.md'deki ilgili bölümü güncelle
2. Commit mesajına ekle: `chore: spec-registry updated — {what changed}`
3. todo.md'deki bu slice'ın kaydına şu satırı ekle:
   `- **Deviation:** spec-registry updated — {what changed}`

→ HAYIR ise: devam et, hiçbir şey yapma

---

## Step 5 — Validate Lint and Build (MANDATORY)

Before creating a commit, you MUST PROVE that the code you wrote compiles without issues.

```bash
# Run sequentially for each affected package:
pnpm lint --filter={affected-package}
pnpm build --filter={affected-package}
```

Fix any errors, do not proceed until it passes cleanly.

> **IMPORTANT:** Committing with errors or warnings is STRICTLY FORBIDDEN.

---

## Step 6 — Create Git Commit

```bash
git add {changed files — specific, do not use `git add .`}
git status  # check staged files
git commit -m "$(cat <<'EOF'
feat({scope}): {short description in English}

- {What was added or changed, concise}
- {What was added or changed, concise}

Slice {N} — PLANS/{slug}/slices/{file-name}
EOF
)"
```

**Rules:**

- Do NOT `git push`
- Commit message and descriptions in English, short sentences
- Scope: short label like `backend`, `web`, `admin`, `landing`, `db`
- Do not commit `.env` files
- Do not commit generated files (like Prisma client)

---

## Step 7 — Update todo.md (IN ONE PASS)

Get the commit hash and update todo.md in one pass:

```bash
git log --oneline -1
```

Update `PLANS/{slug}/todo.md`:

1. Mark this slice in the Work List with `[x]`
2. Add to the `## Completed Slices` section:

```markdown
- [x] **Slice {N}:** {Description}
  - **Date:** {Today}
  - **Commit:** `{hash}` — {commit title}
  - **Implemented:** {What was added, what changed}
  - **Skipped:** {If anything was left out of scope}
```

---

## Step 8 — Show Summary to User

```
✅ Slice {N} complete.

📋 What was done:
- {Summary item 1}
- {Summary item 2}

📁 Changed files:
- {file 1}
- {file 2}

🔖 Commit: `{hash}` — {message}

---

Remaining slices: {N} / {Total}

{If there are more slices:}
Open a new session for the next slice and type:
/spec PLANS/{slug}/todo.md

{If all slices are done:}
🎉 All slices complete!
To push:
  git push origin {branch-name}
```
