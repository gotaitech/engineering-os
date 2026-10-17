# Bugfix Workflow

This workflow takes a reported failure through investigation, a small safe
correction, regression protection, independent review, human approval, and
completion. A bugfix restores intended behavior; calling work a bug does not
authorize new product behavior, architecture, or production access.

[AGENTS.md](../AGENTS.md) is authoritative. The Human Tech Lead retains final
authority. This workflow connects the existing rules rather than replacing them:

| Topic | Owned by |
| --- | --- |
| Task statuses, transition ownership, readiness, completion | [Task standard](../standards/task-specification.md) |
| Architecture approval and supersession | [ADR standard](../standards/architecture-decisions.md) |
| Security decisions and exceptions | [Security standard](../standards/security.md) |
| Independent review and finding severity | [Review standard](../standards/code-review.md) |
| Branches, commits, PRs, merge, history | [Git standard](../standards/git.md) |
| Roles, permissions, handoffs | [Agent definitions](../agents/) |
| Normal delivery gates | [Feature workflow](feature.md) |

If instructions conflict, stop and report the conflicting sources and impact
to the Tech Lead. Do not independently resolve the conflict.

# 1. Workflow Overview

Bug Report / Failure Signal → Triage → Reproduce / Establish Evidence →
Expected vs Actual Behavior → Impact / Risk Assessment → Root Cause Investigation
→ Define Fix Scope → Requirements / Architecture / Security Check →
Task Preparation / Assignment → Readiness Verification → READY FOR IMPLEMENTATION
→ Branch → Implement Fix → Validation / Regression Testing → Independent Review
→ READY FOR HUMAN APPROVAL → Human Approval → Merge → Post-Merge Verification
where applicable → DONE.

Stages are activities, not new task statuses. Investigation can iterate as
evidence changes. A regression test may precede the fix where practical.

| Stage | Normal task relationship | Responsibility |
| --- | --- | --- |
| Intake, triage, evidence, investigation | Pre-task or DRAFT; existing tasks keep their governed status | Human-led triage; assigned investigator within authorized scope |
| Specification and assignment | DRAFT | ChatGPT prepares; Tech Lead approves scope and assigns implementer/reviewer |
| Readiness verification | DRAFT or BLOCKED → READY FOR IMPLEMENTATION | ChatGPT or Tech Lead verifies; reserved approvals remain the Tech Lead's |
| Branch and implementation | READY FOR IMPLEMENTATION → IN PROGRESS | Assigned Agent |
| Validation and handoff | IN PROGRESS → IN REVIEW | Assigned Agent reports actual evidence |
| Independent review and approval gate | IN REVIEW → READY FOR HUMAN APPROVAL only when all gates pass | Independent Reviewer |
| Human approval, merge, applicable verification | READY FOR HUMAN APPROVAL | Tech Lead controls approval/merge; verification by authorized operator |
| Completion | READY FOR HUMAN APPROVAL → DONE | Tech Lead only, after completion gates |

Early investigation does not authorize implementation before readiness. Work
that changes repository artifacts, adds instrumentation, or needs a substantial
investigation uses an appropriately scoped task and normal branch/review gates.
An investigation task can have evidence-gathering acceptance criteria without
claiming to fix the bug. Production investigation requires separate applicable
authorization, including access to production diagnostics.

# 2. Bug Intake

A report may come from users, support, QA, tests, monitoring/alerts, production
incidents, developer discovery, security review, regressions, or dependency and
integration failures.

Record enough to identify the behavior. Where available capture affected
feature/component, environment, actual and expected behavior, reproduction steps,
frequency, affected users/scope, timestamps, request/correlation identifiers,
screenshots/logs/errors, and relevant versions, commits, or releases.

Missing fields are allowed; label them unknown rather than fabricating evidence.
Keep important evidence and decisions in the repository-backed task or approved
restricted records referenced by it. Redact secrets and sensitive data; do not
copy raw production payloads or exploit details into broadly visible artifacts.

# 3. Triage

The Tech Lead directs triage and prioritization, with engineering input.
Determine enough to choose a safe next action: reproducibility, business/user
impact, security, data integrity, tenant isolation, availability, regression
status, scope, workarounds, affected environments, and urgency. Reassess impact
as evidence improves.

