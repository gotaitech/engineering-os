# Feature Workflow

This workflow takes a normal software feature from business requirement
to DONE. It connects existing Engineering OS rules; it does not define a
separate lifecycle.

| Topic                              | Owned by                                                                  |
| ---------------------------------- | ------------------------------------------------------------------------- |
| Authority, Git, review, Definition of Done | [AGENTS.md](../AGENTS.md) (authoritative)                        |
| Task statuses, transitions, gates  | [standards/task-specification.md](../standards/task-specification.md)     |
| ADR statuses, approval, supersession | [standards/architecture-decisions.md](../standards/architecture-decisions.md) |
| Agent roles and handoff formats    | [agents/](../agents/)                                                     |

If this workflow conflicts with any of those documents, stop and report
the conflict to the Human Tech Lead. Do not resolve it yourself.

---

# Overview

| #  | Stage                              | Task status                            | Responsible                             |
| -- | ---------------------------------- | -------------------------------------- | --------------------------------------- |
| 1  | Requirement intake                 | —                                      | Tech Lead, ChatGPT                      |
| 2  | Architecture decision, if required | —                                      | ChatGPT proposes; Tech Lead decides     |
| 3  | Task specification                 | DRAFT                                  | ChatGPT prepares                        |
| 4  | Assignment                         | DRAFT                                  | Tech Lead                               |
| 5  | Readiness verification             | DRAFT → READY FOR IMPLEMENTATION       | ChatGPT or Tech Lead; Tech Lead approves |
| 6  | Branch                             | READY FOR IMPLEMENTATION → IN PROGRESS | Assigned Agent                          |
| 7  | Implementation                     | IN PROGRESS                            | Assigned Agent                          |
| 8  | Validation                         | IN PROGRESS → IN REVIEW                | Assigned Agent                          |
| 9  | Independent review                 | IN REVIEW                              | Reviewer                                |
| 10 | Pull request                       | (any time from IN PROGRESS)            | Assigned Agent                          |
| 11 | Ready for human approval           | IN REVIEW → READY FOR HUMAN APPROVAL   | Reviewer                                |
| 12 | Human approval and merge           | READY FOR HUMAN APPROVAL               | Tech Lead                               |
| 13 | Completion                         | READY FOR HUMAN APPROVAL → DONE        | Tech Lead                               |

A task becomes READY FOR IMPLEMENTATION only in Step 5, after the
Assigned Agent and Reviewer are recorded in Step 4.

Pull request timing may vary. A pull request may be opened as a draft
during implementation or at the start of review. Task status gates and
independent review requirements do not change with PR timing.

---

# Step 1 — Requirement Intake

The Human Tech Lead provides the business requirement.

ChatGPT, as PM / BA / Architect
([agents/chatgpt.md](../agents/chatgpt.md)):

- Clarifies the requirement and the expected behavior.
- Identifies ambiguity and labels assumptions.
- Identifies security-sensitive decisions.
- Identifies whether an architecture decision is required.

Do not invent missing business behavior. Critical ambiguity blocks task
preparation until the Tech Lead resolves it. Decisions made here are
recorded in the repository, not left in chat.

---

# Step 2 — Architecture Decision (If Required)

