# Code Review Standard

This standard defines independent code review in Engineering OS, whether
performed by Claude Code, Codex, a human, or another authorized reviewer.
It is used in Step 9 of [workflows/feature.md](../workflows/feature.md).

[AGENTS.md](../AGENTS.md) is authoritative. Task statuses and transitions
are owned by [standards/task-specification.md](task-specification.md);
architecture authority is owned by
[standards/architecture-decisions.md](architecture-decisions.md). This
standard does not redefine either. If it conflicts with any of them, stop
and report the conflict to the Human Tech Lead. Do not resolve it
yourself.

---

# Core Principle

Review is an active attempt to find defects and risks. Summarizing the
diff or confirming that code "looks reasonable" is not a review.

Review against:

- Task requirements and acceptance criteria
- Approved architecture (APPROVED ADRs)
- Security requirements
- Repository standards
- Existing behavior
- Actual validation evidence

---

# Reviewer Independence

- The Assigned Agent must not be the sole reviewer of its own work.
- Claude Code may review Codex's implementation; Codex may review
  Claude Code's implementation. Humans and other reviewers named in the
  task may also review.
- Copilot review assistance may supplement a review, but is never the
  sole required independent review.
- A reviewer who made substantive changes to the implementation is no
  longer independent for those changes. See
  [Reviewer Changes](#reviewer-changes).
- The Human Tech Lead retains final approval authority. A review result
  is a recommendation.

---

# Review Inputs

Before a formal review, read as applicable:

- AGENTS.md and the reviewer's agent definition
- The task (`TASK-NNN`)
- Referenced ADRs
- Relevant standards and workflows
- The changed code, and the full diff against the base branch
- Related existing implementation
- Tests
- Validation evidence reported by the Assigned Agent
- Migration and configuration changes

If critical context is missing (for example, the task file, a referenced
ADR, or validation evidence), report it as a finding or a question.
Do not invent the missing context.

---

# Risk-Based Depth

Review depth scales with risk.

Review more deeply when the change touches:

- Authentication or authorization
- Payments
- Personal or sensitive data
- Destructive migrations
- Tenant isolation
- Concurrency
- External integrations
- Infrastructure or security configuration

A small documentation or low-risk change does not require pretending that
every category below applies. State which categories were reviewed and
which were not applicable.

---

# Review Scope

## Requirements and Correctness

- The implementation matches the task requirements.
- Acceptance criteria are satisfied.
- Business behavior is correct.
- Edge cases are considered.
- No unrelated scope was introduced.

## Architecture

- The implementation follows the applicable APPROVED ADRs.
- Service and module boundaries are respected.
- No architecture decision is silently introduced. A new significant
  decision needs an ADR, not just code.
- Superseded or rejected architecture is not used.

## Security

Where applicable:

- Authentication
- Authorization
- Tenant isolation
- Input validation
- Secrets and credentials
- Sensitive data exposure
- Injection risks
- Insecure defaults
- Dependency risks
- Logging of sensitive data
- Privilege escalation and trust boundaries

Never accept a security bypass because tests or builds pass.

## Error Handling and Reliability

- Errors are handled explicitly.
- Failures are not silently ignored.
- Retries and timeouts are reasonable, where applicable.
- Partial failure behavior is considered.
- Resources are cleaned up correctly.

## Data and Database

Where applicable:

- Schema correctness and constraints
- Migrations, including destructive changes and rollback risk
- Backward compatibility of stored data
- Transactions
- Indexes and query behavior
- Data integrity

## Concurrency

Where applicable:

- Race conditions and shared state
- Locking
- Duplicate processing and idempotency
- Transaction boundaries
- Async and background behavior

Do not raise concurrency findings when the code has no concurrency
concerns.

## API and Compatibility

Where applicable:

- API contract correctness
- Request and response compatibility
- Breaking changes and versioning implications
- Validation behavior
- Status codes and error responses

## Performance

Look for meaningful risks, such as:

- N+1 queries
- Unbounded loops or result sets
- Excessive memory use
- Unnecessary network calls
- Blocking operations
- Obvious scaling problems

Do not demand speculative optimization without evidence.

## Tests and Validation

- Relevant tests exist.
- Important failure paths are tested.
- Edge cases are tested where practical.
- Authorization and security behavior is tested where applicable.
- Reported validation corresponds to actual commands and results.

Reviewing tests is not executing them. Do not report tests as passing
unless evidence shows they were executed. If the reviewer runs checks,
report those commands and results separately from the Assigned Agent's
evidence.

## Maintainability

- Readability
- Unnecessary complexity
- Duplication
- Naming
- Dead code
- Hidden coupling

Do not block on personal style preferences when repository standards
are satisfied.

---

# Severity Levels

Use the AGENTS.md severity levels. BLOCKER and HIGH must be resolved
before merge (AGENTS.md); MEDIUM may be deferred with justification
(agent definitions).

| Severity   | Meaning                                                                                              | Effect on approval                          |
| ---------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| BLOCKER    | Cannot safely merge: severe security exposure, data corruption or loss, fundamental requirement failure, or a change that makes the system unusable. | Must be resolved                  |
| HIGH       | Serious correctness, security, architecture, compatibility, or reliability problem.                  | Must be resolved                            |
| MEDIUM     | Real defect or risk that should normally be corrected.                                               | Fix, or defer with a documented reason      |
| LOW        | Limited-impact issue.                                                                                | Does not normally block                     |
| SUGGESTION | Optional improvement, alternative, or polish.                                                        | Does not block                              |

Rate severity by impact. Do not inflate severity to enforce personal
preferences, and do not lower it to make a review pass.

A BLOCKER or HIGH finding is resolved only when it is fixed and
re-reviewed, or when the Reviewer or the Tech Lead accepts a documented
explanation that it does not apply. The Assigned Agent cannot close it
alone. Disagreements on severity go to the Tech Lead.

---

# Finding Format

Each finding contains:

- **Severity**
- **Title** — short
- **Location** — file and line, or the affected component when the
  issue is architectural or cross-cutting
- **Problem**
- **Why it matters**
- **Recommended correction**
- **Evidence level** — see [Evidence and Uncertainty](#evidence-and-uncertainty)

When useful, also include a reproduction scenario, supporting evidence,
and the affected acceptance criterion or ADR.

Do not invent line numbers for cross-cutting issues.

When requesting changes, also classify each finding as an **ordinary
correction** or **readiness-invalidating**, as required by
[Review Findings](task-specification.md#review-findings).

---

# Evidence and Uncertainty

Label each finding with one evidence level:

| Level      | Meaning                                                        |
| ---------- | -------------------------------------------------------------- |
| Verified   | Confirmed by reading the code, running it, or a failing test.  |
| Likely     | Strong indication, not confirmed.                              |
| Question   | Missing context or unclear intent; needs an answer.            |
| Suggestion | Optional improvement; not a defect.                            |

Do not state speculation as confirmed fact. If a conclusion depends on
tests or tools that were not run, say so in **Not Verified**.

---

# Review Outcome

A formal review returns **PASS** or **REQUEST CHANGES**.

## PASS

PASS means all of the following:

- No unresolved BLOCKER or HIGH findings.
- The review covered the scope required by the change's risk.
- The required evidence (task, ADRs, validation results) was available.
- Any deferred findings are disclosed with their reasons.

PASS does **not** mean:

- Human Tech Lead approval
- Merge authorization
- Production or release approval
- Task DONE

The Reviewer moves the task to READY FOR HUMAN APPROVAL only when the
full [approval gate](task-specification.md#ready-for-human-approval) is
also met, including CI. Otherwise, report what remains.

## REQUEST CHANGES

REQUEST CHANGES means one or more findings require correction,
clarification, governance recovery, or renewed validation. The task moves
to CHANGES REQUESTED, and then follows the task standard.

---

# Reviewer Changes

A formal reviewer reviews; it does not silently fix the implementation.

- **Non-substantive changes** (typos in comments or documentation,
  formatting, broken links) may be made by the reviewer when the review
  assignment allows file changes. Each change is listed in the review
  report.
- **Substantive changes** (anything that affects behavior, tests,
  configuration, security, data, or acceptance criteria) are reported as
  findings and returned to the Assigned Agent.
- If the reviewer makes a substantive change anyway, the reviewer is a
  contributor to that change. Another independent reviewer must review
  it before PASS.
- If the review assignment says not to modify files, make no changes.

---

# Re-review

After REQUEST CHANGES:

1. The Assigned Agent addresses the findings.
2. Applicable validation is repeated and the actual results reported.
3. The Reviewer re-reviews the affected areas.
4. The Reviewer checks that the fixes did not introduce regressions.
5. Unresolved findings remain listed until resolved or explicitly
   deferred.

Fixing one finding does not resolve the review. Each round ends with a
new PASS or REQUEST CHANGES.

---

# Governance-Impacting Findings

If review finds invalid requirements, missing architecture approval, an
unresolved security-sensitive decision, use of a superseded ADR, or any
other readiness-invalidating issue, it is not an ordinary code
correction. Classify the finding as readiness-invalidating and follow the
task standard:

IN REVIEW → CHANGES REQUESTED → BLOCKED → resolve the governance issue →
readiness verified again → READY FOR IMPLEMENTATION → IN PROGRESS →
validation → IN REVIEW

Transition ownership is defined in
[standards/task-specification.md](task-specification.md#transitions).
The Reviewer does not resolve the governance issue itself.

---

# Review Report

Use this shape for every formal review.

```markdown
## Review: TASK-NNN — Round N

**Result:** PASS | REQUEST CHANGES
**Reviewer:** <name>
**Implementation by:** <Assigned Agent>
**Scope reviewed:** <branch, commits, or PR>

### Findings
- [HIGH] <Title> — <location>
  - Problem: <...>
  - Why it matters: <...>
  - Correction: <...>
  - Evidence: Verified | Likely | Question
  - Classification: ordinary correction | readiness-invalidating
- [MEDIUM] <...>

### Categories Reviewed
- <Reviewed categories; categories not applicable, with reason>

### Validation Reviewed
- <Assigned Agent's reported commands and results>

### Validation Executed by Reviewer
- `<command>` — <result> (or "None")

### Not Verified
- <What was not checked or run, and why>

### Reviewer Changes
- <Non-substantive changes made, or "None">

### Residual Risks / Deferred Findings
- <Finding> — <reason for deferral>
```
