# Architecture Decision Record Standard

This standard defines how significant architecture decisions are proposed,
approved, rejected, superseded, and referenced. Use
[templates/ADR.md](../templates/ADR.md) to create an Architecture Decision
Record (ADR).

Architecture decisions must not depend on chat history. An architecture
decision exists only when it is recorded in the repository.

[AGENTS.md](../AGENTS.md) is authoritative. If this standard conflicts
with AGENTS.md, an agent definition in [agents/](../agents/), or
[standards/task-specification.md](task-specification.md), stop and report
the conflict to the Human Tech Lead. Do not resolve it yourself.
This standard grants no authority beyond what those documents allow.

This standard implements the decision status rules in
[agents/chatgpt.md](../agents/chatgpt.md#decision-status). The statuses
PROPOSED, APPROVED, REJECTED, and SUPERSEDED are the same as Proposed,
Approved, Rejected, and Superseded used there and in the task standard.

---

# When an ADR Is Required

Write an ADR for significant decisions, such as:

- System architecture
- Service boundaries
- Major technology choices
- Authentication or authorization architecture
- Data ownership
- Major database architecture
- Integration patterns
- Deployment architecture
- Security-sensitive architecture
- Decisions that are difficult or expensive to reverse

An ADR is normally not required for:

- Formatting
- Naming
- Minor refactoring
- Local implementation details that do not affect architecture

If it is unclear whether a decision is significant, ask the Tech Lead.
Do not skip an ADR to avoid the approval step.

---

# ADR Identity and Location

- ADR IDs use `ADR-NNN`, for example `ADR-001`, `ADR-012`, `ADR-105`.
- Use the next unused number, zero-padded to at least three digits.
  After `ADR-999`, continue with `ADR-1000`.
- Never reuse an ID, including for rejected or superseded ADRs.
- Store ADRs in `docs/adr/` as `ADR-NNN-short-title.md`, unless the
  project defines another location.
- Before assigning an ID, check existing ADR files and open branches.
  If two ADRs receive the same ID, report it to the Tech Lead.

---

# Status

| Status     | Meaning                                                                         | Usable by tasks |
| ---------- | ------------------------------------------------------------------------------- | --------------- |
| PROPOSED   | A proposal exists but is not approved for implementation.                       | No              |
| APPROVED   | The Human Tech Lead has explicitly approved the decision, with evidence.        | Yes             |
| REJECTED   | The Human Tech Lead rejected the proposal. Kept for history.                    | No              |
| SUPERSEDED | A newer APPROVED ADR replaces this decision. Kept for history.                  | No; use the replacement |

Every ADR has exactly one status at a time.

---

# Authority

The Human Tech Lead has final architecture authority. Only the Tech Lead
approves, rejects, or supersedes an ADR.

## Authority Is Not Authorship

Approval authority comes from role, not from who drafted the ADR.

- **AI authorship or contribution never grants approval authority.**
  No AI agent approves an ADR, whether it authored it, contributed to
  it, reviewed it, or had no involvement.
- **The Human Tech Lead's authority applies regardless of who drafted
  the ADR.** The Tech Lead may author an ADR and also approve it. No
  second human approver is required.
- Every approval, including one of a Tech Lead-authored ADR, must be
  explicit and auditable as defined in [Approval Evidence](#approval-evidence).

| Who                  | May                                                                                   | Must not                                                                 |
| -------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Human Tech Lead      | Approve, reject, or supersede ADRs; request changes to proposals                      | —                                                                        |
| ChatGPT              | Analyze architecture, compare alternatives, document trade-offs, write and update PROPOSED ADRs | Approve any ADR, including its own; mark an ADR APPROVED without Tech Lead approval evidence; silently choose unresolved architecture for implementation |
| Claude Code, Codex   | Give technical feedback, identify risks, propose alternatives, implement APPROVED architecture | Approve any ADR, including ones they contributed to or that their own work requires |
| GitHub Copilot       | Assist the human with ADR content                                                     | Approve architecture                                                     |

Recommendations, review results (including PASS), silence, lack of
objection, and continued discussion are never approval.

---

# Required Content

Every ADR contains:

1. ADR ID
2. Title
3. Status
4. Date
5. Decision Owner
6. Decision
7. Context
8. Drivers
9. Considered Options
10. Decision Rationale
11. Consequences
12. Risks
13. Security Considerations
14. Operational Considerations
15. Related Tasks
16. Related ADRs
17. Approval Evidence
18. Superseded By, when applicable

The template also includes Status History, which records every status
change.

A section that does not apply contains `N/A` and a one-line reason.
Required sections must not be removed or left silently empty.

Field notes:

- **Date** is the date the ADR was first proposed. Later status changes
  are dated in Status History.
- **Decision Owner** is the Human Tech Lead, or the human the Tech Lead
  names as accountable for the decision. It is never an AI agent.
- **Author** records who drafted the ADR. Authorship does not affect
  approval authority; see [Authority Is Not Authorship](#authority-is-not-authorship).

---

# Considered Options

- For significant choices, record the meaningful alternatives that were
  actually considered, including their trade-offs.
- Do not invent alternatives to fill the template. If only one option is
  viable, say so and explain why.
- The Decision Rationale explains why the selected option is preferred
  over the others.

---

# Approval Evidence

An ADR is **APPROVED** only when all of the following are recorded in the
repository:

1. The ADR's Status is `APPROVED`.
2. The Approval Evidence section names the Human Tech Lead as approver,
   gives the approval date, and gives an approval reference.
3. The Status History contains a `PROPOSED → APPROVED` entry by the
   Tech Lead, with the same reference.
4. The change that sets the status is merged through the normal pull
   request workflow.

A valid approval reference is one of:

- **An ADR approval pull request:** a pull request whose stated purpose
  includes approving this ADR, approved or merged by the Tech Lead.
- **A recorded Tech Lead instruction:** a written approval by the
  Tech Lead committed to the repository (for example, in the ADR's
  Status History or Approval Evidence), identifying the ADR by ID.

Not approval:

- A recommendation or review result from ChatGPT, Claude Code, Codex, or
  Copilot.
- Approval stated only in chat.
- Merging an implementation pull request that happens to contain or
  reference the ADR.
- An agent setting the Status to `APPROVED` without one of the valid
  references above.

An agent may edit the Status line or Status History to record an
approval only on the Tech Lead's instruction. The edit takes effect only
when the Tech Lead approves and merges it.

When the Tech Lead authored the ADR, the same evidence applies: a
`PROPOSED → APPROVED` Status History entry by the Tech Lead with a
reference. If the platform does not allow approving one's own pull
request, the Tech Lead's recorded instruction plus the merge is the
reference. The ADR may be created and approved in the same pull request,
with both Status History entries recorded.

Because a task that requires an ADR cannot be READY FOR IMPLEMENTATION
until the ADR is APPROVED, ADR approval is not bundled into the
implementation pull request for that task.

Rejections and supersessions are recorded the same way, with the
Tech Lead as decider and a reference.

---

# Status Transitions

| From       | To         | Who                     | Condition                                                             |
| ---------- | ---------- | ----------------------- | --------------------------------------------------------------------- |
| (new)      | PROPOSED   | Tech Lead, ChatGPT, or an agent the Tech Lead directs | ADR drafted with all required sections    |
| PROPOSED   | PROPOSED   | Author                  | Proposal updated before a decision; changes noted in Status History  |
| PROPOSED   | APPROVED   | Tech Lead only          | [Approval evidence](#approval-evidence) recorded                      |
| PROPOSED   | REJECTED   | Tech Lead only          | Reason and reference recorded                                         |
| APPROVED   | SUPERSEDED | Tech Lead only          | A newer ADR that replaces it is APPROVED                              |

Not permitted:

- Any transition out of REJECTED or SUPERSEDED. To revisit a rejected
  or superseded decision, write a new ADR that references it.
- APPROVED → PROPOSED or APPROVED → REJECTED. To change or reverse an
  approved decision, write a new ADR that supersedes it.
- Any agent moving an ADR to APPROVED, REJECTED, or SUPERSEDED.

---

# Changes, History, and Superseding

Approved ADRs are historical decision records. Do not rewrite them to
make the original decision appear different.

## Material Changes

A material change alters the decision, its scope, its rationale, its
consequences, or what implementers must do. For a material change:

1. Write a new ADR with status PROPOSED. Reference the old ADR in
   Related ADRs and explain what changes and why.
2. The Tech Lead approves the new ADR.
3. The Tech Lead marks the old ADR SUPERSEDED.
4. Record both directions: the old ADR's **Superseded By** names the new
   ADR; the new ADR's Related ADRs names the one it supersedes.
5. Assess every unfinished task that depends on the old ADR, as defined
   in [Supersession and Unfinished Tasks](#supersession-and-unfinished-tasks).

The old ADR stays APPROVED until the new ADR is APPROVED. A proposal to
replace it does not change its status.

A new ADR replaces the whole earlier decision. If parts of the earlier
decision still apply, restate them in the new ADR.

## Minor Corrections

These are allowed on an APPROVED ADR without a new ADR, through the
normal pull request workflow:

- Typos, grammar, formatting, and broken links
- Adding entries to Related Tasks or Related ADRs
- Filling in **Superseded By** when the ADR is superseded

Record minor corrections in Status History. If there is any doubt
whether a change is material, treat it as material.

---

# Task Integration

This section applies [standards/task-specification.md](task-specification.md)
to ADRs.

If a task requires an architecture decision:

- The ADR exists in the repository.
- The ADR is APPROVED, with approval evidence as defined above.
- The task references the ADR by ID and path in its Architecture
  Decisions section, with its status and approval reference.

The task cannot be READY FOR IMPLEMENTATION, and is BLOCKED, while a
required ADR is:

- PROPOSED
- REJECTED
- SUPERSEDED without an APPROVED replacement
- Missing

When ADR-002 supersedes ADR-001, tasks reference the currently
applicable APPROVED ADR (ADR-002).

Related Tasks in the ADR lists the tasks that implement or depend on it.

## Supersession and Unfinished Tasks

When an APPROVED ADR is superseded, every unfinished task that depends
on it must be assessed. Find affected tasks through the old ADR's
Related Tasks and a repository search for its ID.

Unfinished tasks include tasks in:

- READY FOR IMPLEMENTATION
- IN PROGRESS
- IN REVIEW
- CHANGES REQUESTED
- READY FOR HUMAN APPROVAL
- BLOCKED, where the blocker does not already cover the change

For each affected task, ChatGPT or the Tech Lead:

1. Identifies the currently applicable replacement ADR.
2. Performs an impact assessment.
3. Updates the task's Architecture Decisions references.
4. Determines whether requirements, acceptance criteria, implementation,
   tests, validation, or security review are affected.
5. Performs readiness verification again where required.
6. Obtains any required Tech Lead approvals.
7. Ensures implementation, validation, and independent review are
   repeated where the architecture change affects them.

The outcome is recorded in the task's Status History with a reference
to the replacement ADR.

- **No impact:** the task keeps its status, with the updated reference
  and the recorded assessment.
- **Any impact:** the change is readiness-invalidating under
  [standards/task-specification.md](task-specification.md). The task
  moves to BLOCKED and follows the BLOCKED recovery process. Work and
  review affected by the change are repeated before the task can return
  to READY FOR HUMAN APPROVAL.

Until the assessment is recorded and any impact is resolved, an affected
task must not advance. In particular:

- **READY FOR HUMAN APPROVAL → DONE is not permitted** while the task
  depends on a superseded ADR whose impact has not been assessed and
  resolved. Reaching READY FOR HUMAN APPROVAL earlier does not exempt a
  task built on obsolete architecture.
- Anyone who notices a task depending on a superseded ADR may move it
  to BLOCKED, as the task standard allows.

## DONE and CANCELLED Tasks

DONE and CANCELLED tasks are historical records. Do not rewrite them or
their ADR references when an ADR is superseded. If completed work needs
to change to match the new ADR, create a new task for that follow-up
and reference both ADRs.
