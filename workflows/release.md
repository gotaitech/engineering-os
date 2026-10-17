# Release Workflow

This workflow takes completed, approved changes through release preparation,
production authorization, deployment, verification, recovery when needed, and
completion. It supports both manual deployment and explicitly authorized
automated deployment, without assuming either.

**Core principles:**

- Merge readiness is not release readiness.
- Release readiness is not deployment authority.
- Tool capability, repository permission, CI/CD capability, and production
  credentials are not authority.
- Automation may execute production changes only when that automation and the
  release path have been explicitly authorized.
- **Automatic execution is not automatic authority.**

[AGENTS.md](../AGENTS.md) is authoritative. The Human Tech Lead retains final
authority over production changes. This workflow connects existing rules; it
does not replace them:

| Topic | Owned by |
| --- | --- |
| Task statuses, transitions, DONE | [Task standard](../standards/task-specification.md) |
| Architecture approval and supersession | [ADR standard](../standards/architecture-decisions.md) |
| Review and finding severity | [Review standard](../standards/code-review.md) |
| Branches, commits, PRs, merge | [Git standard](../standards/git.md) |
| Security requirements and exceptions | [Security standard](../standards/security.md) |
| Database, migrations, recovery | [Database standard](../standards/database.md) |
| Backend and frontend behavior | [Backend](../standards/backend.md), [Frontend](../standards/frontend.md) |
| Delivery of individual changes | [Feature workflow](feature.md), [Bugfix workflow](bugfix.md) |

If instructions conflict, stop and report the conflict to the Tech Lead. Do
not resolve it yourself.

---

# 1. Overview

Normal flow:

Completed / approved changes → Release scope → Release candidate → Release
readiness → Validation → Security, data, compatibility, and operational checks
→ Deployment plan → Recovery plan → **Production authorization** → Deploy →
Deployment verification → Release verification → Observe → Complete release

On failure:

Stop / contain → Determine actual state → Recover (roll back, roll forward, or
other approved strategy) → Verify recovery → Record outcome → Follow-up task
or the project's incident process

These are workflow stages, not task statuses. This workflow creates no task
statuses and no incident lifecycle; projects use their own incident process
where one exists. Every stage scales with risk: a small, low-risk release does
not need a heavyweight plan for each stage.

---

# 2. Release Scope

Record what is being released, where applicable:

- Included PRs, commits, and tasks
- Release or version identifier
- Services and components (frontend, backend, mobile)
- Schema changes and migrations
- Configuration, infrastructure, and feature-flag changes
- Dependency and external integration changes
- Data remediation
- Operational, documentation, and runbook changes

The scope must be auditable enough to determine what changed. Do not silently
include unrelated work. If scope changes materially after readiness or
authorization, reassess the affected checks and obtain authorization again for
the changed scope.

---

# 3. Release Candidate

A release candidate is the specific set proposed for release. Identify, where
applicable:

- Source commit, tag, or version
- Build artifact, image, or package
- Configuration version
- Migration set
- Dependency lock state
- Infrastructure definition
- Mobile or client version

What was validated should be what is released, as closely as the delivery
architecture permits.

- Do not rebuild a materially different artifact after validation without
  revalidating it.
- Do not treat a moving branch head as an immutable candidate when stronger
  identification is available.

No versioning, tagging, or packaging strategy is mandated.

---

# 4. Release Readiness

Assess, in proportion to risk:

- Required tasks are in the required pre-release state, with reviews and
  Human approvals complete
- CI, tests, and build or packaging succeeded
- Security, architecture consistency, and approved ADRs
- Database and migration readiness
- Compatibility, configuration, secrets, and dependencies
- Operational readiness and observability
- Deployment plan and recovery plan
- Known risks, deferred issues, and required documentation

Release readiness is an assessment. It does not grant deployment authority.

---

# 5. Relationship to the Task Lifecycle

- Tasks reach their statuses only under the
  [task standard](../standards/task-specification.md). This workflow does not
  redefine them.
- A release may aggregate many changes. Releasing them does not move any task
  to DONE.
- Where a task's DONE criteria include deployment or post-deployment
  verification, the task stays in its current status (normally READY FOR
  HUMAN APPROVAL, as in the [feature workflow](feature.md)) until those
  criteria are met and the Tech Lead records DONE.
