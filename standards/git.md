# Git Standard

This standard defines practical Git collaboration for humans and AI agents
in Engineering OS. [AGENTS.md](../AGENTS.md) is authoritative. If instructions
conflict, stop and report the sources and impact to the Human Tech Lead;
do not resolve the conflict independently.

Task readiness and transitions belong to
[standards/task-specification.md](task-specification.md). Architecture
approval belongs to [standards/architecture-decisions.md](architecture-decisions.md).
Follow [workflows/feature.md](../workflows/feature.md) for delivery and the
relevant [agent definition](../agents/) for permissions. This standard
neither grants Git permissions nor replaces those gates.

---

# Core Principles

- `main` represents integrated, reviewed work. Normal feature, fix, and
  refactor work must not be implemented directly on `main`.
- Never commit directly to `main`. Pull requests are the normal integration
  path; agents must not push directly to `main`.
- Work is traceable to a repository-backed task where applicable. Normal
  implementation starts only after assignment and readiness verification.
- Keep branches focused and short-lived, and commits understandable and scoped.
- The Human Tech Lead retains final approval and merge authority. An AI
  review result does not authorize integration.
- Never manipulate history to hide defects, failed review, unauthorized
  changes, security issues, or validation failures.

---

# Updating Local Main

Before starting normal work, inspect the current branch and working tree:

```sh
git status
git diff
git diff --cached
```

If there are uncommitted changes, preserve them. Do not automatically
stash, commit, discard, or move them into a different task. Establish who
owns the changes and a safe way to preserve them before switching branches.
Report conflicts or unclear ownership to the human.

Once it is safe to switch, update local `main`, then create the task branch:

```sh
git checkout main
git pull --ff-only origin main
```

Check each command's result before continuing. `--ff-only` permits only a
fast-forward and prevents Git from silently choosing merge/rebase behavior
when local `main` has diverged. If fast-forward fails:

STOP → inspect → do not automatically reset, rebase, or merge.

Unexpected local commits or divergence require an understood, authorized
resolution before continuing. A destructive reset is not normal
synchronization.

The example assumes `origin` is the intended project remote. Use the
project's documented remote if different. Do not switch away from an
existing task branch merely to perform this startup sequence again.

---

# Branch Naming

Use:

```text
<type>/<TASK-ID>-<short-description>
```

Use the task's exact ID and a lowercase kebab-case description. Examples:

```text
feature/TASK-021-user-login
fix/TASK-042-refresh-token
refactor/TASK-080-user-service
docs/TASK-010-git-standard
```

Types include `feature/`, `fix/`, `refactor/`, `docs/`, `test/`, and `chore/`.
Use an existing suitable type instead of inventing a new one.

A task ID may be omitted only for repository maintenance that legitimately
has no task and is allowed by the Human Tech Lead and project governance.
Record that authorization in the PR. This exception does not waive normal
task readiness for implementation or any review requirement.

---

# Branch Scope

One branch normally represents one coherent task or change. Do not mix
unrelated features, bug fixes, drive-by refactors, formatting, or dependency
upgrades. Include supporting work only when genuinely required by the
approved scope.

Report newly discovered work and create or follow a separate task unless
it is already within the approved scope. Scope or architecture changes
follow task-standard BLOCKED recovery and renewed approvals; a convenient
Git branch does not authorize them.

---

# Commit Standard

Use clear, focused commits. Prefer Conventional Commit style:

```text
<type>(<optional-scope>): <actual change>
```

