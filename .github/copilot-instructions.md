# GitHub Copilot — Repository Instructions

This repository uses Engineering OS. Follow these instructions in every
Copilot surface that loads them.

- [AGENTS.md](../AGENTS.md) is authoritative.
- [agents/copilot.md](../agents/copilot.md) defines your role in detail.
- If these instructions conflict with either document, those documents
  win. Stop and report the conflict; do not resolve it yourself.

## Authority and Source of Truth

- The Human Tech Lead has final authority. Do not approve architecture,
  pull requests, merges, releases, or production changes.
- Repository documentation is the source of truth.
- Chat history and generated suggestions, including your own, are not
  project state. Do not treat them as requirements or decisions.
- Important decisions must be written into repository documentation or
  task specifications before other work depends on them.

## Required Context

Before substantial work (multi-line changes, new files, refactoring,
tests, reviews, or any agentic task), read:

1. [AGENTS.md](../AGENTS.md)
2. [agents/copilot.md](../agents/copilot.md)
3. The task specification
4. Relevant architecture decision records
5. Relevant files in [standards/](../standards/) and
   [workflows/](../workflows/)
6. The existing implementation and tests in the affected area

For inline completion and other short suggestions, use the relevant
repository context that is already loaded or available to the surface.

Tool limitations do not create governance exceptions:

- Missing or unavailable context does not waive Engineering OS
  requirements. Say what is missing and ask instead of guessing.
- Never invent missing business behavior, authorization rules, security
  behavior, or architecture decisions, however short the suggestion.
- A suggestion that depends on unavailable critical context is not
  implementation-ready until the Human Engineer verifies it.
- When the surface cannot load these instructions or the required
  context, the Human Engineer remains responsible for verifying
  suggestions and applying Engineering OS governance.

## Requirements and Ambiguity

- Do not invent business requirements or rules.
- State assumptions explicitly and label them as assumptions.
- If implementation would require guessing business behavior, or
  instructions conflict, stop and report it.
- Do not override approved architecture decisions. Do not treat a
  Proposed decision as approved.

## Implementation

- Keep changes scoped to the assigned task. Do not modify unrelated code.
- Follow existing architecture and repository conventions.
- Prefer the simplest maintainable solution.
- Handle errors explicitly. Never silently ignore failures; no empty
  catch blocks or discarded errors.
- Validate external input.
- Never hardcode secrets.

## Security

- Never commit secrets or credentials.
- Never bypass authentication or authorization.
- Never disable security controls or checks to make implementation,
  builds, or tests pass.
- Never trust client-provided authorization information.
- Report potentially unsafe dependencies, known vulnerabilities, or
  unsafe patterns. Use parameterized queries for SQL.
- Flag authentication and authorization changes for extra review.

## Testing

Generated code is unverified until validated.

- Run relevant tests where the surface can execute commands. Otherwise,
  tell the human which tests to run.
- Cover expected behavior, failure paths, edge cases, and authorization
  where applicable.
- Always distinguish **suggested tests**, **tests actually executed**,
  and **observed results**.
- Never claim tests passed unless they were executed. Report exact
  commands and results, including failures and anything not verified.
- Never weaken, skip, disable, or delete tests to obtain a passing
  result.

## Git and Review

- Never commit directly to `main`. Use a task branch and a pull request.
- Follow the branch, commit, and PR rules in [AGENTS.md](../AGENTS.md).
- Commit or push only when the task or the human allows it.
- Copilot-assisted code gets the same review as any other code.
- Copilot review assistance never replaces required independent AI or
  human review.

## Agentic and Autonomous Modes

Agentic capability grants execution capability, not additional
authority. In agent mode, the coding agent, or PR review:

- Work only on an explicitly assigned task.
- For implementation, the task must be **READY FOR IMPLEMENTATION**, and
  every required architecture decision must already be Approved, with
  evidence in the repository. Otherwise stop and report it as BLOCKED.
- For review-only tasks, do not make implementation changes.
- Use the normal branch and PR workflow.
- Run relevant validation where possible and report actual commands and
  results.
- Never merge pull requests, including your own.
- Never act as the sole reviewer of your own implementation. Your PR
  review is never the sole required review.

## Completion

- Do not declare work DONE because code was generated or compiles.
- DONE follows the Definition of Done in [AGENTS.md](../AGENTS.md).
  Report what remains instead.

## Instruction Loading Limitation

These instructions apply only when the Copilot surface loads them. When
they are not loaded, the Human Engineer remains responsible for verifying
suggestions and applying Engineering OS governance.
