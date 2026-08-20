---

name: pr-report-generation

description: Generates a structured, evidence-based pull request report from a code change, review findings, tests, verification results, and change metadata. Use after code review when you need a concise report suitable for a PR description, reviewer summary, or engineering record.

---

# PR Report Generation

## Overview

Generate a concise, decision-oriented PR report from the completed change and its verification evidence.

The report should explain:

* What changed
* Why it changed
* What files/components were affected
* What was reviewed
* What was tested
* What issues were found and resolved
* What remains outstanding
* Whether the change is ready to merge

The report is **not a second code review**. It summarizes the review and verification evidence into a durable engineering record.

The report must be factual. Never claim that tests, builds, manual verification, security checks, or performance checks were performed unless there is evidence that they were actually performed.

## When to Use

* After completing a feature or bug fix
* After a code review
* Before opening or updating a PR
* When converting review findings into a PR description
* When documenting verification performed on an AI-generated change
* When a reviewer needs a concise summary of implementation and quality status

## Report Principles

### 1. Evidence Over Assumptions

Only report what is known.

Good:

> Unit tests were run with `pytest tests/test_parser.py` and passed.

Bad:

> All tests pass.

unless the complete test suite was actually executed.

Never infer:

* Tests passed because the code looks correct
* Security was verified because no obvious vulnerability was found
* Performance is acceptable without evidence
* Manual testing occurred without a recorded result
* Build succeeded without actually running it

When verification is unknown, explicitly state:

> Not verified.

### 2. Summarize, Don't Repeat the Diff

The PR report should explain intent and impact rather than describe every changed line.

Prefer:

> Extracted validation into a dedicated helper so request parsing and business logic remain separate.

Avoid:

> Added a function called `validate_request()` on line 42.

### 3. Lead With Impact

The reader should understand the change quickly.

The report should answer:

1. What problem does this solve?
2. What changed?
3. What is the risk?
4. How was it verified?
5. Is it ready to merge?

### 4. Preserve Review Severity

Review findings must retain their severity.

Use:

* **Critical:** Blocks merge
* **Required:** Must be addressed before merge
* **Optional:** Improvement, not required
* **Nit:** Minor style or preference
* **FYI:** Informational

Do not downgrade a blocking issue merely because it was inconvenient to fix.

## Report Structure

Generate the following sections when applicable.

```markdown
# PR Report: [Change Title]

## Summary

[2–4 sentence explanation of what changed and why.]

## Changes

- [Meaningful implementation change]
- [Meaningful implementation change]
- [Meaningful implementation change]

## Files / Components

- `path/to/file` — [purpose of change]
- `path/to/file` — [purpose of change]

## Review Findings

### Correctness

- [Finding or "No blocking issues identified."]

### Readability & Simplicity

- [Finding or "No blocking issues identified."]

### Architecture

- [Finding or "No blocking issues identified."]

### Security

- [Finding or "No blocking issues identified."]

### Performance

- [Finding or "No blocking issues identified."]

## Tests

- `[test command]` — [result]
- `[test command]` — [result]

## Verification

- Build: [Passed / Failed / Not run]
- Tests: [Passed / Failed / Partial / Not run]
- Manual verification: [result]
- Security verification: [result]
- Performance verification: [result]

## Outstanding Items

- [Unresolved issue]
- [Follow-up task]

If none:

> None.

## Risk Assessment

**Risk:** [Low / Medium / High]

[Short explanation of why.]

## Verdict

**[Approve / Request Changes / Needs Verification]**

[One concise explanation.]

## Change Description

[Standalone version-control description.]

### Why

[Reason for the change.]

### Approach

[Important implementation/design decisions.]

### Verification

[How the change was verified.]
```

## Step 1: Understand the Change

Before generating the report, establish:

* Change title
* Problem being solved
* Expected behavior
* Actual behavior
* Scope of the change
* Files/components affected
* Whether this is a feature, bug fix, refactor, dependency change, or combination

