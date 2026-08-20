---

name: integration-test-and-verify

description: Given a user story referenced in a PR comment, retrieves the corresponding acceptance criteria, derives integration test scenarios, creates isolated tests with mocked database and external dependencies, executes the tests against the changed implementation, and reports coverage, failures, and verification results.

---

# Integration Test and Verify

## Overview

Validates an implementation against the acceptance criteria of a user story.

The skill works from the **behavioral contract first**, rather than deriving tests solely from the implementation.

The workflow is:

```text
PR Comment
    │
    ▼
Identify User Story
    │
    ▼
Fetch User Story
    │
    ▼
Extract Acceptance Criteria
    │
    ▼
Map Acceptance Criteria → Test Scenarios
    │
    ▼
Inspect Implementation & Existing Tests
    │
    ▼
Create Integration Tests
    │
    ▼
Mock Database / External Dependencies
    │
    ▼
Run Tests
    │
    ▼
Analyze Failures
    │
    ▼
Generate Verification Report
```

## When to Use

* When a PR comment references a user story
* When acceptance criteria need to be validated against an implementation
* When integration tests need to be generated automatically
* When database connections should be isolated from test execution
* When external services need to be mocked
* When an agent needs to independently verify another agent's implementation
* Before approving a PR where integration behavior is important

## Core Principle

**Acceptance criteria are the source of truth.**

Do not derive expected behavior solely from the implementation.

The implementation may be wrong while still appearing internally consistent.

The test-generation process should therefore follow:

```text
Acceptance Criteria
        ↓
Expected Behavior
        ↓
Test Scenario
        ↓
Test Implementation
        ↓
Actual Behavior
        ↓
Pass / Fail
```

## Inputs

The skill expects:

### Required

* PR identifier or PR context
* PR comment containing or referencing the user story
* Repository/workspace containing the implementation

### Optional

* Existing test suite
* Test configuration
* Database schema
* API specifications
* Mock fixtures
* CI configuration
* Existing integration-test utilities

If the user story cannot be identified, stop and report:

> Unable to identify the user story from the PR comment.

Do not guess the story.

---

# Step 1: Parse the PR Comment

Inspect the PR comment and identify:

* User story ID
* User story URL
* Ticket/reference ID
* Relevant feature description
* Any explicit testing instructions

Examples:

```text
Story: US-1234
```

or:

```text
Implements ABC-456
```

or:

```text
As a customer, I want...
```

Extract the canonical identifier whenever possible.

Do not treat unrelated PR discussion as acceptance criteria.

---

# Step 2: Fetch the User Story

Retrieve the referenced user story from the connected project-management system.

The story should provide:

* Story title
* Description
* Acceptance criteria
* Business rules
* Constraints
* Dependencies
* Relevant examples

Prefer the canonical story source over duplicated text in the PR.

If the story cannot be retrieved:

```text
STATUS: BLOCKED

Reason:
The referenced user story could not be retrieved.

Action:
Provide access to the story or provide its acceptance criteria.
```

Do not generate tests from assumptions.

---

# Step 3: Extract Acceptance Criteria

Normalize the acceptance criteria into independently testable behaviors.

Example:

```text
AC1:
Given a valid customer ID,
when the customer requests their order history,
then the API returns their orders.

AC2:
Given an unknown customer ID,
when the customer requests their order history,
then the API returns 404.

AC3:
Given a database timeout,
when the request is processed,
then the API returns 503.
```

Convert each criterion into a testable contract:

| AC  | Condition        | Action         | Expected Result |
| --- | ---------------- | -------------- | --------------- |
| AC1 | Valid customer   | Request orders | Orders returned |
| AC2 | Unknown customer | Request orders | 404             |
| AC3 | DB timeout       | Request orders | 503             |

Every acceptance criterion should map to at least one test.

---

# Step 4: Inspect the Implementation

Before generating tests, inspect:

* Changed files
* Application entry points
* API routes
* Services
* Repositories
* Database access
* External service calls
* Existing integration tests
* Existing fixtures
* Existing mocks
* Test configuration