Use project-defined bug priority or incident rules where they exist. This
workflow creates no universal incident severity system. Review finding severity
is a separate classification and does not automatically set incident priority.
Escalate suspected security exposure or data loss promptly to the Tech Lead.

# 4. Reproduction and Evidence

Prefer a reproducible failure before changing code where practical. Evidence may
come from an automated test, local or integration reproduction, logs/traces,
deterministic fixtures, captured request/response, controlled simulation, or
existing production evidence when direct reproduction is unsafe.

Do not require production reproduction or mutate production data to reproduce a
bug without applicable explicit authorization. Prefer synthetic/generated or
properly de-identified fixtures; production-derived data follows the
database [test-data](../standards/database.md#28-test-data) and
[sensitive-data](../standards/database.md#16-sensitive-data) rules.

If exact reproduction is impossible, record the strongest available evidence,
conditions checked, limitations, and uncertainty. Correlation alone does not
prove a root cause. Distinguish observations from hypotheses.

# 5. Expected vs Actual Behavior

Record both what should happen and what currently happens, with evidence and
references. Expected behavior comes from authoritative approved requirements,
acceptance criteria, applicable approved ADRs, API/contracts, recorded product
decisions, authoritative documented behavior, or tests that correctly encode it.

Existing code is not automatically the requirement; an existing test is not
automatically correct. Do not derive intended behavior from personal preference.
Material ambiguity or contradiction stops affected work and follows task-standard
BLOCKED/readiness governance. If no task exists yet, record the question in a
DRAFT task and resolve readiness before implementation.

# 6. Root Cause Investigation

Investigate the cause, not just the visible symptom. Consider applicable business
logic and state transitions, validation, authorization, tenant scoping,
concurrency/races, transactions, caches, contracts, schemas/data/migrations,
external integrations, async delivery/retries/idempotency, configuration,
environment differences, dependencies, time/timezones, and UI state.

Separate trigger, symptom, root cause, and contributing factors where possible.
Record supporting evidence, rejected hypotheses where useful, and remaining
uncertainty. Do not manufacture a complete narrative when evidence is incomplete.
Safe investigation may continue during implementation within the approved scope;
uncertainty about implementation mechanics is not permission to guess product or
security behavior.

# 7. Fix the Cause, Not Only the Symptom

Address the smallest verified cause that safely restores intended behavior.
Do not suppress errors, swallow exceptions, disable validation, bypass
authorization, remove tenant filters, weaken integrity constraints, silently
ignore malformed data, or return success for failed operations to hide a bug.
Do not increase retries without understanding failure semantics or add arbitrary
sleeps to conceal races. Never disable tests or loosen assertions merely to pass.

Containment can be appropriate within approved scope, but label it as containment
or mitigation, document residual risk, and record follow-up ownership/work. A
mitigation does not prove root-cause correction. If only mitigation is deliverable,
obtain normal approval of that scope and acceptance criteria; do not self-declare
the original full fix complete.

# 8. Scope Definition

Define affected components/code paths, tests, data/schema/migration implications,
API/contracts, security, compatibility, operational effects, and documentation
where applicable. Choose the smallest safe scope, not the fewest changed lines
at the cost of correctness.

Avoid unrelated refactoring. Small refactoring needed to implement or test the fix
safely is acceptable when explicitly related. Report materially expanded scope
through the task standard; split follow-up work when appropriate.

# 9. Requirements Check

Distinguish implementation that differs from clear approved behavior from
intended behavior that is missing, ambiguous, contradictory, unsafe, or needs
material change. Only the former normally proceeds as an ordinary bugfix.

For the latter, stop affected implementation and obtain clarification/approval
through requirements, security, architecture, and task governance as applicable.
Update authoritative requirements and acceptance criteria through that process.
A bug label or urgency is not approval to change product semantics.

# 10. Architecture Check

Determine whether the correction follows approved architecture, needs a new
significant decision, contradicts an ADR, or exposes unresolved architecture.
Follow the [ADR standard](../standards/architecture-decisions.md): required ADRs
must be applicable and APPROVED with repository approval evidence before
readiness. No AI agent, including ChatGPT, approves architecture.

Do not hide architecture changes inside a fix. Trivial local implementation
choices need no artificial ADR. If a required ADR is superseded before task
completion, follow its unfinished-task impact assessment and recovery rules;
previous readiness/review does not authorize use of obsolete architecture.

# 11. Security Check

Apply the [Security standard](../standards/security.md), including authentication,
authorization, tenant isolation, sensitive data, secrets, injection, files,
external trust, sessions/tokens, and logging/audit where relevant.

Never repair one defect by weakening another security control. Missing security
decisions or invalid trust assumptions use BLOCKED recovery. Authentication and
authorization changes require extra review and applicable negative tests.
Restrict exploit evidence appropriately; keep a safe traceable repository
reference without unnecessarily publishing sensitive details.

# 12. Data / Database Check

Apply the [Database standard](../standards/database.md) to incorrect data,
constraints, schema/migrations, tenant data, transactions, concurrency, backfills,
and destructive corrections. Fixing code does not authorize changing historical
production data. Determine separately whether remediation is needed (Section 29).

Production remediation requires applicable authorization and recovery planning.
For production-affecting migrations, disclose automatic merge/deployment/startup
execution triggers and obtain required authorization before enabling execution,
or gate it under the approved deployment architecture, as the database standard
requires. A merged fix or passing CI is not production execution authority.

# 13. Frontend / Client Check

Apply the [Frontend standard](../standards/frontend.md) to loading/empty/error/
success states, stale responses, races, tenant switches/caches, optimistic
updates, duplicate submission, navigation, accessibility, persisted state, and
mixed-version clients where applicable. Do not hide backend failure by changing
presentation or move trusted authorization into the UI.

# 14. Backend Check

Apply the [Backend standard](../standards/backend.md) to validation,
authorization, contracts, transactions, concurrency, idempotency, retries,
external calls, async behavior, time, configuration, and resource lifecycle.
Increased retries are not a generic correction. Account for duplicate effects
and unknown outcomes rather than assuming timeout/cancellation proves rollback.

# 15. Task Preparation

ChatGPT or the Tech Lead prepares the repository-backed task using the
[task standard](../standards/task-specification.md) and
[task template](../templates/TASK.md). Keep it DRAFT while preparing it.

Include the standard's required sections and, where applicable, bug summary,
evidence/reproduction, expected vs actual behavior, impact, proven cause or
labeled hypothesis, bounded fix scope, testable acceptance criteria, regression
expectations, security/data/architecture implications, and validation plan.
Separate confirmed requirements, assumptions, and open questions. References
must enable implementation and independent review without hidden chat history.
There is no parallel bug-specific task lifecycle.

# 16. Assignment and Independence

Before readiness verification, the Tech Lead assigns and records exactly one
Assigned Agent and an independent Reviewer. Claude Code and Codex may implement
or review each other's work according to their roles.

The implementing agent must not be the sole reviewer of its own fix. Substantive
reviewer edits require another independent review under the review standard.
Copilot interactive assistance leaves ownership with the human/assigned agent;
autonomous Copilot implementation requires explicit assignment and the same
gates. Copilot review is never the sole required independent review.

# 17. Readiness Verification

ChatGPT or the Tech Lead verifies the complete
[readiness gate](../standards/task-specification.md#ready-for-implementation)
after assignment. Check clear expected behavior, sufficient reproduction or
alternative evidence, bounded scope, testable acceptance criteria, available
dependencies, named implementer/reviewer, recorded scope approval, applicable
APPROVED ADRs, resolved critical questions, required security approvals, and
repository references/evidence for every required approval.

Only after every gate passes may ChatGPT or the Tech Lead record READY FOR
IMPLEMENTATION. Reserved approvals remain the Tech Lead's. Perfect root-cause
certainty is not required when safe investigation remains within approved scope;
unresolved decisions materially affecting intended behavior still block readiness.

Whoever discovers a readiness-invalidating blocker may record/report BLOCKED as
the task standard permits. This grants no authority to decide the missing rule,
approve architecture, resume implementation, approve/merge a PR, or mark DONE.
Recovery uses the task standard: resolve and record the issue, obtain required
approvals, reverify readiness, then BLOCKED → READY FOR IMPLEMENTATION →
IN PROGRESS. Never resume directly from BLOCKED.

# 18. Branching

Follow the [Git standard](../standards/git.md). Start implementation on a focused
task branch, for example `fix/TASK-042-refresh-token`; never implement normal
fixes or commit directly on `main`. The Assigned Agent starts IN PROGRESS when
work begins on the branch after readiness.

Preserve unrelated working-tree/staged changes; never silently include, discard,
or move them. Commit, push, and PR creation require authority under the agent
definitions and explicit Tech Lead or task instructions. Assignment, repository
access, and tool capability do not themselves grant those permissions. Respect
instructions to leave changes uncommitted and unpushed; do not rewrite history
without the Git standard's required authorization.

# 19. Implementation

The Assigned Agent implements the smallest safe correction against approved
requirements, architecture, security, and applicable standards. Preserve
unrelated behavior and required compatibility, avoid unrelated refactors, add or
update tests, and update documentation/contracts where needed.

If assumptions or scope prove invalid, stop affected work, report the issue, and
use the task standard's applicable path. Do not silently broaden scope or alter
specifications/approvals to unblock yourself.

# 20. Regression Test First Where Practical

Where practical, demonstrate a test failing because of the bug before applying
the fix, then execute it again afterward. Record both observed results. This
shows the test represents the original defect and detects the changed behavior.

A pre-fix failing automated test is not mandatory when reproduction is unsafe,
cannot reasonably precede the fix, has disproportionate cost, or is operational/
environmental rather than directly testable. Document alternative evidence and
remaining limitations. Never fabricate a red test or claim a pre-fix run occurred
when it did not. These exceptions do not waive relevant validation.

# 21. Regression Testing

Every meaningful fix considers protection against recurrence. Target the failed
behavior, not just the new implementation path. Cover applicable original
failure, successful behavior, negative paths, boundaries, authorization, tenant
isolation, concurrency, transactions, errors, retries/idempotency, migrations/
data, and frontend async races. Test depth scales with risk.

Use real integration behavior where mocks would hide correctness, including
storage constraints/transactions or cross-layer/platform behavior. Tests must
not be concealed, bypassed, weakened, skipped, disabled, or deleted merely to
pass. A demonstrably invalid/outdated test may be corrected under agent rules:
cite the reason, preserve required coverage, and expose the correction to review.
If expected behavior is disputed, stop and seek clarification.

# 22. Validation

Run the checks appropriate to the task: targeted and broader regression tests,
build/typecheck, lint/static analysis, integration/database/migration/security
tests, accessibility checks, manual reproduction, and environment-specific
verification where relevant. Use the approved task's expected validation plan.

Record actual commands, environment/versions where relevant, and results.
Distinguish passed, failed, not run (with reason), and not applicable (with reason).
Expected checks and generated tests are not execution evidence. Never claim
validation passed unless it ran; report failures and gaps without hiding them.

# 23. Fix Verification

Verify the original failure no longer occurs under reproduced conditions,
expected behavior occurs, regression tests pass where applicable, relevant
adjacent behavior remains intact, and applicable checks find no known new
security/data/tenant regression. Evidence must support the stated confidence;
changed code alone does not prove the bug is fixed. Where exact reproduction was
impossible, state what alternative verification establishes and what remains
unknown. The Assigned Agent reports actual validation and hands off for IN REVIEW.

# 24. Independent Review

Follow the [Review standard](../standards/code-review.md). Actively seek defects,
including symptom-only patches. Examine expected vs actual behavior, evidence,
root-cause reasoning, scope, correctness, regression risk, architecture, security,
data integrity, concurrency, compatibility, test quality, and validation evidence
in proportion to risk. Reviewing tests is not executing them.

Use its existing BLOCKER / HIGH / MEDIUM / LOW / SUGGESTION severity model.
BLOCKER/HIGH findings must be resolved before merge; MEDIUM findings are fixed or
deferred with documented justification under existing rules. PASS is a review
recommendation, not human approval, merge authority, release permission,
production authority, or DONE.

# 25. Review Findings and Rework

The Reviewer owns review findings and CHANGES REQUESTED during IN REVIEW.
Ordinary defects within approved scope use:

IN REVIEW → CHANGES REQUESTED → IN PROGRESS → correction / affected validation
→ IN REVIEW, with independent re-review.

Readiness-invalidating findings use the task standard's CHANGES REQUESTED →
BLOCKED path, followed by resolution, required specification/ADR/security
approvals, renewed readiness, READY FOR IMPLEMENTATION, and IN PROGRESS before
validation/re-review. Disputed classification follows that standard as well.
Do not route every review comment through BLOCKED or redefine transition owners.

If a new issue arises at READY FOR HUMAN APPROVAL, stop advancement and report to
the Tech Lead. Do not silently modify/push approval-ready content; report changed
reviewed content before pushing it to the open PR. Ordinary corrections follow
the Tech Lead's CHANGES REQUESTED decision; readiness-invalidating blockers may
be recorded/reported by their discoverer where the task standard permits.
Affected validation and independent review repeat before the task returns to
READY FOR HUMAN APPROVAL. Apply the security standard's late-finding safeguards.

# 26. Human Approval

Only the Reviewer records READY FOR HUMAN APPROVAL in the normal review path,
and only when the full
[approval gate](../standards/task-specification.md#ready-for-human-approval)
is met, including actual passing required tests/build, documentation, CI,
independent review, no unresolved BLOCKER/HIGH findings, and required MEDIUM
handling. Review PASS alone does not meet every gate.

The Human Tech Lead considers intended behavior, risk, review, validation,
compatibility, and operational/data effects, then makes the final approval
decision under existing governance. Approval does not itself mark the task DONE.

# 27. Merge

Use a focused PR under the Git standard, with task/evidence references, fix scope,
actual validation, independent review, applicable ADRs, compatibility/migration
notes, and residual risks. An authorized draft PR may be opened during
implementation; PR timing does not change the gates.

Merge requires required validation, independent review, CI, Tech Lead approval,
and branch protections. AI agents must not merge PRs or infer authority from
completion, PASS, access, assignment, or tool capability. Do not bypass required
checks. PR creation is not approval; merging an implementation PR is not ADR
approval or production authorization. Where merge triggers production
deployment, production authorization must be recorded before merge
([release workflow](release.md#8-automated-deployment)). The task remains
READY FOR HUMAN APPROVAL through approval and merge under the feature workflow.

# 28. Post-Merge Verification

Where risk warrants it, plan an owner, target environment, checks, and evidence
for verification after merge/deployment: original failing path, error recovery,
data correction, migration result, integration, or UI/client behavior. This is
not mandatory for every trivial bug. Record what was checked and any failure.

Deployment and production verification require applicable authorization; this
workflow grants no production access or independent deployment authority.
Explicitly defined and authorized automated deployment remains governed by
AGENTS.md. Merge and production release remain separate decisions.

Required verification must complete before DONE. If production verification is
pending an authorized release, report it as pending rather than marking the task
complete. Optional monitoring or separately approved follow-up can be tracked as
new work with an owner. New failures use task governance; do not silently patch
reviewed/merged content or rewrite history.

# 29. Data Remediation

Code correction does not necessarily repair historical records. When remediation
is needed, define affected data, an approved correction rule, tenant scope,
validation, recovery/backup, batching/restartability, and production authorization
where applicable under the database standard. Never invent historical values
that cannot be safely derived.

Track remediation separately when appropriate, with dependencies and ownership.
If required by the fix's acceptance criteria, it must be verified before that task
is DONE. A separate follow-up does not waive an existing completion criterion;
changes to approved scope require normal governance and readiness verification.
Remediation execution is separately authorized even when its code was reviewed.

# 30. Hotfix / Urgent Bugs

Urgency can shorten elapsed time, not silently remove authority or security
requirements. Reduce scope, prioritize evidence, run the minimum safe validation
appropriate to risk, and preserve required independent review and human approval.
Follow project-specific emergency/release governance where it exists.

Any exception follows [feature-workflow exceptions](feature.md#exceptions) and
[security exception governance](../standards/security.md#25-security-exceptions):
explicit Tech Lead authorization, recorded scope/reason/risk and remaining work,
with owner/follow-up as applicable. Authorization does not silently amend
requirements or supersede ADRs; readiness-affecting changes use normal governance.

Never treat hotfix as direct production authority, permission to bypass security,
skip required checks/approval, weaken tests, or invent product behavior. Deferred
non-critical validation or cleanup must be explicitly handled and tracked; do not
report it as executed. Required tests, review, CI, and other completion gates
remain required for DONE. No generic emergency bypass is created here.

# 31. Unable to Reproduce

Record environments checked, evidence examined, attempts made, uncertainty, and
additional evidence needed. Unable to reproduce does not mean the bug does not
exist, is fixed, or should be closed. Do not make speculative changes merely to
see whether it disappears. A safe, scoped observability improvement may gather
evidence through its own applicable implementation/review gates.

# 32. Intermittent Bugs

Consider concurrency, races/timing, retries, networks, external dependencies,
stale caches, distributed state, resource exhaustion, and timezone/clock effects.
Prefer deterministic reproduction or scoped instrumentation where practical.
Arbitrary sleeps or retry increases are not proof of correction. Report
uncertainty and use evidence to bound claims of success.

# 33. Third-Party / Dependency Bugs

Confirm affected versions/behavior where possible. Assess upgrade/downgrade
compatibility, workaround risk, security, vendor status where relevant, fallback
behavior, and tests protecting the integration contract. Do not blindly upgrade
or pin a vulnerable version simply to restore behavior. Report known vulnerable
dependencies and handle risk under the security standard; never suppress scanner
findings to make CI pass.

# 34. Bugfix Documentation

Update documented behavior, operations, configuration, recovery, or support
procedures when affected. Do not rewrite documentation to make incorrect
behavior appear intended. A change to approved expected behavior requires normal
requirements governance. Persist evidence, decisions, validation, residual risk,
and follow-up references so another engineer/agent can continue without chat.

# 35. Completion / DONE

Only the Tech Lead records DONE under AGENTS.md, the task standard, and the
feature workflow. Confirm acceptance criteria and fix verification, regression
protection or recorded alternative evidence, actual passing required tests/build,
independent review, resolved BLOCKER/HIGH findings, required MEDIUM handling,
updated relevant documentation, passing CI, required human approval, and merge.
Complete required post-merge verification/remediation; record ownership of any
legitimately separate follow-up. A patch, PASS, merge, or mitigation alone is not
DONE, and incomplete or unverified required work must remain reported as such.

# 36. Bugfix Checklist

Apply proportionally before implementation and during review/completion. Mark
`N/A` with a reason where genuinely inapplicable. No row universally requires
production reproduction, an ADR, database changes, extra security review beyond
applicable risk, data remediation, post-production verification, or a pre-fix
failing automated test.

| Area | Check where applicable |
| --- | --- |
| Evidence / reproduction | Report identifiable; safe reproduction or strongest alternative evidence; uncertainty recorded |
| Expected / actual | Both explicit; expected behavior traced to authoritative sources |
| Impact / triage | User, availability, security, data, tenant and regression risks assessed |
| Root cause | Evidence distinguished from hypothesis; symptom and contributing factors understood |
| Scope / requirements | Small safe correction; no invented behavior or unrelated refactoring |
| Architecture / security | Applicable approved ADRs and required security decisions; no weakened controls |
| Data / clients / backend | Applicable standards, contracts, tenant scope, races and compatibility considered |
| Task / readiness | Repository task, assigned implementer and independent reviewer; every readiness gate passes |
| Git / PR | Focused branch/PR; unrelated changes preserved; action permissions respected |
| Regression protection | Original failure covered or alternative evidence recorded; no weakened tests |
| Validation / verification | Actual commands/results and gaps; original failure and adjacent behavior checked |
| Independent review / rework | Review complete; findings handled; affected validation and re-review repeated |
| Human approval / merge | Full approval gate, passing CI, Tech Lead approval and controlled merge |
| Post-merge / remediation | Required verification complete; historical data addressed or governed separate work |
| Documentation / DONE | Evidence and follow-ups persisted; AGENTS.md and task-standard gates met; Tech Lead records DONE |