- Problems found during release that affect a task follow the task standard,
  including BLOCKED recovery where readiness is invalidated.

---

# 6. Relationship to Review

Review follows [standards/code-review.md](../standards/code-review.md) and its
severity levels. PASS means the reviewed change met review requirements. It
does not authorize merge, release, deployment, production data changes,
infrastructure changes, or DONE.

Release readiness never reopens, downgrades, or ignores unresolved review
findings. A release date never resolves a BLOCKER or HIGH finding.

---

# 7. Production Authority

The Human Tech Lead holds final authority over production changes, unless an
explicitly approved process delegates execution under defined conditions.

| Concept       | Meaning                                                    |
| ------------- | ---------------------------------------------------------- |
| Authorization | Permission for this production change to occur             |
| Execution     | The actor or mechanism that performs the authorized change |

An AI agent, CI/CD runner, deployment bot, repository integration, script, or
human operator may be able to execute a change without having authority to
decide that it should happen.

Production authorization is explicit and auditable under the project's
approved release process. It is never inferred from:

- PR approval or review PASS
- CI success or a successful staging deployment
- Merge permission or repository write access
- Deployment credentials, cloud access, or environment access
- Task assignment, agent autonomy, or tool capability

---

# 8. Automated Deployment

Explicitly authorized automated deployment is permitted. This workflow does not
prohibit it.

If merge, a tag, release creation, artifact publication, a branch update, or
another repository event automatically triggers production deployment:

- That trigger is part of the production execution path.
- Reviewers and approvers must be told that the action can cause production
  execution.
- Release readiness and production authorization must be complete **before
  the triggering action**.
- High-risk or destructive changes must not rely on authorization obtained
  after the trigger has already become executable.

If production authorization has not been obtained, prevent automatic
production execution by one of:

- Gating the production environment
- Excluding or disabling the production deployment
- Separating build from deploy
- Requiring an approved manual or environment gate
- Another mechanism consistent with the approved deployment architecture

Merge, a tag, or release creation may **trigger** production execution under
an explicitly approved deployment process. The required production
**authorization** must already be recorded before that trigger; the trigger
itself never supplies it. Neither merge, tags, releases, CI, automation, nor
credentials independently grant production authority, and automation never
creates its own authority.

---

# 9. Manual Deployment

For manual production deployment:

- Identify the authorized operator or process, where required.
- Identify the approved release candidate.
- Confirm readiness and authorization.
- Execute only the approved scope.
- Record the result and evidence.

Having credentials does not authorize an operator or AI agent to deploy. A
request to prepare deployment instructions is not approval to execute them.

---

# 10. Deployment Plan

For non-trivial or risky releases, define where applicable:

- Target environment and release candidate
- Deployment sequence and service ordering
- Migration, configuration, and feature-flag ordering
- Compatibility window
- Health checks and verification steps
- Expected duration or window
- Operator or automation
- Stop conditions and recovery trigger

The plan is proportional to risk. A trivial release does not need a
heavyweight plan.

---

# 11. Database and Migration Safety

Follow [standards/database.md](../standards/database.md). For releases with
migrations, determine where applicable:

- Whether migrations run automatically, and what triggers them
- Deployment and migration ordering
- Compatibility with old and new application versions
- Locking, runtime, and data volume
- Backfill behavior and restartability
- Destructive effects and recovery
- Replica and read behavior
- Verification

