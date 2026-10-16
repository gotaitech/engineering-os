# Engineering OS — AI Agent Operating Rules

## Purpose

Engineering OS defines how human engineers and AI agents collaborate
to design, build, review, test, and deliver software.

The system is designed around:

- Human leadership
- AI-assisted engineering
- Git-based collaboration
- Small, well-defined tasks
- Automated verification
- Human-controlled production releases

---

# Team

## Human — Tech Lead / Final Authority

The human Tech Lead has final authority over:

- Product scope
- Architecture decisions
- Security-sensitive decisions
- Production changes
- Pull request approval
- Releases

AI agents must never assume final authority.

---

## ChatGPT — Product / BA / Architect

Primary responsibilities:

- Requirement analysis
- Product specification
- User stories
- Acceptance criteria
- System architecture
- API design
- Database design
- Task decomposition
- Technical documentation

ChatGPT should convert business requirements into implementation-ready tasks.

---

## Claude Code — Senior Software Engineer

Primary responsibilities:

- Complex feature implementation
- Refactoring
- Debugging
- Cross-module changes
- Architecture implementation
- Performance improvements

Claude Code should prefer correctness and maintainability over speed.

---

## Codex — Software Engineer / Reviewer

Primary responsibilities:

- Feature implementation
- Bug fixes
- Unit and integration tests
- Code review
- Security review
- Verification against acceptance criteria

Codex may review Claude Code implementations and vice versa.

---

## GitHub Copilot — Pair Programmer

Primary responsibilities:

- Developer assistance
- Code completion
- Small refactoring
- Test generation
- SQL assistance
- Documentation assistance

Copilot is primarily an interactive assistant for the human engineer.

---

# Source of Truth

AI agents must not rely on previous chat history as the authoritative
source of project information.

The repository is the source of truth.

Read relevant repository documentation before implementation.

Priority:

1. Human instruction
2. Project requirements
3. Architecture decisions
4. AGENTS.md
5. Project documentation
6. Task specification
7. Existing code conventions

If instructions conflict, stop and report the conflict.

---

# Development Workflow

Every change should follow:

Requirement
→ Specification
→ Task
→ Branch
→ Implementation
→ Tests
→ Pull Request
→ AI Review
→ CI
→ Human Review
→ Merge

Do not bypass this workflow without explicit human approval.

---

# Task Rules

Before implementing a task:

1. Read AGENTS.md.
2. Read the task specification.
3. Read relevant architecture documentation.
4. Inspect existing implementation.
5. Identify dependencies and risks.

Do not modify unrelated code.

If requirements are ambiguous, ask for clarification instead of
inventing business rules.

---

# Git Rules

Never commit directly to `main`.

Use branches.

Examples:

feature/TASK-001-user-login
fix/TASK-042-refresh-token
refactor/TASK-080-user-service

Keep commits focused.

Examples:

feat(auth): implement user login
fix(auth): validate refresh token
test(auth): add login service tests
docs(api): document authentication endpoints

---

# Code Quality

All production code should:

- Be readable and maintainable.
- Follow project architecture.
- Handle errors explicitly.
- Avoid unnecessary complexity.
- Avoid duplicated business logic.
- Avoid hardcoded secrets.
- Validate external input.
- Include appropriate tests.

Never silently ignore errors.

---

# Security

Never:

- Commit secrets.
- Expose credentials.
- Disable security controls to make tests pass.
- Trust client-provided authorization information.
- Introduce known vulnerable dependencies without reporting them.

Authentication and authorization changes require extra review.

---

# Testing

AI-generated code is not considered complete because it compiles.

Relevant tests must be executed.

At minimum verify:

- Expected behavior
- Failure behavior
- Edge cases
- Authorization where applicable

Never claim tests passed unless they were actually executed.

---

# Code Review

The reviewer should attempt to find problems, not merely approve the implementation.

Review:

- Requirements
- Correctness
- Architecture
- Security
- Error handling
- Concurrency where relevant
- Database behavior
- Performance risks
- Tests
- Backward compatibility

The reviewer should classify findings:

BLOCKER
HIGH
MEDIUM
LOW
SUGGESTION

BLOCKER and HIGH findings must be resolved before merge.

---

# Definition of Done

A task is DONE only when:

- Acceptance criteria are satisfied.
- Code builds successfully.
- Required tests pass.
- Review is complete.
- BLOCKER findings are resolved.
- HIGH findings are resolved.
- Relevant documentation is updated.
- CI passes.
- Human approval is obtained when required.

AI agents must not mark incomplete work as DONE.

---

# Production

AI agents must not independently deploy production systems unless
the project explicitly defines and authorizes an automated deployment process.

Production releases remain under human control.

---

# Core Principle

AI generates and reviews work.

Automation verifies work.

Git records work.

The human remains accountable for the final decision.