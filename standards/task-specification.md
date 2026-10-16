# Task Specification Standard

This standard defines how Engineering OS tasks are written, tracked, and
completed. Use [templates/TASK.md](../templates/TASK.md) to create a task.

A task specification is the implementation handoff contract between:

Human Tech Lead → ChatGPT → Implementation Agent → Reviewer → Human Tech Lead

It must allow an implementation agent to execute the work using only the
repository, without depending on any chat history.

[AGENTS.md](../AGENTS.md) is authoritative. If this standard conflicts
with AGENTS.md or an agent definition in [agents/](../agents/), stop and
report the conflict to the Human Tech Lead. Do not resolve it yourself.
This standard grants no authority beyond what those documents allow.

---

# Task Identity

Every task contains:

| Field          | Rule                                                         |
| -------------- | ------------------------------------------------------------ |
| Task ID        | `TASK-NNN`, for example `TASK-006`, `TASK-021`, `TASK-105`   |
| Title          | Short and specific                                           |
| Status         | One of the [statuses](#status-lifecycle) below               |
| Assigned Agent | The single owner of the implementation                       |
| Reviewer       | Who performs the independent review                          |
| Priority       | As set by the Tech Lead                                      |

Task IDs:

- Use the next unused number, zero-padded to at least three digits.
  After `TASK-999`, continue with `TASK-1000`.
- Never reuse an ID, including for cancelled tasks.
- Before assigning an ID, check existing task files and branch names.
  If two tasks receive the same ID, report it to the Tech Lead.

The task ID appears in the branch name, commit context, and pull request,
as defined in AGENTS.md.

---

# Status Lifecycle

## Statuses

| Status                   | Meaning                                                                        |
| ------------------------ | ------------------------------------------------------------------------------ |
| DRAFT                    | Being written. Not actionable.                                                 |
| BLOCKED                  | Cannot proceed. The blocker is recorded in the task's Status History.          |
| READY FOR IMPLEMENTATION | Passed [readiness verification](#readiness-verification). May be started.      |
| IN PROGRESS              | The Assigned Agent is implementing on a task branch.                           |
| IN REVIEW                | Implementation and actual validation reported; independent review under way.   |
| CHANGES REQUESTED        | The Reviewer or Tech Lead requires corrections.                                |
| READY FOR HUMAN APPROVAL | Passed the [approval gate](#ready-for-human-approval). Awaiting the Tech Lead. |
| DONE                     | Meets the AGENTS.md Definition of Done and has human approval.                 |
| CANCELLED                | Stopped by the Tech Lead. Kept for history.                                    |

## Transitions

| From                     | To                       | Who                          | Condition                                                                   |
| ------------------------ | ------------------------ | ---------------------------- | --------------------------------------------------------------------------- |
| (new)                    | DRAFT                    | ChatGPT or Tech Lead         | Task created                                                                |
| DRAFT                    | BLOCKED                  | ChatGPT or Tech Lead         | A readiness condition cannot yet be met                                     |
| DRAFT or BLOCKED         | READY FOR IMPLEMENTATION | ChatGPT or Tech Lead         | [Readiness verification](#readiness-verification) passed, with evidence     |
| READY FOR IMPLEMENTATION | IN PROGRESS              | Assigned Agent               | Work starts on a task branch                                                |
| IN PROGRESS              | IN REVIEW                | Assigned Agent               | Implementation complete; actual validation reported                         |
| IN PROGRESS              | BLOCKED                  | Assigned Agent               | A [readiness-invalidating issue](#readiness-invalidating-issues) is found   |
| IN REVIEW                | CHANGES REQUESTED        | Reviewer                     | Review findings require changes                                             |
| IN REVIEW                | READY FOR HUMAN APPROVAL | Reviewer                     | Every [approval gate](#ready-for-human-approval) item is met                |
| CHANGES REQUESTED        | IN PROGRESS              | Assigned Agent               | Every finding is an [ordinary correction](#ordinary-corrections)            |
| CHANGES REQUESTED        | BLOCKED                  | Reviewer or Assigned Agent   | Any finding is readiness-invalidating, or its classification is disputed    |
| READY FOR HUMAN APPROVAL | CHANGES REQUESTED        | Tech Lead                    | Tech Lead requires corrections                                              |
| READY FOR HUMAN APPROVAL | DONE                     | Tech Lead only               | Definition of Done met and approval given                                   |
| Any except DONE          | CANCELLED                | Tech Lead only               | Tech Lead decision                                                          |

Not permitted:

- **BLOCKED → IN PROGRESS.** A blocked task always passes readiness
  verification and returns to READY FOR IMPLEMENTATION first.
- The Assigned Agent moving a task to CHANGES REQUESTED, READY FOR
  HUMAN APPROVAL, READY FOR IMPLEMENTATION, or DONE. The Assigned Agent
  does not request changes from itself, act as its own Reviewer, or
  resolve its own governance blockers.
- Any agent moving a task to DONE. Generating code, passing tests, or
  opening a pull request is not DONE.

Rules:

- Moving a task *into* BLOCKED is always allowed for whoever finds the
  blocker. Moving it *out* of BLOCKED requires readiness verification.
- The Tech Lead may move a task to any status and records the reason.
  A task leaving BLOCKED for implementation still passes through
  READY FOR IMPLEMENTATION.
- Every move to or from BLOCKED, CHANGES REQUESTED, READY FOR
  IMPLEMENTATION, or DONE is recorded in the task's Status History with
  who made it, why, and a repository reference to any evidence.
- Status is recorded in the task file in the repository. If an agent
  cannot update the file (for example, it is not permitted to commit),
  it states the intended transition in its handoff and the human
  records it.

## Review Findings

During IN REVIEW, the Reviewer owns the findings and decides whether
they require CHANGES REQUESTED. The Assigned Agent may identify and
report additional problems, but does not act as the Reviewer or make
review transitions.

When requesting changes, the Reviewer (or the Tech Lead, when the
Tech Lead requests changes) classifies each finding as an ordinary
correction or readiness-invalidating. If the Assigned Agent
disagrees with a classification, it reports the disagreement and the
task moves to BLOCKED for the Tech Lead to decide.

### Ordinary Corrections

A finding is an ordinary correction when it can be fixed within the
approved task specification and does not change requirements, scope,
acceptance criteria, architecture approval, security-sensitive
approval, dependencies, or any other readiness condition.

Path:

CHANGES REQUESTED → IN PROGRESS → IN REVIEW

The Assigned Agent fixes the findings, re-runs validation, reports the
actual results, and returns the task to IN REVIEW. Independent review is
required again.

### Readiness-Invalidating Issues

An issue is readiness-invalidating when it invalidates any of:

- Confirmed requirements
- Approved scope
- Acceptance criteria
- A required architecture decision
- A security-sensitive decision
- A required dependency
- Any other [READY FOR IMPLEMENTATION](#ready-for-implementation)
  condition

Path:

CHANGES REQUESTED → BLOCKED, or IN PROGRESS → BLOCKED

The task must not return directly to IN PROGRESS. The issue is recorded
in Status History, and the task follows the
[BLOCKED recovery](#blocked-recovery) path.

## BLOCKED Recovery

Before implementation resumes:

1. Requirements and the task specification are updated in the
   repository where necessary.
2. Architecture decisions are updated or newly proposed where
   necessary, and approved by the Tech Lead.
3. Dependencies are resolved.
4. Security-sensitive decisions receive required Tech Lead approval.
5. Readiness verification is performed again.

Path:

BLOCKED → READY FOR IMPLEMENTATION → IN PROGRESS

## Readiness Verification

| Who                     | Does                                                                                  |
| ----------------------- | ------------------------------------------------------------------------------------- |
| ChatGPT or Tech Lead    | Prepares and updates the task; assesses every readiness condition                     |
| Tech Lead only          | Gives approvals reserved to the Tech Lead: scope, architecture, security-sensitive    |
| ChatGPT or Tech Lead    | Records READY FOR IMPLEMENTATION once every condition is met with evidence            |
| Assigned Agent, Reviewer | Report blockers; never record approvals or READY FOR IMPLEMENTATION                  |

Rules:

- ChatGPT must not approve its own architecture proposals or any other
  decision reserved for the Tech Lead, and must not record an approval
  the Tech Lead has not given.
- Each required approval is recorded in the task with a repository
  reference (decision record, approving pull request, or recorded
  Tech Lead instruction). Approval that exists only in chat does not
  count.
- Significant changes to the task or specification during
  implementation require readiness verification again.

Agent handoff formats use report statuses such as READY FOR REVIEW.
A handoff marked READY FOR REVIEW corresponds to moving the task to
IN REVIEW.

---

# Required Content

Every implementation task contains these sections:

1. Task ID
2. Title
3. Status
4. Goal
5. Context
6. Requirements
7. Acceptance Criteria
8. Dependencies
9. Constraints, including out of scope
10. Required Reading / Repository References
11. Architecture Decisions
12. Security Considerations
13. Testing Expectations
14. Validation Commands, when known
15. Definition of Done
16. Assigned Agent
17. Reviewer
18. Open Questions
19. Assumptions

A section that does not apply contains `N/A` and a one-line reason.
Required sections must not be removed or left silently empty.

---

# Requirements

- Describe required behavior precisely enough that an implementation
  agent can work without hidden conversation context.
- Do not invent missing business behavior. Missing behavior is an open
  question, not a requirement.
- Keep three things separate:
  - **Confirmed requirements** — agreed by the Tech Lead or recorded in
    repository documentation.
  - **Assumptions** — labeled, each awaiting confirmation.
  - **Open questions** — each marked critical or non-critical, with who
    must answer it.
- A critical open question blocks implementation readiness.

---

# Acceptance Criteria

Each criterion must be:

- Observable
- Testable where practical
- Implementation-independent where possible

Cover failure behavior, edge cases, and authorization where applicable.
Use Given / When / Then where it improves clarity; it is not required
for every criterion.

---

# Repository References

Important project state must be referenced from repository files.
Reference the relevant:

- Architecture decisions
- Standards
- Workflows
- API contracts
- Database designs
- Existing implementation

Use repository paths. Chat transcripts are not authoritative references.
If a needed decision or clarification exists only in chat, record it in
the repository first or list it as an open question.

---

# Architecture Readiness

Every architecture decision the task requires must:

- Exist in repository documentation
- Have status Approved, with Human Tech Lead approval evidence as
  defined in [agents/chatgpt.md](../agents/chatgpt.md#decision-status)
- Be referenced by path in the task's Architecture Decisions section

If any required decision is Proposed, unresolved, Rejected, Superseded
without an Approved replacement, or missing, the task is **BLOCKED**.

ChatGPT cannot approve its own architecture proposal. If the task needs
no architecture decision, state `N/A` with a reason.

---

# Security Considerations

State whether the task touches authentication, authorization, secrets,
external input, sensitive data, dependencies, or other security controls.

- Changes to authentication or authorization require extra review.
- Security-sensitive decisions require Human Tech Lead approval before
  the task is READY FOR IMPLEMENTATION.
- If none apply, state `N/A` with a reason.

---

# Assignment

| Role           | Rule                                                                 |
| -------------- | -------------------------------------------------------------------- |
| Assigned Agent | Exactly one. Assigned by the Tech Lead. Owns the implementation.     |
| Reviewer       | Did not implement the change. Performs the independent review.       |

- The Assigned Agent must not be the sole reviewer of its own work.
- Copilot interactive assistance does not make Copilot the Assigned
  Agent. The engineer or agent using Copilot remains the Assigned Agent.
- If Copilot implements the task in an autonomous or agentic mode,
  record Copilot as the Assigned Agent.
- A Copilot PR review must not be the only review. If Copilot is listed
  as a reviewer, list another independent reviewer as well.
- AI review does not replace human review. Final approval remains with
  the Tech Lead.

---

# Scope

- Keep tasks small and independently verifiable, normally one pull
  request each.
- Do not combine unrelated changes in one task.
- If a task becomes too large, split it into smaller tasks and record
  the dependencies between them.

---

# Validation

- State expected validation before implementation, when reasonably
  known: what must be tested and the commands to run.
- Agents report the commands actually executed and the actual results,
  including failures and anything not verified.
- An expected validation command in the task is not evidence. Never
  report it as passed unless it was executed.

---

# Changes During Implementation

If implementation discovers any of the following, the task must not
silently drift:

- Requirement ambiguity
- Architecture conflict
- Security-sensitive uncertainty
- Scope expansion
- Missing dependency

The Assigned Agent:

1. Stops work in the affected area.
2. Reports the newly discovered ambiguity, conflict, or risk to the
   Tech Lead and records it in Status History.
3. Moves (or reports) the task to **BLOCKED** when continuing would
   require guessing behavior or violating governance.
4. Resumes only through [BLOCKED recovery](#blocked-recovery), after
   readiness verification by ChatGPT or the Tech Lead.

The Assigned Agent does not request changes from itself and does not
resolve the blocker by changing the specification, approvals, or scope
on its own. During review, the Reviewer owns findings and
CHANGES REQUESTED; see [Review Findings](#review-findings).

Important decisions made while resolving the issue must be recorded in
repository documentation, not only in chat.

---

# Completion Gates

## READY FOR IMPLEMENTATION

Checked during [readiness verification](#readiness-verification),
including every time a task leaves BLOCKED. All of the following are
true:

- [ ] All required sections are present, with `N/A` and a reason where
      not applicable.
- [ ] Requirements are confirmed; none depend on chat-only context.
- [ ] No critical open questions or critical ambiguity remain.
- [ ] Every required architecture decision is Approved and referenced.
- [ ] Required dependencies are available or completed.
- [ ] Security-sensitive decisions have Tech Lead approval.
- [ ] Assigned Agent and Reviewer are named, and they are different.
- [ ] The Tech Lead has approved the scope, with a recorded reference.
- [ ] Every required approval has a repository reference in the task.
- [ ] The task is recorded in the repository.

If any item is not met, the task is **BLOCKED** (or still DRAFT while
being written).

## READY FOR HUMAN APPROVAL

All of the following are true:

- [ ] Acceptance criteria are satisfied.
- [ ] Code builds successfully, where applicable.
- [ ] Required tests were executed and pass, with commands and results
      reported.
- [ ] Independent review is complete.
- [ ] No BLOCKER or HIGH findings remain unresolved.
- [ ] Relevant documentation is updated.
- [ ] CI passes.
- [ ] Remaining MEDIUM findings are fixed or deferred with justification.

At this point the Tech Lead can make the final decision.

## DONE

DONE requires the Definition of Done in [AGENTS.md](../AGENTS.md) and
the required human approval. Only the Tech Lead moves a task to DONE.
