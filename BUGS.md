# Bug Report – IQ Taskboard Assessment

## Overview

This report documents issues identified through functional testing and source-code inspection of the IQ Taskboard application.

Findings are prioritized based on security impact, data integrity, and user experience. Observations whose expected behavior depends on business requirements are listed separately and are not presented as confirmed defects.

---

## Finding 1: Viewer task updates are not correctly rejected

**Priority:** P1 – High
**Category:** Security
**Severity:** High
**Status:** Runtime defect reproduced; attempted fix remains unverified

**Location:** `src/app/api/tasks/[id]/route.ts`

**Code references:**
- PATCH handler: authentication and task retrieval, approximately lines 17–26.
- Authorization checks and update logic, approximately lines 29–40.

**Description:**

During the initial runtime testing, an authenticated Viewer was able to update a task despite the expected restriction on Viewer permissions.

The PATCH handler was subsequently modified to include project membership and role authorization checks before updating the task. However, the automated test continues to return HTTP 200 instead of the expected HTTP 403. Therefore, the authorization issue is not yet considered resolved.

**Steps to reproduce:**

1. Log in as `dev@example.com`, a Viewer in Q3 Launch.
2. Identify a task in the Q3 Launch project.
3. Send a PATCH request to `/api/tasks/{taskId}` using the Viewer authentication token.
4. Include a task update, such as a changed title.
5. Inspect the HTTP response and verify whether the change persists.

**Actual result:**

- Initial manual runtime testing returned HTTP 200 OK and the changes persisted.
- After the attempted code change, the automated test still reports HTTP 200 instead of HTTP 403.

**Expected result:**

The API should verify project membership and role permissions before updating a task. A Viewer should receive HTTP 403 Forbidden when attempting to update a task.

**Evidence:**

- Initial runtime PATCH response: `200 OK`.
- Automated test: `a viewer cannot update a task`.
- Latest test result: expected `403`, received `200`.
- The PATCH handler has since been modified with authorization checks, but the automated test failure remains unresolved.

**Recommended action:**

Investigate why the PATCH test still receives HTTP 200. Verify that the test uses the Viewer token, targets the correct application instance, and executes the updated route handler. Retain the expected 403 assertion.

---

## Finding 2: Login email matching is case-sensitive

**Priority:** P2 – Medium
**Category:** Security / Authentication consistency
**Severity:** Medium
**Status:** Confirmed through source inspection and runtime testing

**Location:** `src/app/api/auth/login/route.ts`

**Code reference:**
- Line 16: Email lookup.

**Related files:**
- `src/schemas/auth.ts`
- `prisma/schema.prisma`
- `prisma/migrations/20260101000000_init/migration.sql`

**Description:**

The login endpoint uses the email address as submitted when querying the database. No lowercase normalization is performed by the inspected login handler or validation schema.

As a result, an otherwise valid email address entered with different capitalization does not match the stored lowercase email address.

**Steps to reproduce:**

1. Log in using `dev@example.com` and the valid password.
2. Log out.
3. Attempt login using `DEV@EXAMPLE.COM` with the same password.
4. Attempt login using `Dev@example.com` with the same password.

**Actual result:**

The lowercase email succeeds, while the uppercase and mixed-case variants return invalid credentials.

**Expected result:**

If email authentication is intended to be case-insensitive, capitalization differences should not prevent a valid user from logging in.

**Evidence:**

- Source inspection: `prisma.user.findFirst({ where: { email } })` at line 16.
- Runtime testing: uppercase and mixed-case login attempts failed while the lowercase email succeeded.

**Recommended action:**

Normalize email addresses consistently during registration and login. Ensure that uniqueness is also enforced consistently to prevent duplicate accounts with different email capitalization.

---

## Additional observations requiring requirement confirmation

These observations were identified during testing or source inspection. They should not be classified as confirmed defects until the expected product behavior is established.

### Observation 3: Member task deletion permissions

**Category:** Security
**Status:** Behavior observed – Permission policy requires confirmation

**Location:** `src/app/api/tasks/[id]/route.ts`, DELETE handler, approximately lines 49–55.

**Description:**

The DELETE endpoint checks project membership and calls `canEditTasks` before deleting a task. The inspected authorization logic permits roles that can edit tasks, including Members.

A Member was reportedly able to delete a task created by an Admin. This behavior may be valid if Members are allowed to delete any task within their project.

**Expected behavior to confirm:**

Should Members be allowed to delete all tasks in their project, or only tasks they created?

**Recommended action:**

Confirm the intended deletion policy. If Members should only delete their own tasks, add an ownership check before performing the deletion.

### Observation 4: Drag-and-drop task movement is not implemented

**Category:** Architecture
**Status:** Feature gap observed – Requirement confirmation needed

**Location:**
- `src/app/projects/[id]/page.tsx`
- `src/components/StatusColumn.tsx`
- `src/components/TaskCard.tsx`
- `src/components/TaskDetail.tsx`

**Description:**

The inspected board components do not show a drag-and-drop implementation or an `onDragEnd` handler. Task status can instead be changed through the task detail interface, which uses the PATCH endpoint.

This is a potential feature gap only if drag-and-drop movement is part of the expected product requirements.

**Expected behavior to confirm:**

Should users be able to move tasks between status columns by dragging and dropping them?

**Recommended action:**

Confirm the expected board interaction. If drag-and-drop is required, implement and test movement between status columns, including persistence and authorization.

---

## Summary

| Finding | Category | Priority | Status |
|---|---|---|---|
| Viewer task update authorization | Security | P1 – High | Reproduced; attempted fix unverified |
| Case-sensitive email login | Authentication consistency | P2 – Medium | Confirmed |
| Member task deletion permissions | Security | To be determined | Policy confirmation needed |
| Drag-and-drop task movement | Architecture | To be determined | Requirement confirmation needed |

## Testing and evidence notes

The findings are based on source inspection and runtime checks performed during the assessment.

Before final submission:

1. Include sanitized API request and response evidence for the confirmed findings.
2. Include reproducible curl commands without exposing authentication tokens, cookies, or passwords.
3. Investigate the remaining Viewer PATCH test failure.
4. Confirm the intended permission policy for Member task deletion.
5. Confirm whether drag-and-drop is an explicit product requirement.
6. Keep `TEST_OUTPUT.md` separate for Part 2 test execution results.

No database reset or seed-data modification was performed intentionally during this investigation.