The purpose is to determine the correct integration boundary.

Do not automatically mock everything.

The test should exercise as much of the application's real behavior as practical while isolating unavailable or unsafe dependencies.

---

# Step 5: Determine the Test Boundary

Classify dependencies.

### Keep Real

Prefer keeping these real when practical:

* Application routing
* Request validation
* Service orchestration
* Serialization
* Business logic
* Error handling

### Mock

Mock dependencies when they are:

* External databases
* Production credentials
* Third-party APIs
* Payment systems
* External queues
* Unavailable infrastructure
* Expensive or destructive services

Example:

```text
HTTP Request
    ↓
Real API
    ↓
Real Service
    ↓
Mock Repository
    ↓
Mock Database
```

The test should therefore validate integration between application components without requiring production infrastructure.

---

# Step 6: Mock Database Connections

Never allow generated tests to connect to a production database.

Before running tests:

* Inspect database configuration
* Identify connection creation points
* Identify repository/data-access interfaces
* Replace the production connection with a controlled test double
* Ensure credentials cannot leak into test execution

Preferred hierarchy:

```text
Existing test database
        ↓
Existing repository mock
        ↓
In-memory database
        ↓
Dedicated test container
        ↓
Mock connection
```

Use the project's existing testing infrastructure when available.

Do not introduce a new mocking framework if an existing one already solves the problem.

---

# Step 7: Generate Test Cases

Generate tests from acceptance criteria.

For every criterion create:

### Positive Case

Expected successful behavior.

### Negative Case

Invalid or rejected input where applicable.

### Boundary Case

Relevant limits or edge conditions.

### Dependency Failure

Database/external dependency failure where the acceptance criteria or implementation makes it relevant.

### Regression Case

For bug fixes, reproduce the original failure and verify the fix.

Example:

```text
TC-001
Acceptance Criteria: AC1
Scenario: Retrieve orders for valid customer
Input: customer_id=123
Expected: HTTP 200 and expected orders
Dependencies: Mock repository
```

```text
TC-002
Acceptance Criteria: AC2
Scenario: Retrieve orders for unknown customer
Input: customer_id=999
Expected: HTTP 404
Dependencies: Mock repository returning not-found
```

---

# Step 8: Test Case Quality Gate

Before writing tests, verify:

* [ ] Every acceptance criterion has coverage
* [ ] Expected behavior comes from the acceptance criterion
* [ ] Tests exercise real application behavior
* [ ] Assertions validate outcomes
* [ ] Failure paths are covered
* [ ] Relevant boundaries are covered
* [ ] Database access is isolated
* [ ] External services are isolated
* [ ] No production credentials are required
* [ ] Existing test utilities are reused

Avoid tests that merely verify that a mocked function was called.

Prefer:

```text
Request → Application → Dependency → Response
```

over:

```text
Call function → Assert mock called
```

when the purpose is integration verification.

---

# Step 9: Implement the Tests

Place generated tests according to the repository's existing conventions.

Examples:

```text
tests/integration/
tests/e2e/
integration_tests/
```

Do not create a new test structure if the repository already has one.

Follow existing:

* Naming conventions
* Fixtures
* Test runners
* Setup/teardown
* Mocking patterns
* Environment configuration

Keep generated tests focused.

Do not modify production code merely to make a generated test pass.

---

# Step 10: Execute the Tests

Run the narrowest relevant test suite first.

Example:

```bash
pytest tests/integration/test_orders.py
```

Then, when appropriate:

```bash
pytest tests/integration/
```

Finally, run the project's broader suite if practical.

Record:

* Command
* Number of tests
* Passed
* Failed
* Skipped
* Errors
* Execution time

Never claim tests passed without actually executing them.

---

# Step 11: Analyze Failures

For every failure determine whether it is:

### Implementation Failure

The implementation violates the acceptance criteria.

```text
AC3 expected HTTP 503.
Implementation returned HTTP 500.
```

### Test Failure

