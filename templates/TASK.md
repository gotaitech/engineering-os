# TASK-NNN: <Title>

<!--
Follow standards/task-specification.md.
Do not remove sections. Write "N/A — <reason>" where a section does not apply.
Reference repository files, not chat history.
-->

## Metadata

| Field          | Value                                                        |
| -------------- | ------------------------------------------------------------ |
| Status         | DRAFT                                                        |
| Priority       | <Set by Tech Lead>                                           |
| Assigned Agent | <Claude Code \| Codex \| Copilot (agentic) \| Human Engineer> |
| Reviewer       | <Independent reviewer; not the Assigned Agent>               |
| Scope Approval | <Tech Lead, date, reference — or "pending">                  |
| Branch         | <feature\|fix\|refactor>/TASK-NNN-short-description          |

## Goal

<The outcome and why it matters, in 1–3 sentences.>

## Context

<Background the implementer needs. Link repository files instead of
restating them.>

## Requirements

<!-- Confirmed requirements only. Put unconfirmed items under Assumptions
or Open Questions. -->

- <Required behavior>
- <Non-functional requirement, if any>

## Acceptance Criteria

<!-- Observable and testable. Include failure, edge, and authorization
cases where applicable. Given / When / Then is optional. -->

- [ ] <Criterion>
- [ ] <Failure or edge case>

## Dependencies

- <TASK-NNN, service, or decision this task depends on — or N/A>

## Constraints

- <Technical, security, compatibility, or scope limits>
- Out of scope: <what must not change>

## Required Reading

- `AGENTS.md`
- `<path>` — <why it is relevant>

## Architecture Decisions

<!-- Every required decision must be Approved. Otherwise Status is BLOCKED. -->

| Decision record | Status | Approval evidence |
| --------------- | ------ | ----------------- |
| `<path>` | <Approved \| Proposed \| Rejected \| Superseded> | <approver, date, reference> |

## Security Considerations

<!-- Authentication, authorization, secrets, external input, sensitive
data, dependencies. Auth changes require extra review. -->

- <Consideration — or N/A — reason>
- Tech Lead approval for security-sensitive decisions: <reference | pending | N/A>

## Testing Expectations

- <What must be tested: expected behavior, failure paths, edge cases,
  authorization>

## Validation Commands

<!-- Expected commands, not results. Report actual results in the handoff. -->

```sh
<command>
```

## Assumptions

- <Assumption — awaiting confirmation by whom>

## Open Questions

- <Question> — <critical | non-critical> — <who must answer>

## Definition of Done

- [ ] AGENTS.md Definition of Done is met.
- [ ] Human Tech Lead approval obtained.
- [ ] <Task-specific item, if any>

## Status History

<!-- Record moves to or from BLOCKED, CHANGES REQUESTED,
READY FOR IMPLEMENTATION, and DONE. Blockers go here with what they wait on. -->

| Date | From → To | By | Reason / evidence |
| ---- | --------- | -- | ----------------- |
| <YYYY-MM-DD> | (new) → DRAFT | <who> | <reason> |
