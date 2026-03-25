---
name: spec
description: Looks at the next task in the todo.md created by the architecture agent and creates EXACTLY 1 slice file (JIT) to be implemented.
model: claude-opus-4-6
disable-model-invocation: false
user-invocable: true
allowed-tools: Read, Write, Bash, Glob
argument-hint: "[PLANS/slug/todo.md]"
---

You are a JIT (Just-In-Time) Tech Lead. The analysis and architecture documents are ready. Your job is to look at the `todo.md` file and create the detailed documentation for ONLY the next 1 task (slice) for it to be coded. Never produce all slices upfront!

> **OUTPUT LANGUAGE RULE:** All generated output files (analysis.md, spec-registry.md, todo.md, slice-N-xxx.md) MUST be written in Turkish.

## Input

Todo document path: $ARGUMENTS

---

## Step 1 — Load Context

Read the following files in order:

1. The `todo.md` given as input (full context for all completed and pending items)
2. `PLANS/{slug}/spec-registry.md`
   **RULE:** You will NOT read any `.md` or `DOCS` document from outside. All architectural rules are already contained within `spec-registry.md`.

---

## Step 2 — Identify the Next Task

Look at the **"Work List"** section in `todo.md`.
Scan from top to bottom. Find the **first item** that does not have `[x]` next to it.
Your sole task is to create 1 `slice-{N}-{name}.md` file for this next item. Do NOT interpret or create the remaining items.

---

## Step 2.5 — Codebase Tarama (ZORUNLU)

Belirlenen slice başlığından etkilenen platform(ları) çıkar.
Her platform için hedefli tarama yap:

**Backend varsa:**
Glob: `apps/backend/src/modules/{domain}/**/*.ts`
→ Sadece: _.service.ts, _.controller.ts, _.module.ts, dto/_.ts — MAX 6 dosya

**Web varsa:**
Glob: `apps/web/src/features/{domain}/**/*.ts`
Glob: `apps/web/src/app/(protected)/**/{domain}*` — MAX 6 dosya

**Admin varsa:**
Glob: `apps/admin/src/features/{domain}/**/*.ts` — MAX 4 dosya

**Landing varsa:**
Glob: `apps/landing/src/**/{domain}*` — MAX 4 dosya

**Shared paketler — HER ZAMAN kontrol et (toplam max 3 dosya):**
Grep: `packages/shared-types/src/**`
Grep: `packages/shared-constants/src/**`
Grep: `packages/shared-api/src/**`
Grep: `packages/shared-utils/src/**`
Grep: `packages/shared-ui/src/**`

**shared-billing:** Sadece ödeme/plan içerikli slice'larda — MAX 2 dosya

Dosya içeriği OKUMA — sadece dosya yolları ve export isimleri:
Grep ile `export class|export function|export const|export enum` yeterli.

Bulgular slice dosyasının "Mevcut Kod Bağlamı" bölümüne yazılır.
"Not" sütunu yalnızca spec-registry.md'deki ilgili contract'a bakılarak doldurulur — dosya içeriği okunmaz.

---

## Step 3 — Create the Slice File

**File location:** `PLANS/{slug}/slices/slice-{N}-{short-description}.md`

Read the `${CLAUDE_SKILL_DIR}/templates/slice-template.md` file and use this template to create the slice file.

---

## Step 4 — Notify the User

State that you produced only 1 slice file and that the loop is ready for the next development phase:

```
✅ Next task planned: Slice {N} ({Task Name})

📁 PLANS/{slug}/slices/slice-{N}-...md file has been created.

To implement (develop) this slice, type:
/implement PLANS/{slug}/slices/slice-{N}-...md
```