The generated test incorrectly models the acceptance criteria.

### Environment Failure

The test could not execute because of:

* Missing dependency
* Incorrect configuration
* Infrastructure failure
* Missing credentials
* Test environment problem

### Existing Regression

The change breaks behavior that previously worked.

Do not automatically modify the implementation when a test fails.

First classify the failure.

---

# Step 12: Iterate

If a generated test fails:

```text
Failure
  ↓
Classify
  ↓
Implementation bug?
  ├── Yes → Report
  │
  └── No
       ↓
Test problem?
  ├── Yes → Correct test
  │
  └── No
       ↓
Environment problem?
           ↓
         Report
```

The agent must not weaken assertions simply to make the suite pass.

Do not:

* Remove failing cases
* Lower assertions
* Mock away the behavior under test
* Convert failures to skips without justification
* Change acceptance criteria to match implementation

---

# Step 13: Acceptance-Criteria Coverage

Produce a coverage matrix.

```markdown
| Acceptance Criterion | Test Cases | Result |
|---|---|---|
| AC1 | TC-001 | PASS |
| AC2 | TC-002 | PASS |
| AC3 | TC-003 | FAIL |
```

A story is fully verified only when every required acceptance criterion has a successful test result.

If an acceptance criterion cannot be tested:

```text
AC4 — NOT VERIFIED

Reason:
Required external payment service is unavailable in the test environment.

Impact:
The PR cannot receive full integration-test approval.
```

---

# Step 14: Generate Test Report

Produce:

```markdown
# Integration Test Report

## User Story

[Story ID and title]

## Acceptance Criteria

[List normalized acceptance criteria.]

## Test Coverage

| AC | Test Case | Status |
|---|---|---|
| AC1 | TC-001 | PASS |
| AC2 | TC-002 | PASS |
| AC3 | TC-003 | FAIL |

## Test Environment

- Database: Mocked
- External APIs: Mocked
- Application: Real
- Test runner: [runner]

## Execution

Command:

`[actual command]`

Results:

- Tests: [count]
- Passed: [count]
- Failed: [count]
- Skipped: [count]
- Errors: [count]

## Failures

### TC-003

**Acceptance Criterion:** AC3

**Expected:** [expected behavior]

**Actual:** [actual behavior]

**Classification:** Implementation failure

## Verification Summary

- Acceptance criteria covered: X/Y
- Acceptance criteria passed: X/Y
- Integration tests passed: X/Y
- Database isolated: Yes/No
- External dependencies isolated: Yes/No

## Verdict

**PASS / FAIL / BLOCKED**

[Explanation.]
```

---

# Safety and Isolation

Generated integration tests must never:

* Connect to production databases
* Modify production data
* Use production API credentials
* Send real payment requests
* Send destructive external requests
* Delete real resources
* Commit secrets
* Disable authentication merely to make tests pass

Before execution, verify that test configuration points to safe dependencies.

If this cannot be established:

```text
STATUS: BLOCKED

Tests were not executed because dependency isolation could not be verified.
```

---

# Definition of Done

The skill is complete when:

* [ ] PR comment was inspected
* [ ] User story was identified
* [ ] Canonical acceptance criteria were retrieved
* [ ] Every acceptance criterion was mapped to test coverage
* [ ] Existing test infrastructure was inspected
* [ ] Database dependencies were isolated
* [ ] External dependencies were isolated
* [ ] Integration tests were generated
* [ ] Tests were actually executed
* [ ] Failures were classified
* [ ] Acceptance-criteria coverage matrix was generated
* [ ] Test results were recorded
* [ ] No production dependency was accessed
* [ ] Final PASS / FAIL / BLOCKED verdict was produced

## Final Rule

**Never confuse "tests were generated" with "the implementation was verified."**

Verification requires:

```text
Acceptance Criteria
       ↓
Test Cases
       ↓
Executable Tests
       ↓
Actual Execution
       ↓
Evidence
       ↓
Verdict
```

A test that was not executed is a **test case**, not verification.