Required only for significant decisions, as listed in
[When an ADR Is Required](../standards/architecture-decisions.md#when-an-adr-is-required).
Trivial implementation choices do not need an ADR.

1. ChatGPT (or the Tech Lead) writes an ADR from
   [templates/ADR.md](../templates/ADR.md). It starts as PROPOSED.
2. The Tech Lead reviews it and APPROVES or REJECTS it, with approval
   evidence recorded in the ADR.
3. Implementation never depends on a PROPOSED or REJECTED ADR.

No AI agent approves an ADR.

---

# Step 3 — Task Specification

ChatGPT prepares `TASK-NNN` from [templates/TASK.md](../templates/TASK.md),
following the task standard, and records it with status **DRAFT**. The
task must contain enough repository context to implement without chat
history.

The task stays DRAFT while it is being written. It does not become
READY FOR IMPLEMENTATION during drafting.

---

# Step 4 — Assignment

Before readiness verification, the Tech Lead assigns one Assigned Agent
and an independent Reviewer, and both are recorded in the task's
Metadata. ChatGPT may recommend an assignment; the Tech Lead decides.
Typical pairings:

| Assigned Agent | Reviewer    |
| -------------- | ----------- |
| Claude Code    | Codex       |
| Codex          | Claude Code |

- The Assigned Agent is never the sole reviewer of its own work.
- Copilot interactive assistance does not make Copilot the Assigned
  Agent.
- If Copilot implements in agentic mode, Copilot is the Assigned Agent.
- A Copilot PR review is never the only review.

---

# Step 5 — Readiness Verification

After assignment, ChatGPT or the Tech Lead verifies the
[readiness gate](../standards/task-specification.md#ready-for-implementation),
following
[Readiness Verification](../standards/task-specification.md#readiness-verification).
Every gate condition must pass, including:

- Critical requirements resolved; acceptance criteria defined
- Dependencies available
- Required ADRs APPROVED and referenced
- Security-sensitive decisions approved by the Tech Lead
- Required repository references present
- Assigned Agent and independent Reviewer recorded, and different
- Tech Lead scope approval recorded
- The task file recorded in the repository through the normal pull
  request workflow

Only when every condition passes:

- DRAFT → READY FOR IMPLEMENTATION, or
- BLOCKED → READY FOR IMPLEMENTATION, when recovering from a blocker

ChatGPT may assess readiness, but cannot approve architecture or any
other decision reserved for the Tech Lead. If any condition fails, the
task is BLOCKED (or stays DRAFT while still being written).

---

# Step 6 — Branch

The Assigned Agent creates a dedicated branch using AGENTS.md
conventions, for example:

```
feature/TASK-021-user-login
```

Never commit directly to `main`. The task moves to IN PROGRESS when work
starts on the branch.

---

# Step 7 — Implementation

The Assigned Agent:

- Reads AGENTS.md, its agent definition, the task, and every ADR,
  standard, and workflow the task references.
- Inspects the existing implementation and tests.
- Changes only what the task requires.
- Handles errors explicitly and follows security rules.
- Updates documentation where behavior changes.

If implementation finds critical ambiguity, an architecture conflict,
security uncertainty, a missing dependency, or any other
readiness-invalidating issue:

stop → report to the Tech Lead → move or report the task as BLOCKED

Do not silently expand scope. See
[Changes During Implementation](../standards/task-specification.md#changes-during-implementation).

---

# Step 8 — Validation

Run what applies to the project:

- Formatting and linting
- Compilation / build
- Unit and integration tests
- Security checks
- Task-specific validation from the task's Validation Commands

Record the actual commands and results, including failures and anything
not verified. Never claim a check passed unless it ran. Failed validation
is reported, not hidden or bypassed.

The Assigned Agent then hands off for review (handoff status READY FOR
REVIEW) and the task moves to IN REVIEW.

---

# Step 9 — Independent Review

The Reviewer, who did not implement the change, reviews against:

- Task requirements and acceptance criteria
- Referenced ADRs
- Security
- Correctness and error handling
- Tests and validation evidence
- Backward compatibility
- Scope

Use the review rules and severity levels in AGENTS.md
(BLOCKER / HIGH / MEDIUM / LOW / SUGGESTION). BLOCKER and HIGH findings
must be resolved before approval.

The Reviewer owns the findings and classifies each one
([Review Findings](../standards/task-specification.md#review-findings)):

**Ordinary correction** — fixable within the approved task:

IN REVIEW → CHANGES REQUESTED → IN PROGRESS → IN REVIEW

**Readiness-invalidating** — affects requirements, scope, acceptance
criteria, architecture, security approval, or dependencies:

IN REVIEW → CHANGES REQUESTED → BLOCKED → resolve, update, approve →
readiness verified again (Step 5) → READY FOR IMPLEMENTATION → IN PROGRESS

Each return to IN REVIEW requires independent review again.

---

# Step 10 — Pull Request

The pull request gives the Tech Lead what is needed for final review:

- Task reference (`TASK-NNN`) and path to the task file
- Summary and implementation notes
- ADR references, where applicable
- Validation performed: actual commands and results
- Independent review result and reviewer
- Known risks and limitations
- Unresolved non-blocking findings, with justification

Report only results that have occurred. Do not state that CI, review, or
tests passed before they have.

A pull request is not approval. Merging a pull request is not
architecture approval.

---

# Step 11 — READY FOR HUMAN APPROVAL

The Reviewer moves the task to READY FOR HUMAN APPROVAL only when the
[approval gate](../standards/task-specification.md#ready-for-human-approval)
is met:

- Implementation complete and acceptance criteria satisfied
- Validation completed, with actual results
- Independent review complete
- BLOCKER and HIGH findings resolved
- Required documentation updated
- Required ADRs still APPROVED and not superseded
- CI passes
- Remaining findings and risks disclosed

If a required ADR is superseded before completion, follow
[Supersession and Unfinished Tasks](../standards/architecture-decisions.md#supersession-and-unfinished-tasks).
The task does not proceed until the impact is assessed and resolved.

---

# Step 12 — Human Approval and Merge

The Tech Lead performs the final review and may:

| Decision        | Result                                                  |
| --------------- | ------------------------------------------------------- |
| Approve         | Pull request approved; proceeds to merge                |
| Request changes | Task → CHANGES REQUESTED                                |
| Block           | Task → BLOCKED, with the reason recorded                |
| Cancel          | Task → CANCELLED, with the reason recorded              |

Merge happens only after the Tech Lead's approval and any repository
protections (required checks, branch rules) are satisfied. Production
releases remain under human control and are outside this workflow.

## Approval, Merge, and DONE

These are three separate things:

| Event                 | Means                                             | Does not mean                         |
| --------------------- | ------------------------------------------------- | ------------------------------------- |
| Tech Lead approval    | The Tech Lead accepts the change for merge        | The task is DONE                      |
| Pull request merge    | The change is in `main`                           | The task is DONE, or any ADR approved |
| Task DONE             | Every Definition of Done item is met and recorded | A production release                  |

The task stays READY FOR HUMAN APPROVAL through approval and merge, and
becomes DONE only in Step 13.

---

# Step 13 — DONE

After merge, the Tech Lead moves the task to DONE only when the
[AGENTS.md Definition of Done](../AGENTS.md) is satisfied. No agent marks
a task DONE.

Record in the task's Status History: the merged pull request, the
review result, and validation evidence.

Follow-up work outside the approved scope becomes a new task. Do not
expand a completed task.

---

# Exceptions

Any deviation from this workflow requires explicit Tech Lead
authorization, recorded in the task or pull request. An exception must
still:

- Preserve security controls and required approvals
- Remain auditable in the repository
- Be followed by any documentation or review that was deferred

Urgency alone is not an exception. It does not bypass review, security,
or testing.

---

# Example: Starting Implementation

Once `TASK-021` is READY FOR IMPLEMENTATION and assigned, the Tech Lead
can start the Assigned Agent with:

```
Implement TASK-021.
Follow AGENTS.md and all documents referenced by the task.
```

This works only when the task is READY FOR IMPLEMENTATION. If it is
DRAFT or BLOCKED, the agent must stop and report instead of starting.

A review can be started the same way:

```
Review TASK-021.
Follow AGENTS.md and all documents referenced by the task.
Do not modify files.
```
