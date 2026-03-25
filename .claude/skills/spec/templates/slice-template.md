# Slice {N}: {Task Name from todo.md}

**Goal:** {What will be working when this slice is complete? 1-2 sentences.}
**Dependency:** {Which previous slice must be completed? If none: None}

---

## Mevcut Kod Bağlamı

> spec skill tarafından codebase taranarak doldurulur.
> implement agent bu listeye göre çalışır — tahmin yapmaz.

### Backend

| Dosya                                                   | Durum                 | Not                     |
| ------------------------------------------------------- | --------------------- | ----------------------- |
| `apps/backend/src/modules/{domain}/{domain}.service.ts` | mevcut - düzenlenecek | `createXxx()` eklenecek |

### Web

| Dosya                                                      | Durum               | Not |
| ---------------------------------------------------------- | ------------------- | --- |
| `apps/web/src/features/{domain}/hooks/use{Domain}Query.ts` | YOK - oluşturulacak | —   |

### Admin _(varsa)_

| Dosya | Durum                        | Not |
| ----- | ---------------------------- | --- |
| —     | Bu slice Admin'i etkilemiyor | —   |

### Landing _(varsa)_

| Dosya | Durum                          | Not |
| ----- | ------------------------------ | --- |
| —     | Bu slice Landing'i etkilemiyor | —   |

### Shared Packages

| Paket              | Dosya          | Durum                        |
| ------------------ | -------------- | ---------------------------- |
| `shared-types`     | `src/enums.ts` | mevcut — `XxxEnum` eklenecek |
| `shared-constants` | —              | Bu slice etkilemiyor         |

---

## Architecture Rules Specific to This Slice

> Only the parts of spec-registry.md relevant to this slice. The `implement` agent does not read DOCS — this is its sole reference.

- [Endpoint & Guard Structure rule from spec-registry]
- [Frontend Data Fetching rule from spec-registry — if applicable]
- [Other relevant rules from spec-registry]

---

## Files to Create / Modify

### Backend

- [ ] `apps/backend/src/modules/{feature}/{feature}.module.ts` — {new / update}
- [ ] `apps/backend/src/modules/{feature}/{feature}.controller.ts` — {new / update}
- [ ] `apps/backend/src/modules/{feature}/{feature}.service.ts` — {new / update}
- [ ] `apps/backend/src/modules/{feature}/dto/create-{feature}.dto.ts` — {new}
- [ ] `apps/backend/src/modules/{feature}/dto/{feature}-response.dto.ts` — {new}
- [ ] `apps/backend/prisma/migrations/...` — {if migration needed}

### Frontend — Web (if applicable)

- [ ] `apps/web/src/features/{feature}/hooks/use{Feature}Query.ts` — {new}
- [ ] `apps/web/src/features/{feature}/hooks/use{Feature}Mutation.ts` — {new}
- [ ] `apps/web/src/app/(dashboard)/{feature}/page.tsx` — {new / update}
- [ ] `apps/web/src/components/{Feature}List.tsx` — {new}

### Frontend — Admin (if applicable)

- [ ] `apps/admin/src/features/{feature}/...`

---

## Backend Tasks

- [ ] Create Prisma migration: `pnpm prisma migrate dev --name {feature}`
- [ ] Write DTO: `Create{Feature}Dto` — class-validator + `@saas/shared-constants` regex
- [ ] Write Response DTO: `{Feature}ResponseDto` — sensitive fields removed
- [ ] Write Service method: `{methodName}()` — scoped with tenantId
- [ ] Write Controller endpoint: `{METHOD} {path}` — guard + swagger + i18n
- [ ] Add i18n keys: `src/core/i18n/en/` and `src/core/i18n/tr/`

## Frontend Tasks (if applicable)

- [ ] Write `use{Feature}Query.ts` hook — TanStack Query `useQuery`
- [ ] Write `use{Feature}Mutation.ts` hook — `useMutation` + invalidateQueries
- [ ] Write page component — Server Component + prefetch
- [ ] If form: React Hook Form + Zod schema + use `<Controller>`

---

## API Contract for This Slice

> The endpoint(s) belonging to this slice are copied here from spec-registry.md.

```
{METHOD} {path}
Guard: {guard order}
Body: {dto}
Response: {status} { data: {response dto} }
Errors: {error codes}
```

---

## Acceptance Criteria

- [ ] `pnpm lint` passes without errors
- [ ] `pnpm build --filter={affected package}` passes without errors
- [ ] Manual verification: {How do you prove this slice works? e.g.: "POST /api/v1/xxx endpoint returns 201"}
