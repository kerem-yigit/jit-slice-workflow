# Spec Registry — {Feature Name}

This file is the architectural contract between all agents. `spec` and `implement` agents do not read DOCS — everything they need must be present here completely.

---

## Shared Types & Interfaces

```typescript
// --- Input DTOs (derived from analysis.md User Stories + DOCS DTO patterns) ---

interface CreateXxxDto {
  name: string; // @IsString(), @Matches(REGEX_ORGANIZATION_NAME)
  price: number; // @IsNumber(), @IsPositive()
  description?: string; // @IsOptional(), @IsString()
}

interface UpdateXxxDto {
  // PartialType(CreateXxxDto) — all fields optional
}

// --- Response DTOs (sensitive fields removed) ---

interface XxxResponseDto {
  id: number;
  tenantId: number;
  name: string;
  price: number;
  createdAt: string;
  updatedAt: string;
}

// --- Pagination (for list endpoints) ---

interface PaginationMeta {
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}
```

---

## API Contracts

```
// Each endpoint: Method + Path + Guard order + Body + Response + Error codes

POST /api/v1/xxx
Guard: JwtAuthGuard → TenantGuard → @Roles(OWNER, ADMIN)
Body: CreateXxxDto
Response: 201 { data: XxxResponseDto }
Errors: 400 (validation), 403 (forbidden), 409 (already exists)

GET /api/v1/xxx
Guard: JwtAuthGuard → TenantGuard
Query: page, limit, search
Response: 200 { data: XxxResponseDto[], meta: PaginationMeta }

GET /api/v1/xxx/:id
Guard: JwtAuthGuard → TenantGuard
Response: 200 { data: XxxResponseDto }
Errors: 404 (not found)

PATCH /api/v1/xxx/:id
Guard: JwtAuthGuard → TenantGuard → @Roles(OWNER, ADMIN)
Body: UpdateXxxDto
Response: 200 { data: XxxResponseDto }
Errors: 400, 403, 404

DELETE /api/v1/xxx/:id
Guard: JwtAuthGuard → TenantGuard → @Roles(OWNER)
Response: 204 No Content
Errors: 403, 404
```

---

## Prisma Model Reference

```prisma
// tenantId FK is REQUIRED for tenant-bound tables
// createdAt and updatedAt are REQUIRED

model Xxx {
  id          Int      @id @default(autoincrement())
  tenantId    Int
  name        String
  price       Float
  description String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  tenant      Tenant   @relation(fields: [tenantId], references: [id])

  @@index([tenantId])
}
```

---

## Enums & Constants & Error Codes & Feature Flag

```typescript
// Error codes — to be added to @saas/shared-constants ErrorCodes
// XXX.NOT_FOUND        → "xxx.error.notFound"
// XXX.ALREADY_EXISTS   → "xxx.error.alreadyExists"
// XXX.FORBIDDEN        → "xxx.error.forbidden"

// Feature flag (if applicable)
// Flag key: "xxx_feature", Type: boolean, Default: false
// Plan mapping: Active for PRO and above
```

---

## Critical Architectural Constraints (REQUIRED)

> `spec` and `implement` agents read this file. They do not read DOCS. Each rule must be a concrete, definitive expression of the chosen branch for this feature.

### [Endpoint & Guard Structure]

[e.g.: All endpoints are protected with `/api/v1/...` prefix, `JwtAuthGuard → TenantGuard → RolesGuard` in order. `X-Tenant-Id` header is required.]

### [Frontend Data Fetching & Mutation]

[e.g.: Web platform → TanStack Query. Query: `useXxxQuery.ts`. Mutation: `useXxxMutation.ts`. Optimistic update applied.]

### [Cache Strategy] (if applicable)

[e.g.: Redis cache. Key: `xxx:{tenantId}`, TTL: 5min. Invalidated with `redisService.del(key)` after mutation.]

### [Other rule categories — chosen branches from DOCS decision trees]

[...]

### Library API Notes (Context7)

> Only for libraries whose current API the architecture agent verified with Context7. Deprecated methods, new API patterns, version-specific notes go here. Delete this section if Context7 was not queried.

[e.g.: "In TanStack Query v5, `useQuery({ queryKey, queryFn })` format is required. The old `useQuery(key, fn)` syntax has been removed."]
[e.g.: "In Prisma 6, `findUnique` only accepts unique fields in `where`. Use `findFirst` for composite keys."]

---

## Non-Functional Requirements (NFR) and Technical Edge Cases

> Write only items that are "Yes", delete the rest.

- **Rate limiting:** [Which endpoint, which tier (AUTH/WRITE/READ/CRITICAL), limit]
- **Scheduled task / Cron:** [Task, cron expression]
- **Websocket:** [Channel, broadcast event]
- **Email / Push notification:** [Trigger event, recipients, queue usage]
- **File upload:** [MIME types, max size MB, storage bucket]
- **Custom permission / Guard:** [New scope or guard name, justification]
- **Webhook (In/Out):** [Direction, retry policy, signature verification]
- **Audit log:** [Events to log, fields to log]
- **Idempotency:** [Transaction or Idempotency-Key header]
- **Race condition (Locking):** [Optimistic (version column) or Pessimistic (FOR UPDATE)]