Types include `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `build`,
and `ci`. Examples:

```text
feat(auth): add refresh token rotation
fix(payroll): prevent duplicate payroll run
docs(review): define code review standard
test(auth): cover expired refresh tokens
```

Messages describe the actual change. Avoid vague messages such as `update`,
`changes`, `fix stuff`, `final`, or `working` when a meaningful description
is available. Add context or a reason in the body when useful.

Do not split a coherent change artificially. Multiple commits are useful
when they separate understandable changes, not when they create ceremony.
Keep the task relationship discoverable through branch, commit context,
and PR references; a task ID need not be repeated on every line.

---

# AI Commit Rules

AI agents may edit only within their assigned scope and permissions. They
may commit, push, or open a PR only when an explicit Human Tech Lead
instruction or explicit task instructions grant that specific action.
Being the Assigned Agent, having repository access, having a capable tool,
or having permission to edit does not itself grant commit, push, PR
creation, or merge authority.

Tool permissions must also allow the operation; they never supply authority
on their own. Explicit instructions such as **Do not commit** and
**Do not push** must be obeyed. Commit/push/PR authority does not grant
merge authority.

Agents must not:

- Commit unrelated user changes, including previously staged changes.
- Discard user changes or rewrite unrelated history.
- Push directly to `main` or bypass required review through Git operations.
- Treat an example command, available credential, or successful check as
  authorization.

Inspect existing changes before editing and before committing. Preserve
pre-existing changes, stage only authorized content, and report conflicts
or ambiguity. Do not unstage or overwrite another contributor's work as
an automatic cleanup step. If unrelated changes are already staged and the
intended commit cannot safely be created without altering another user's
staging state, STOP and report/ask. Do not silently unstage, discard,
include, or rewrite those changes.

---

# Staging

Prefer explicit staging of task files:

```sh
git add path/to/file1 path/to/file2
```

Before committing, review:

```sh
git status
git diff
git diff --cached
```

Verify the complete staged diff, including files staged before this task.
If a file contains unrelated edits, stage only the intended hunks or ask
for clarification. A path belonging to the task does not make every edit
in that path authorized.

`git add .`, `git add -A`, and `git commit -a` are bulk operations. Use them
only after verifying that all included changes belong to the intended
commit, including any already-staged content. They must not be the default
for AI agents. Never stage or commit blindly.

---

# Pull Requests

Keep PRs focused and reviewable. Use the repository template where present.
Include, as applicable:

- Task ID and repository path to the task specification
- Summary and important implementation decisions
- Relevant approved ADR references
- Actual validation commands and results, including failures and gaps
- Independent review status and reviewer
- Known risks and unresolved non-blocking findings with justification
- Migration, configuration, and breaking-change notes

Expected validation is not executed validation. Do not claim tests, CI, or
review passed unless evidence supports that claim. A draft PR may be opened
early when authorized; report pending work accurately.

PR creation does not mean code approval, architecture approval, task DONE,
or production deployment authorization. Merging an implementation PR that
contains or references an ADR is not architecture approval. Required ADRs
must be approved beforehand through the ADR standard's explicit evidence
process.

GitHub web UI is a valid default for PR creation and merge. GitHub CLI is
optional; `gh` is not an Engineering OS dependency.

---

# Keeping Branches Current

When `main` advances, use the repository or project's chosen integration
strategy. Do not impose rebase or merge universally.

If no strategy is documented, use the least disruptive approach: fetch and
inspect the changes, then integrate without rewriting shared history.
A merge of updated `main` into a task branch is an option when it preserves
history and fits the change. If the choice is unclear, ask the Tech Lead.
Rebasing a shared branch requires explicit authorization.

Preserve local changes before integration. Resolve conflicts deliberately,
inspect the resulting diff, and rerun affected validation. Changes after
review follow the rules below; previous review and CI results must not be
presented as evidence for unverified new content.

## Changes After Review Starts

These rules apply when updating from `main` or resolving conflicts changes
task content. Task status transitions and their ownership remain defined
by [standards/task-specification.md](task-specification.md#transitions).

- **During IN REVIEW:** the Assigned Agent reports the update to the
  Reviewer, identifying the previously reviewed revision, the new revision,
  and the changed content. The Reviewer determines the required re-review
  scope. Existing review does not cover unreviewed changes. Rerun affected
  validation, report actual results, and follow the task standard for any
  required status change. Do not let review continue against an unidentified
  moving target.
- **During READY FOR HUMAN APPROVAL:** if synchronization, conflict
  resolution, or another Git operation changes reviewed content, the
  Assigned Agent must STOP and report the changed content to the Human
  Tech Lead **before pushing it to the open PR branch**. Do not first
  publish changed or unreviewed content to the approval target, and do not
  silently push it while retaining READY FOR HUMAN APPROVAL. Wait for the
  Tech Lead to determine the lifecycle action according to the task
  standard. Where correction is required, use the existing CHANGES
  REQUESTED path; do not invent a direct transition back to IN REVIEW.
  Affected validation and independent review must be repeated before the
  task can return to READY FOR HUMAN APPROVAL. Earlier review or approval
  cannot authorize merging changed, unreviewed content.
- **No relevant content change:** record the synchronization revision and
  the assessment that reviewed task content is unchanged in the PR or
  handoff where appropriate. Do not imply new validation was executed.

If the update invalidates requirements, scope, architecture, or another
readiness condition, follow the task standard's BLOCKED recovery and
renewed readiness verification. Git synchronization does not resolve a
governance blocker.

---

# Destructive Operations and History Rewriting

Any Git operation that can discard user work, delete remote/shared work,
or rewrite published history requires explicit authorization for the
specific situation and an understood impact. For AI agents, authorization
must come from the Human Tech Lead. High-risk examples include:

- `git push --force`
- `git push --force-with-lease`
- `git reset --hard`
- `git clean -fd` (including broader destructive clean variants)
- Destructive interactive rebase or equivalent history rewriting
- `git restore .` or `git checkout -- <path>` when they could discard work
  that is not known to belong solely to the agent
- `git stash drop` or `git stash clear`
- `git branch -D`
- `git push --delete`
- `git commit --amend` on already-pushed commits

These examples are not an exhaustive whitelist. Evaluate the actual effect,
ownership, and publication state, including equivalent commands or variants.
Ordinary scoped use of `git restore` for explicitly identified, uncommitted
changes created solely by the agent within its authorized task is not
categorically forbidden. Inspect the exact diff and path first; if ownership
or impact is uncertain, preserve the work and ask rather than discard it.

Describe the affected branches, commits, local files, and collaborators
before authorization. Tool permission alone is insufficient.
`--force-with-lease` does not waive this requirement.

Never rewrite `main` or shared protected history to make review easier.
Preserve review and validation evidence. Secret incident remediation is
coordinated separately by the Tech Lead; it is not an automatic license
to rewrite history.

---

# Merge Rules

Normal work merges through a PR only after required validation,
independent review, CI, Human Tech Lead approval, and repository protections
are satisfied. Run CI when configured; if required CI is missing or cannot
run, report the gap rather than silently waiving the task's completion gate.
BLOCKER and HIGH findings must be resolved before merge. MEDIUM findings
follow the task standard's correction or justified-deferral rules.

AI agents must not merge pull requests. The Human Tech Lead controls final
merge according to AGENTS.md and the feature workflow. An implementation
agent cannot merge its own work merely because review returned PASS.

Merge commit, squash, or rebase merge is project-specific. Follow the
chosen strategy without imposing a global default. Preserve the task,
review, and validation relationships when squashing or changing commit IDs.

After merge, delete completed short-lived branches when they are no longer
needed and branch deletion is authorized. Do not delete active or unmerged
work automatically. Deleting a merged branch does not delete task, ADR,
review, or validation history.

Human approval, merge, and DONE are separate events. Under the feature
workflow, the task remains READY FOR HUMAN APPROVAL through approval and
merge. Only the Tech Lead records DONE after the AGENTS.md Definition of
Done is met. Merge does not authorize production deployment.

---

# Conflict Handling

For merge or rebase conflicts:

1. Inspect both sides and understand their intended behavior.
2. Preserve approved requirements and architecture; do not silently drop
   either side's intended changes.
3. Resolve deliberately, review the full resulting diff, and check for
   leftover conflict markers.
4. Rerun relevant validation and report actual results.
5. Apply [Changes After Review Starts](#changes-after-review-starts) when
   review has begun, and obtain independent review of affected content
   before merge.

Never blindly choose `ours` or `theirs`. If resolving the conflict requires
a business, architecture, scope, or security-sensitive decision:

STOP → report to the Tech Lead → follow task governance and readiness rules.

Do not invent the decision while resolving Git conflicts. Record the issue
and follow BLOCKED recovery when readiness is invalidated. Resume only
after repository specifications and required approvals are updated and
readiness is verified again.

---

# Sensitive Data

Never commit passwords, access tokens, API secrets, private keys,
production credentials, or secret environment files. Do not commit sensitive
customer data unless storage in the repository is explicitly designed and
approved. This allowance for data never permits committing credentials.

If a secret is accidentally committed, stop further propagation and report
it immediately to the Tech Lead without reproducing the secret in the
report. Deleting it from the latest version is insufficient: credential
revocation or rotation and history remediation may be required, even if
the commit was not pushed.

The Tech Lead coordinates the response. Do not automatically run a generic
history-rewrite procedure or assume a rewritten repository revokes exposed
credentials.

---

# Generated and Large Files

Do not commit build artifacts, dependency directories, logs, local IDE
files, or generated output unless the repository intentionally tracks them.
Evaluate large files against project storage rules before adding them.
Respect `.gitignore` and project-specific rules.

Do not alter `.gitignore` merely to hide unexpected files. Understand their
source, whether they belong in the repository, and whether they contain
secrets before deciding how to handle them.

---

# Traceability

Where applicable, preserve these relationships through repository paths,
branch names, commit context, and PR links:

TASK ↔ Branch ↔ Commits ↔ PR ↔ Review / Validation ↔ Merge

Reference applicable ADRs where architecture affects the task. Keep actual
review and validation evidence discoverable after merge; do not depend on
hidden chat history. Apply task governance proportionally: authorized tiny
maintenance does not need artificial tasks or commit splitting, but retains
security, review, and human approval requirements.

---

# Emergency Changes

Urgency does not authorize direct pushes to `main`, bypassing security,
hiding validation failures, or bypassing required approval. Any exception
requires explicit Human Tech Lead authorization for that specific situation
and must be recorded in the task or PR with its scope, reason, and remaining
work. Preserve security controls and required approvals. Follow the feature
workflow's exception rules; do not treat emergency handling as a standing
permission or an expansion of an agent's authority.

---

# Practical Example

These are examples, not permission to commit or push. Run only actions
granted by explicit Human Tech Lead instruction or explicit task
instructions, and permitted by the tools. When told **Do not commit**
or **Do not push**, leave those commands unexecuted and hand off the diff.
The task must be assigned and READY FOR IMPLEMENTATION before work starts.
The startup example assumes a safely preserved working tree and local
`main` without unexpected divergence.

```sh
git status
git checkout main
git pull --ff-only origin main
git checkout -b feature/TASK-021-user-login

# Implement the approved task and run relevant validation.
git status
git diff

# Replace the paths below with only the authorized task files.
git add path/to/task-file1 path/to/task-file2
git diff --cached
git commit -m "feat(auth): implement user login"
git push -u origin feature/TASK-021-user-login
```

Check each command succeeds before running the next. Then use the GitHub
web UI (or optional CLI): PR → independent review → CI → Human Tech Lead
approval → merge. Follow the task lifecycle for corrections, governance
blockers, and completion; no Git command bypasses those rules.
