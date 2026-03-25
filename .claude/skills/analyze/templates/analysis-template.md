# [Feature/Bug/Integration Name] — Analysis Document

**Date:** [Today]
**Type:** [Feature / Bug Fix / Infra-Integration / Refactor]
**Status:** Analysis Complete

---

## Problem Statement

> 📌 **For all types.**

[What are we solving? What is the current state, what is the target state? 2-3 sentences — business language, not technical.]

---

## Business Value and Success Metric

> 📌 **For Feature type only. Delete this section for others.**

[What is the core benefit of this feature for users or the business? By what metric will we measure success?]

---

## Scope & Out of Scope

> 📌 **For all types.**

### In Scope

- [Thing to do 1]
- [Thing to do 2]

### Out of Scope (Deliberately Excluded)

- [Excluded item 1] — _Reason: [justification]_
- [Excluded item 2] — _Reason: [justification]_

---

## User Stories & Scenarios

> 📌 **For Feature type only. Delete this section for Bug Fix / Infra / Refactor.**

### User Stories

> **US-1:** As a [Role], I want to [action], because [motivation].
> **US-2:** As a [Role], I want to [action], because [motivation].

### Acceptance Criteria

**For US-1:**

- [ ] Criterion 1: [Expected behavior]
- [ ] Criterion 2: [Expected behavior]

### Main Flow (Happy Path)

1. User enters [X] screen.
2. Performs [Y] step.
3. On success, sees [W] result.

### UI/UX Flow

[Which page/modal/drawer, success/error states, loading/empty state]

---

## Role & Permission Matrix

> 📌 **For Feature type only. Delete this section for Bug Fix / Infra / Refactor.**

### Tenant Roles

| Action | Owner | Admin | Member        |
| ------ | ----- | ----- | ------------- |
| View   | ✅    | ✅    | ✅            |
| Create | ✅    | ✅    | ❌ (disabled) |
| Edit   | ✅    | ✅    | ❌            |
| Delete | ✅    | ❌    | ❌            |

### Admin Panel Roles (if applicable)

| Action                   | Platform Admin |
| ------------------------ | -------------- |
| Cross-tenant viewing     | ✅             |
| Configuration management | ✅             |

### Permission Scopes (if granular permission is needed)

| Scope             | Description | Default Roles        |
| ----------------- | ----------- | -------------------- |
| `resource:read`   | Viewing     | Owner, Admin, Member |
| `resource:create` | Creating    | Owner, Admin         |
| `resource:update` | Editing     | Owner, Admin         |
| `resource:delete` | Deleting    | Owner                |

---

## Bug Analysis

> 📌 **For Bug Fix type only. Delete this section for others.**

### Current Faulty Behavior

[What is happening now?]

### Expected Behavior

[What should be happening?]

### Reproduction Conditions

- Environment: [development / staging / production]
- User profile: [which user/tenant type]
- Steps: [1. ... 2. ... 3. ...]

### Impact Scope

[How many users/tenants are affected? Continuous or intermittent?]

### Regression Risk

[Areas that may be affected by side effects after the fix. Which flows should be tested?]

---

## Integration Analysis

> 📌 **For Infra / Integration type only. Delete this section for others.**

### Current State → Target State

|          | Current                                  | Target                            |
| -------- | ---------------------------------------- | --------------------------------- |
| Method   | [e.g.: SMTP / nodemailer]                | [e.g.: Resend API / SDK]          |
| Behavior | [e.g.: Mail sent, not added to audience] | [e.g.: Mail sent + audience sync] |

### Affected Flows

- [Which service, endpoint or user flow is affected]

### Error Scenarios and Fallback

| Scenario                | Behavior                                           |
| ----------------------- | -------------------------------------------------- |
| [e.g.: API timeout]     | [e.g.: Mail still sent, sync skipped, log written] |
| [e.g.: Partial failure] | [e.g.: Added to retry queue]                       |

### Migration / Rollout Plan

- Does existing behavior break during transition? [Yes/No — explain]
- Is rollback possible? [Yes/No — how?]

---

## Business Rules and Edge Cases

> 📌 **For all types. Delete irrelevant items.**

### Business Rules

- [e.g.: A contact that cannot be added to the audience will be retried on the next mail send.]

### Edge Cases

- [Edge case 1]: [How will it be handled?]
- [Edge case 2]: [How will it be handled?]

---

## Industry / Market Standards

> 📌 **Optional. Add only if research was done, otherwise delete this section.**

[How have competitors solved this problem? What are the best practices in the market?]

---

## Dependencies

> 📌 **For all types. Delete this section if there are no dependencies.**

### External Dependencies

[Third-party service, library, infrastructure dependency]

### Internal Dependencies

[Is there another feature or fix that must be completed first?]

---

## Non-Functional Requirements

> 📌 **For all types. Delete rows that do not apply.**

| NFR               | Detail                                         |
| ----------------- | ---------------------------------------------- |
| **Security/Risk** | [Business risks it carries]                    |
| **Performance**   | [Acceptable speed/latency expectation]         |
| **Compliance**    | [Is there a legal requirement like KVKK/GDPR?] |
| **Feature Flag**  | [Flag key, plan mapping — if applicable]       |

---

## Notes & Open Questions

[Topics not yet clarified, to be decided at a later stage]