A migration file in the repository is not execution authority. Where merge,
deployment, or application startup applies migrations automatically, that
action can execute in production, so the required authorization must exist
before it ([Automatic Migration Execution](../standards/database.md#automatic-migration-execution)).

A down migration does not make lost data recoverable
([Rollback vs Recovery](../standards/database.md#22-rollback-vs-recovery)).

---

# 12. Configuration and Secrets

Verify where applicable:

- Required configuration exists, with correct environment-specific values.
- Secrets are available through approved mechanisms.
- No secrets are in source, artifacts, or client bundles.
- Configuration compatibility is understood.
- Recovery configuration is available.

Never print secrets into release evidence or logs. Never copy production
secrets into lower environments to make validation easier.

---

# 13. Feature Flags

Where feature flags are used, distinguish:

| Concept    | Meaning              |
| ---------- | -------------------- |
| Deployment | The code is present  |
| Activation | The behavior is on   |

A deployment behind a disabled flag reduces activation risk; it does not make
the deployment risk-free. Flag changes that materially alter production
behavior are production changes and need applicable authority.

Consider the default state, tenant or user targeting, stale clients,
deactivation, cleanup, and observability. Feature flags are not required for
every release.

---

# 14. Compatibility

Components do not necessarily deploy atomically. Consider where applicable:

- Old frontend with new backend, and the reverse
- Long-lived mobile clients and other API consumers
- Database and schema compatibility during rolling deployment
- Queue and event consumers
- External integrations
- Configuration compatibility

Breaking compatibility requires explicit handling under project requirements
and approved architecture.

---

# 15. Security Check

Follow [standards/security.md](../standards/security.md). Consider where
applicable: authentication and authorization changes, tenant isolation,
sensitive data, secrets, dependencies, exposed endpoints, trust boundaries,
file handling, session or token changes, security-sensitive configuration,
and audit or logging.

Known unresolved security risk is never hidden in release notes or ignored to
meet a release date. Risk acceptance uses the
[security exception process](../standards/security.md#25-security-exceptions);
this workflow creates no other.

---

# 16. Dependencies and Supply Chain

Verify where relevant: dependency lock state, expected versions, known
applicable security issues, build inputs, external package availability, and
any artifact provenance or integrity mechanisms the project requires.

No SBOM, signing, or provenance platform is mandated. Where the project
requires them, preserve those requirements.

---

# 17. Infrastructure Changes

Infrastructure changes are production changes. Assess where applicable: scope,
ordering, access and network changes, capacity, availability, security,
stateful resources, recovery, and observability.

A declarative infrastructure-as-code change is not harmless because it is
declarative. If merge or apply automation can change production
infrastructure, production authorization must precede that trigger.

---

# 18. Data Remediation

Data remediation follows [standards/database.md](../standards/database.md).
Separate where appropriate:

- Application deployment
- Schema migration
- Data migration
- Historical remediation

Each may have different risk, authorization, execution, and recovery. Deploying
the application does not authorize data correction.

---

# 19. Validation Before Deployment

Run applicable validation against the release candidate, such as unit,
integration, and end-to-end tests; build and type checks; linting and static
analysis; security checks; migration and compatibility tests; accessibility
checks; smoke tests; staging validation; and manual verification.

Record what actually ran, and mark each check **passed**, **failed**, **not
run** (with reason), or **not applicable** (with reason). Never claim a check
passed if it did not run. Never hide relevant failures.

A staging pass is evidence, not production authorization.

---

# 20. Release Evidence

Evidence must be enough to reconstruct why the release was considered safe and
was authorized. Record where applicable: release and candidate identifiers,
scope, validation, review status, approvals, known risks, deployment and
recovery plans, deployment result, and verification result.

Evidence may live in the repository, CI/CD, ticketing, release, or deployment
systems the project approves. No tool or format is mandated. Chat history
alone is not the release record.

---

# 21. Recovery Planning

Before a risky production change, decide how the system recovers if it fails.
Strategies include: rollback, roll forward, disabling a feature, restoring
configuration, restoring data, failover, isolating a component, stopping the
deployment, and manual remediation.

Rollback is not always possible. Database and data changes may make
application rollback unsafe. Recovery plans account for actual data and
compatibility semantics.

---

# 22. Rollback vs Roll Forward

Choose based on actual risk and system state. Rollback may be unsafe when:

- The schema is incompatible with the old code
- New data cannot be read by old code
- A destructive migration has run
- External side effects have occurred or messages were emitted
- Clients already depend on the new behavior

Roll forward may be safer where rollback would increase risk. No single
strategy is mandated.

---

# 23. Backup and Restore Assumptions

Backups existing does not prove recoverability. Where restore is part of the
plan, consider backup scope, freshness, retention, restore procedure, restore
validation, expected recovery time, and the data-loss window.

Never claim a restore is verified unless it was actually tested under the
project's requirements
([Backup and Restore](../standards/database.md#23-backup-and-restore)).

---

# 24. Deployment Execution

During deployment:

- Deploy only the authorized candidate and scope.
- Follow the approved sequence and observe stop conditions.
- Record failures.
- Do not improvise material production changes.

If execution requires a material deviation from the authorized plan:

STOP where safe → report → obtain the applicable decision and authorization →
continue only through the approved path

Urgency does not create implicit production authority.

---

# 25. Stop Conditions

For risky releases, define stop criteria, such as:

- Health checks fail
- Error rates rise materially
- Data validation or a migration fails
- Authorization or tenant behavior fails
- A dependency is unavailable
- An unexpected destructive effect occurs
- The deployment state becomes unknown

Never continue because a partial deployment has already happened. Stopping may
lead into recovery rather than immediate rollback.

---

# 26. Deployment Verification

After deployment, verify technical health where applicable:

- The expected version or artifact is running
- Service health and readiness
- Migration status and configuration
- Dependency connectivity
- Error rates and logs
- Queue and background processing
- Infrastructure state

A deployment command returning success is not proof of a successful
deployment.

---

# 27. Release Verification

Verify important user and business behavior where risk warrants it, such as
critical flows, authentication, authorization, tenant isolation, reads and
writes, integrations, background jobs, and client compatibility.

Use safe verification appropriate to production. Never perform unsafe
production actions (such as destructive test transactions or cross-tenant
probes on real data) merely to verify.

---

# 28. Observation Period

Some releases need a defined observation period. Consider error rates,
latency, saturation, queue backlog, business failures, support signals,
security signals, and data-integrity indicators.

No observation window is mandated for every release. The absence of alerts
does not prove stability when required business verification has not
occurred.

---

# 29. Deployment Failure

If deployment fails:

1. Stop further unsafe progression.
2. Determine the actual production state.
3. Contain the impact.
4. Choose the approved recovery strategy.
5. Execute the authorized recovery.
6. Verify the recovery.
7. Record evidence.
8. Create follow-up bug or incident work.

Never rerun a failed deployment without understanding whether it is safe and
idempotent. A partially executed migration or deployment cannot be assumed
retryable.

---

# 30. Partial Deployment

A failed release may leave production in a mixed state, for example:

- Some instances run old code and others new
- The application deployed but the migration failed, or the reverse
- One service deployed and another failed
- Configuration or infrastructure was partially applied

Determine the actual state before choosing recovery. Never report a release as
simply "failed" without stating whether production is in a mixed state.
Compatibility planning accounts for mixed-version operation wherever the
deployment architecture can produce it.

---

# 31. Emergency Release and Hotfix

Emergency releases compress time; they do not create authority. Where urgency
requires a deviation:

- Minimize scope.
- Preserve required Human authority and security constraints.
- Validate against the approved emergency task's validation plan.
- Preserve independent review where required.
- Complete the recorded follow-up after stabilization.

## Deferring Eligible Validation

Where risk-based governance permits it, an explicitly authorized emergency
release may **defer** an eligible, non-critical validation or check. A
deferral:

- Is explicitly authorized by the Tech Lead **before the production
  trigger**, under the [feature workflow](feature.md#exceptions) and, where
  security is affected, the
  [security exception](../standards/security.md#25-security-exceptions)
  rules.
- Is recorded with its reason, risk, and owner.
- Remains visible as **deferred / not run**, never as passed.
- Creates follow-up work where required.
- Leaves the test or check itself unchanged.
- Does not override requirements, security, or task governance, and does not
  satisfy a task's DONE criteria; deferred checks that those criteria require
  must still run before DONE.

## Never Permitted, Even in an Emergency

Emergency authorization never permits:

- Weakening a failing test, or loosening its assertions, to obtain a pass
- Deleting a failing test to make validation pass
- Changing expected results to match incorrect implementation
- Hiding failed test results, or reporting a failed or not-run test as passed
- Weakening security controls to release
- Bypassing security, authorization, or Human approval
- Bypassing required task completion or governance gates by relabeling them
  as emergency exceptions
- Making unrelated changes

This is the same test-integrity rule as
[Failed Validation Before Deployment](#32-failed-validation-before-deployment).
A test that is actually invalid, because the approved requirement changed or
the test itself is defective, is corrected through normal requirement, task,
and review governance. That is a legitimate correction, not prohibited
weakening.

Urgent bugfixes follow
[workflows/bugfix.md](bugfix.md#30-hotfix--urgent-bugs).

---

# 32. Failed Validation Before Deployment

If required release validation fails, do not reinterpret the result as
acceptable. Determine whether:

- The implementation must be corrected
- The test or check is invalid
- A requirement changed
- Risk acceptance is required under existing governance

Never weaken tests or delete checks to meet a release date.

---

# 33. Release Cancellation

A release may be cancelled before production execution. Record enough to
prevent later accidental deployment of the cancelled candidate. If a candidate
is superseded, make sure automation cannot deploy the obsolete one.

Cancelling a release creates no task status. Affected tasks follow the task
standard.

---

# 34. Release Completion

A release is complete when, as applicable:

- The authorized deployment finished and the expected candidate is running
- Required migrations and configuration completed
- Required verification passed and required observation completed
- Unresolved failures are handled through approved follow-up
- Release evidence is recorded
- Documentation and release notes are updated

A release is **not** complete merely because a PR merged, a tag exists, an
artifact was built, a deployment started, or a deployment command succeeded.

Release completion does not mark any task DONE. The Tech Lead records DONE for
each task under the task standard when its own criteria are met.

---

# 35. Release Notes

Where the project requires release notes, communicate relevant user-visible
changes, operational changes, compatibility changes, migration or
configuration requirements, and known issues.

Never include secrets, sensitive exploit details, or unnecessary internal
data. Do not invent release-note requirements the project does not have.

---

# 36. AI Agent Rules

AI agents may:

- Prepare release plans and commands
- Inspect release scope and analyze validation evidence
- Identify risks and review release readiness
- Perform explicitly authorized automated actions within the granted scope

AI agents never infer authority to merge, deploy, modify production, run
production migrations, modify production data, change production
infrastructure, rotate production secrets, or activate high-impact production
flags from tool access, credentials, autonomy, task assignment, review PASS,
CI success, or repository permissions.

If explicit authorization is absent or ambiguous: **STOP and report.**

---

# 37. Release Checklist

Apply in proportion to risk. Mark a row `N/A` with a reason when it does not
apply. Not every release has migrations, infrastructure changes, feature flags,
data remediation, mobile clients, a long observation window, manual
deployment, or automated deployment.

| Area | Check |
| --- | --- |
| Scope | Included tasks, PRs, components, and change types recorded; nothing unrelated |
| Candidate | Specific, identifiable artifact set; validated candidate is what is released |
| Review and approvals | Reviews complete, no unresolved BLOCKER/HIGH; required Human approvals recorded |
| Validation | Run against the candidate; passed, failed, not run, and not applicable reported |
| Security | Security standard applied; no hidden risk; exceptions through existing process |
| Database / migrations | Ordering, compatibility, locking, destructive effects, recovery assessed |
| Configuration / secrets | Present and correct; no secrets in source, artifacts, or evidence |
| Feature flags | Deployment vs activation distinguished; activation authorized |
| Compatibility | Mixed versions, older clients, consumers, and integrations considered |
| Dependencies | Lock state and known issues checked; required provenance preserved |
| Infrastructure | Treated as production change; authorization before apply trigger |
| Data remediation | Separated from deployment; separately authorized |
| Deployment plan | Sequence, ordering, operator or automation, stop conditions defined |
| Recovery plan | Feasible strategy chosen; rollback safety and data semantics assessed |
| Backups / restore | Restore tested where relied on; no assumed recoverability |
| Production authorization | Explicit, auditable, recorded before execution |
| Automatic triggers | Trigger identified and disclosed; authorized before it can execute, or gated |
| Execution | Only the authorized scope deployed; deviations stopped and re-authorized |
| Deployment verification | Expected version running; health, migrations, configuration confirmed |
| Release verification | Critical behavior verified safely |
| Observation | Defined where required; not based on absence of alerts alone |
| Failure / recovery | Actual state determined, including mixed states; recovery verified and recorded |
| Release notes | Where required; no secrets or sensitive details |
| Completion | Evidence recorded; tasks left to the task standard for DONE |