If the context is ambiguous, do not invent the missing intent.

Use:

> Intent: Not provided.

rather than guessing.

## Step 2: Extract Implementation Changes

Summarize changes at the architectural or behavioral level.

Examples:

* Added request validation at the API boundary
* Extracted orchestration from business logic
* Added regression coverage for empty input
* Replaced duplicated parsing logic with the canonical parser
* Added pagination to prevent unbounded data retrieval

Avoid implementation trivia unless it materially affects reviewers.

## Step 3: Incorporate Review Findings

Map review findings into the appropriate axis.

Example:

```markdown
## Review Findings

### Correctness

- **Required:** Added handling for empty API responses to prevent an index error.

### Architecture

- **Required:** Moved feature-specific validation out of the shared utility module.

### Security

- No blocking security issues identified.

### Performance

- No new unbounded operations identified.
```

Resolved findings should remain visible when useful because they explain why the final implementation looks the way it does.

Do not report a finding as unresolved if the author demonstrably fixed it.

## Step 4: Verify the Verification Story

Build the verification section from actual evidence.

Track each verification activity independently:

| Verification       | Status                    |
| ------------------ | ------------------------- |
| Unit tests         | Passed / Failed / Not run |
| Integration tests  | Passed / Failed / Not run |
| Full test suite    | Passed / Failed / Not run |
| Build              | Passed / Failed / Not run |
| Lint               | Passed / Failed / Not run |
| Type checking      | Passed / Failed / Not run |
| Manual testing     | Passed / Failed / Not run |
| Security checks    | Passed / Failed / Not run |
| Performance checks | Passed / Failed / Not run |

Do not collapse partial verification into a generic "Tests passed."

For example:

> Unit tests passed; integration tests were not run.

is preferable to:

> Tests passed.

## Step 5: Assess Risk

Risk should be based on the change surface, not the author's confidence.

### Low

Typically:

* Small isolated change
* Existing behavior preserved
* Strong regression coverage
* No external interfaces changed
* No security-sensitive behavior

### Medium

Typically:

* Multiple components affected
* New integration behavior
* Data flow changes
* Moderate refactoring
* Partial verification

### High

Typically:

* Authentication/authorization changes
* Data migrations
* Security-sensitive functionality
* Public API changes
* Large architectural changes
* Weak or incomplete verification
* Production-critical behavior

If evidence is insufficient:

> Risk: Unknown — verification evidence is incomplete.

Do not automatically classify an unverified change as low risk.

## Step 6: Determine the Verdict

Use exactly one of:

### Approve

Use only when:

* No Critical findings remain
* No Required findings remain
* Verification is sufficient for the change
* The implementation does not introduce a known structural regression

### Request Changes

Use when:

* Critical issues remain
* Required issues remain
* The change actively worsens architecture
* Security issues remain
* Tests expose broken behavior

### Needs Verification

Use when:

* The implementation appears acceptable
* But required verification has not been performed
* The available evidence is insufficient to confidently approve

Do not use "Approve" merely because the code looks good.

## Change Description Generation

Every PR should have a standalone description.

### First Line

Use a short imperative sentence.

Good:

> Add validation for malformed customer records

> Extract request orchestration from the API handler

> Fix pagination for large customer queries

Avoid:

> Fix bug

> Update code

> Phase 1

> Changes

### Body

Explain:

1. What changed
2. Why it changed
3. Important implementation decisions
4. Verification performed

Example:

```markdown
Add validation for malformed customer records

Validate incoming records before they enter the transformation pipeline. This prevents malformed records from reaching downstream processing and makes validation failures explicit at the system boundary.

The validation logic is isolated from transformation logic so the pipeline remains focused on processing valid records.

Verification:
- Unit tests for valid and malformed records
- Regression test for empty input
- Full test suite
```

## Bug Fix Reports

Bug fixes must explicitly connect the fix to the regression.

Use:

