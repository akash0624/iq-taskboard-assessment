# Test Execution Report

## 1. Baseline Test Results

**Command:** `npm test`

**Results before changes:**
- Total tests: 3
- Passed: 1
- Failed: 2

**Failures:**
1. `a viewer cannot update a task` — expected HTTP 403, received HTTP 200.
2. `a viewer cannot create a task` — expected HTTP 401, received HTTP 403.

## 2. Changes Made

1. Updated the Viewer task-creation test to expect HTTP 403 because the user is authenticated but does not have permission to create tasks.
2. Added project-membership and role authorization checks to the task PATCH handler to prevent unauthorized task updates.

## 3. Test Results After Changes

**Command:** `npm test`

**Results:**
- Total tests: 3
- Passed: 2
- Failed: 1

**Remaining failure:**
- `a viewer cannot update a task` — expected HTTP 403, received HTTP 200.

**Status:** The remaining failure is unresolved and requires further investigation. The expected 403 assertion has been retained.

### Latest Test Run

**Command:** `npm test`

**Results:**
- Total tests: 4
- Passed: 3
- Failed: 1

**Passing tests:**
1. `an unauthenticated user cannot update a task` — received HTTP 401 as expected.
2. `a viewer cannot create a task` — received HTTP 403 as expected.
3. `a member can create a task` — passed.

**Remaining failure:**
- `a viewer cannot update a task` — expected HTTP 403, received HTTP 200.

**Status:** The Viewer update authorization issue remains unresolved. The test is retained as evidence of the defect.