```markdown
## Bug

[What was broken.]

## Root Cause

[Why it happened.]

## Fix

[What changed.]

## Regression Prevention

[Which test prevents recurrence.]

## Verification

[Tests/build/manual verification.]
```

A bug fix without regression coverage should normally be reported as an outstanding verification gap.

## Refactoring Reports

Separate structural improvements from behavior changes.

If the change contains both:

```markdown
## Behavioral Changes

[New or changed behavior.]

## Structural Changes

[Refactoring performed.]

## Risk

[Explain whether behavior should remain unchanged.]
```

If the refactor unnecessarily increases complexity, flag it rather than presenting it as an improvement.

## Dependency Change Reports

For dependency changes, include:

* Package changed
* Previous version
* New version
* Reason
* Relevant breaking/behavioral changes
* Test results
* Lockfile changes
* Security implications

Example:

```markdown
## Dependency Change

- Package: `example-package`
- Previous: `1.4.2`
- New: `1.5.0`
- Reason: [reason]

## Verification

- Changelog reviewed
- Tests passed
- Lockfile diff reviewed
```

Never claim changelog or lockfile review unless it actually happened.

## Dead Code

If the review identifies dead code, include it explicitly.

```markdown
## Dead Code Identified

- `formatLegacyDate()` — no remaining references
- `OldTaskCard` — replaced by `TaskCard`

Status: Removal required / Removal deferred with justification.
```

Do not silently claim that dead code was removed unless the change actually removed it.

## AI-Generated Changes

For AI-generated changes, apply the same reporting standard.

Do not say:

> AI-generated code was reviewed and appears correct.

Instead report concrete evidence:

> The implementation was reviewed across correctness, readability, architecture, security, and performance. Regression tests were executed with `[command]` and passed.

AI authorship is not evidence of correctness.

## Large Changes

If the change exceeds approximately:

* **100 changed lines:** review carefully
* **300 changed lines:** verify that the scope is still coherent
* **1000 changed lines:** recommend splitting unless the change is an appropriate automated refactor or deletion

If splitting is recommended:

```markdown
## Scope Concern

The change is too large to review safely as one logical unit.

Recommended split:

1. Introduce shared interfaces/helpers
2. Migrate existing consumers
3. Add new behavior
4. Remove obsolete implementation
```

Do not approve a large change simply because the implementation is functional.

## Report Quality Gate

Before producing the final report, verify:

* [ ] Intent is understood
* [ ] Summary matches the actual implementation
* [ ] No unsupported claims are made
* [ ] Review findings retain their severity
* [ ] Resolved issues are distinguished from outstanding issues
* [ ] Tests are reported with actual commands/results where available
* [ ] Build status is explicit
* [ ] Manual verification status is explicit
* [ ] Security verification is explicit
* [ ] Performance verification is explicit
* [ ] Risk reflects the actual change surface
* [ ] Outstanding issues are listed
* [ ] Verdict follows the evidence
* [ ] Change description is standalone
* [ ] No invented metrics, test results, screenshots, or approvals

## Output Rules

Keep the final PR report concise enough for a reviewer to scan quickly.

Prefer:

> Added boundary validation and regression coverage for malformed records.

over:

> Several modifications were made to improve the way malformed records are handled throughout the application.

Prefer concrete evidence:

> `pytest tests/test_parser.py` — 18 passed

over:

> Tests were successful.

When no evidence exists, say:

> Not run.

Never manufacture verification evidence.

## Final Verdict

Every generated report must end with one clear status:

```markdown
## Verdict

**Approve**

The change satisfies the review criteria and the available verification evidence is sufficient for merge.
```

or:

```markdown
## Verdict

**Request Changes**

Required issues remain before this change is ready to merge.
```

or:

```markdown
## Verdict

**Needs Verification**

No blocking implementation issue was identified, but the available verification evidence is insufficient for approval.
```

The goal of the report is not to make the PR look good. The goal is to leave an accurate, useful engineering record that allows another engineer to understand the change and make an informed merge decision.
