# Database Standard

This is the minimum database engineering baseline for Engineering OS projects.
It covers schemas, integrity, access, transactions, migrations, and operations
for the storage technologies a project actually uses.

This standard is database-neutral. It mandates no vendor, relational model,
SQL, ORM, migration framework, normalization strategy, soft delete, isolation
level, or distributed transaction mechanism. Terms such as row, document,
field, constraint, and index apply only where the chosen technology supports
them. Unsupported mechanisms require an explicit alternative where the
underlying correctness requirement still applies.

Project-specific standards may strengthen or specialize this baseline, but
must not silently weaken applicable governance, security, or data integrity.
Approved ADRs determine project-specific architecture; examples here do not
override them.

[AGENTS.md](../AGENTS.md) is authoritative. Security is owned by
[standards/security.md](security.md), backend behavior by
[standards/backend.md](backend.md), task lifecycle by
[standards/task-specification.md](task-specification.md), architecture
approval by [standards/architecture-decisions.md](architecture-decisions.md),
and review by [standards/code-review.md](code-review.md). Git operations
follow [standards/git.md](git.md). If instructions conflict, stop and report
the sources and impact to the Human Tech Lead; do not resolve them yourself.

---

# 1. Database Engineering Principles

- Data correctness before convenience, with explicit data ownership.
- Integrity enforced at the strongest appropriate boundary.
- Least privilege and tenant isolation where applicable.
- Safe, reviewable schema evolution and required backward compatibility.
- Predictable transaction behavior and explicit concurrency assumptions.
- Recoverability, observability, and production changes proportional to risk.

Application behavior must not rely solely on assumptions that storage can
more reliably enforce, especially with competing writers. Do not weaken
integrity, security, or tests merely to make migrations, CI, or deployment
succeed. Tool limitations do not waive these requirements.

---

# 2. Requirements Before Database Changes

Determine and record where applicable:

| Area | Required understanding |
| ---- | ---------------------- |
| Domain | Data owner, entity/resource identity, required vs optional data, uniqueness, relationships, cardinality |
| Lifecycle | Creation, state transitions, deletion, project-supplied retention rules |
| Security | Tenant scope, sensitive-data classification, authorized access |
| Access | Expected volume, query patterns, ordering, performance needs |
| Consistency | Transaction boundaries, concurrency, multiple writers, side effects |
| Evolution | Compatibility, migration/backfill, deployment order, availability constraints |
| Recovery | Rollback feasibility, recovery expectations, responsible operator |

Do not invent missing business or data rules. Separate confirmed requirements,
assumptions, and open questions in the repository-backed task. Material
unresolved rules block readiness under the task standard. Required architecture
and security decisions need Human Tech Lead approval before implementation.

---

# 3. Schema Design

Represent actual domain requirements. Define applicable stable identifiers,
required/nullability semantics, types, relationships, uniqueness, defaults,
lifecycle/status representation, timestamps, and tenant ownership.

Avoid ambiguous or overloaded fields, unnecessary duplication, undocumented
application assumptions, and stored derived values without a consistency and
reconciliation strategy. Define ownership of updates to duplicated data.
Internal storage models are not automatically public API or event contracts.

Do not force normalization or denormalization globally. Choose based on
correctness, access patterns, consistency, and project needs.

---

# 4. Data Types

Choose types for domain semantics, including range, precision, scale,
identifier behavior, binary/text representation, structured data, and future
compatibility. Use project-specific database types deliberately.

- Do not store numbers as strings merely for convenience when numeric
  operations and ordering are required.
- Do not use floating-point representations for exact financial or decimal
  values when rounding would violate correctness. Define units and rounding
  rules from requirements.
- Distinguish instants from local calendar dates. Do not turn a business date
  into a timestamp unless that matches the domain. State timezone semantics;
  never depend implicitly on the server's local timezone.

---

# 5. Nullability and Defaults

NULL, missing values, empty values, and unknown values have explicit domain
meaning where the store distinguishes them. Do not make fields optional
merely to simplify implementation or migrations.

Defaults must represent valid behavior, not conceal missing required data.
Application and database defaults must not silently contradict each other.
When adding a default, assess whether historical records should receive it;
a convenient default is not evidence of their historical value.

Temporary migration states may differ from the target schema when explicitly
designed, reviewed, and followed by validation and completion steps.

---

# 6. Constraints and Integrity

Use storage constraints where appropriate for invariants that must survive
concurrency and multiple writers: identity, uniqueness, reference integrity,
required fields, and valid states where supported.

Application validation complements rather than automatically replaces storage
integrity. Where enforcement is unavailable or intentionally omitted, document
how the invariant is enforced and tested across all relevant writers.

Assess existing data before adding constraints. Constraint creation on
populated or large datasets requires review of invalid historical data,
validation cost, locking, and concurrent writes. Do not silently remove
constraints to make deployment succeed.

---

# 7. Relationships and Referential Integrity

Define parent/child ownership, cardinality, optional relationships, deletion
behavior, cascading effects, and orphan prevention. References must preserve
tenant and ownership rules, not merely point to an existing identifier.

Do not use cascading deletion merely for convenience. High-impact cascades
require explicit review. Where physical foreign keys are intentionally not
used or unsupported, state the alternative integrity strategy where integrity
matters; foreign keys are not globally mandatory.

---

# 8. Naming

Use consistent project naming conventions and meaningful names. Avoid
unexplained abbreviations and misleading names. This standard imposes no
global casing or pluralization rule. Renames require compatibility and
migration-safety assessment, including consumers outside the application.

---

# 9. Indexes

Indexes support actual integrity and access requirements: uniqueness, common
filters, joins/lookups, tenant-scoped access, ordering, and important queries.
Where composite indexes exist, their order should reflect actual access
patterns and the chosen engine's behavior.

Do not index every field. Assess write, storage, and maintenance costs.
Review redundant or unused indexes using tooling and operational evidence
where available. Large index creation or rebuilds require production impact
review; an index operation is not automatically nonblocking or online.

---

# 10. Query Safety

Use parameterized queries or an equivalent safe mechanism for the chosen
storage API. Never concatenate untrusted input into interpreted expressions.
Dynamic identifiers, operators, and sort choices require safe construction
and allow-lists where they cannot be bound as values.

Bound retrieval where data may grow. Consider result limits, pagination,
selectivity, joins/lookups, query count, N+1 patterns, locking impact, and
memory/resource consumption. Use deliberate ordering when results depend on
order, including stable pagination where required.

Do not use `SELECT *` or equivalent broad retrieval by default when it creates
contract, performance, or sensitive-data exposure risk. Pagination and
aggregation must preserve authorization and tenant scope. Do not optimize
solely from intuition when measurement is available.

---

# 11. Query Plans and Performance

For performance-sensitive changes, inspect actual execution behavior where
tooling permits: plans, indexes used, rows/documents scanned, cardinality
estimates, sorts, locks, duration, and resource consumption.

Use representative volume and data distribution where practical. A query
fast on an empty development database is not evidence of scalability.
Some diagnostic execution tools run the query: understand their effects
before using them, especially against production or mutating operations.

Do not require plan analysis for every trivial query. Scale investigation
to expected volume, operational evidence, and risk.

---

# 12. Transactions

Define required consistency boundaries and the actual atomicity guarantees
of the selected store. Transactions are intentional, as short as practical,
and explicit about commit, rollback, and failure behavior. Do not assume
schema changes support transactional rollback just because data writes do.

Do not hold transactions open unnecessarily across slow external operations.
A database transaction does not automatically include external APIs, message
brokers, files/object storage, email, or unrelated databases.

Coordinate cross-boundary consistency through project-appropriate architecture,
such as a durable outbox, reconciliation, or compensating actions. Follow
[backend transaction guidance](backend.md#9-transactions-and-consistency).
Distributed transactions are not globally mandated; significant choices
follow ADR governance.

---

# 13. Isolation and Concurrency

State concurrency assumptions where competing operations affect correctness.
Consider lost updates, duplicate creation, uniqueness races, conflicting
state transitions, contention, and deadlocks; dirty, non-repeatable, phantom
reads and write skew apply where the technology and operation make them relevant.

Check-then-write application logic is not atomic on its own. Use suitable
constraints, atomic operations, optimistic concurrency, row/document locking,
isolation, or serialization. Do not mandate SERIALIZABLE or one isolation
level globally. A transaction alone does not prove every invariant is safe.

Handle duplicate/conflict outcomes deliberately, including races after a
pre-check. Use meaningful concurrent tests where feasible with the actual
technology when its semantics determine correctness.

---

# 14. Deadlocks and Locking

Understand locks for high-contention or multi-resource operations. Reduce
avoidable deadlocks with consistent access ordering where appropriate;
keep lock scope and duration deliberate.

Handle expected deadlock/serialization failures according to project and
engine behavior. Retry only if the complete operation is safe to repeat,
using bounded attempts and backoff/jitter where appropriate. Do not retry
only a fragment when the whole transaction must restart, repeat external
side effects unintentionally, or hide persistent contention with unlimited
retries. Report exhausted retries explicitly.

A connection loss or timeout may leave the commit outcome unknown. Do not
assume it rolled back; use idempotency or reconciliation before unsafe replay.

---

# 15. Multi-Tenant Data Isolation

Follow [standards/security.md](security.md#6-multi-tenant-isolation).
Where multi-tenancy exists:

- Tenant ownership is explicit and context comes from a trusted source.
- Reads, writes, updates, deletes, joins, and aggregations preserve tenant
  boundaries. Resource ID alone is insufficient when tenant ownership
  participates in authorization.
- Jobs and migrations preserve their intended tenant scope. Deliberate
  cross-tenant administration requires appropriate authorization and review.
- Caches, materialized views, and derived data must not leak across tenants.
- Uniqueness rules specify global versus tenant scope; relevant indexes
  support important tenant-scoped access patterns where appropriate.
- Cross-tenant negative tests are required where applicable.

Database-native tenant controls may provide defense in depth. They replace
application enforcement only when the approved architecture establishes a
trusted enforcement model; they do not waive authorization. Tenant-isolation
architecture follows ADR requirements where significant.

---

# 16. Sensitive Data

Follow [standards/security.md](security.md#10-sensitive-data). Identify
sensitive fields from project requirements, minimize collection and copies,
and expose only what the consumer is authorized to need.

Consider encryption, redaction/masking, access control, and supplied retention
requirements across primary storage, backups, replicas, exports, analytics,
logs, and development/test datasets. Do not invent compliance or retention
law. Do not casually copy production sensitive data into development or tests;
use synthetic or appropriately protected/de-identified data under approved rules.

---

# 17. Credentials and Database Access

Runtime application database credentials must not use database superuser
credentials. Database owner-level credentials are not the default runtime
identity and require explicit architectural/operational justification where
applicable. Least privilege remains required.

Migration and administrative identities may have different privileges under
approved architecture. Separate operational and migration privileges from
runtime privileges where the architecture supports it; privileged migration
execution does not justify privileged runtime access. Never ship database
credentials to untrusted clients.

Secret storage, exposure response, and logging follow
[standards/security.md](security.md#9-secrets-and-credentials). Repository
access or possession of credentials is not authority to use production.

---

# 18. Migrations

Schema and data migrations must be reviewable and reproducible under project
tooling and [Git governance](git.md). Preserve migration history; applied
migrations should not normally be edited in place when environments would
disagree. Prefer a new corrective migration unless tooling and governance
explicitly define another safe mechanism.

Assess existing data, deployment order, application compatibility, locking,
runtime, volume, backfills, indexes, constraints, failure, retries/restartability,
and actual rollback/recovery. Never infer safety from success on an empty
database. Understand whether partial application is possible and how progress
and failure are detected. Do not silently ignore migration errors.

Where supported and appropriate, consider lock/statement time limits or
equivalent safeguards for high-risk migration operations. Assess their failure
and recovery effects rather than assuming cancellation rolls back every change.

---

# 19. Expand / Migrate / Contract

Where simultaneous schema/application changes would break running or older
consumers, consider phased evolution:

1. Expand with compatible structures.
2. Deploy compatible application behavior.
3. Migrate or backfill existing data.
4. Verify correctness and compatibility.
5. Switch usage.
6. Contract or remove obsolete structures.

This is guidance, not a mandatory sequence for every change. Do not remove
old structures until all required consumers, including workers and stored
messages, have safely moved. Define behavior during mixed-version operation.

---

# 20. Data Backfills

Plan volume, batches, transaction size, locks, replication/operational impact,
restartability, idempotency where repeatable, progress visibility, partial
failure, and post-completion validation. A single massive transaction is not
assumed safe.

Account for concurrent application writes so a backfill does not overwrite
newer valid data or leave gaps. Record how completeness and correctness are
verified. Tenant/security-sensitive backfills need deeper review.

---

# 21. Destructive Changes

Treat operations as high risk where data loss or availability impact is
possible: DROP, destructive ALTER, truncation, bulk deletion, irreversible
transformations, incompatible type shrinking, and removal of fields still
used by consumers. These are examples, not SQL requirements or an exhaustive list.

Before destructive production work, determine affected data and consumers,
backup/recovery, migration dependencies, feasibility of rollback or recovery,
maintenance/downtime needs, and required approval. Verify the intended target
and scope. Merge is not inherently production execution authority, but may
trigger configured automation that executes the migration. Apply the
[automatic-execution safeguards](#automatic-migration-execution) before the
change can reach production, including before merge when merge is the trigger.
Production authorization remains governed by AGENTS.md and the project's
explicit deployment process.

---

# 22. Rollback vs Recovery

Document the actual strategy: safe reversal, forward fix, corrective migration,
or restoration from backup as appropriate. Do not promise rollback that is
not technically safe. A down migration in source does not prove production
rollback safety, particularly after new writes or destructive transformations.

Assess recovery's effect on data written since the change and on external
side effects. Destructive changes may need backup/restore or forward recovery
instead of automatic rollback. Recovery operations require their own applicable
production authorization and verification.

---

# 23. Backup and Restore

Where recovery is operationally relevant, project architecture defines backup
scope, retention, encryption, access control, restore procedure, and restore
testing. Use project-supplied recovery point and recovery time objectives
(RPO/RTO); do not invent values. Missing material recovery requirements are
questions for the Tech Lead.

A backup not successfully restore-tested is not proven recovery. Restore
verification should demonstrate usable, consistent data where required, not
merely that a backup file exists. Protect backups containing sensitive data
with equivalent safeguards. Test restores without overwriting production
or exposing data to an unauthorized environment.

---

# 24. Replication and Read Replicas

Where replicas exist, understand lag and consistency; do not assume immediate
read-after-write consistency. Route consistency-sensitive reads appropriately,
monitor replication health where operationally required, and assess failover
behavior. Do not let stale reads make unsafe authorization or integrity decisions.

A replica is not an independent backup by default: destructive or corrupt
writes may propagate. Failover architecture and guarantees are project-specific.

---

# 25. Soft Delete / Hard Delete

Deletion follows domain requirements. Soft delete is neither mandatory nor
automatically safer. If used, address uniqueness, filtering, authorization,
restoration, retention, relationships/cascades, indexes, and accidental exposure.
Do not rely on everyone remembering a filter when stronger enforcement is
required by architecture.

Hard deletion must respect applicable retention and recovery requirements.
Account for copies where requirements demand it; marking a record deleted is
not proof of erasure from backups, replicas, exports, or analytics.

---

# 26. Audit Data

Where auditability is required, define actor, action, resource, timestamp,
and appropriate before/after or change context. Audit data must never expose
secrets; minimize sensitive values and follow
[security audit rules](security.md#17-logging-and-audit).

Application users must not silently rewrite security-sensitive audit history
unless explicitly permitted by approved architecture. Audit/history tables
are not automatically operational backups.

---

# 27. Seed and Reference Data

Define ownership and purpose. Distinguish required system reference data,
development/test fixtures, demo data, and production initialization. Do not
ship development/demo credentials or unsafe fixtures to production.

Seeds expected to run repeatedly should be repeatable/idempotent. Changing
reference data that controls business behavior requires review of affected
consumers; seed execution is not a shortcut around production approval.

---

# 28. Test Data

Prefer deterministic tests and isolated datasets where shared mutable state
would affect correctness. Do not casually use real production sensitive data.

Test important behavior with the actual database technology when correctness
depends on constraints, transactions, isolation, locking, types, or query
semantics. Mocks and in-memory substitutes are not proof when their semantics
differ materially. Configure relevant versions and settings deliberately;
test success on one engine does not establish another's behavior.

---

# 29. Migration Testing

Scale tests to risk. Cover realistic prior schemas and existing rows, invalid
historical data, relevant large datasets, partial failure and restart,
application compatibility, backfill correctness, constraint creation, and
recovery strategy where applicable.

For realistic or representative migration data, prefer synthetic data,
generated representative datasets, or properly de-identified data. A
production-derived copy may be used only when explicitly approved and
protected according to the project's security and data-handling requirements;
it is not the default source of realistic data. Follow
[Sensitive Data](#16-sensitive-data) and [Test Data](#28-test-data), including
the security requirements they reference.

High-risk migrations must not be validated solely on a fresh empty database.
Record actual commands, environment, results, and unverified conditions.
Do not claim recovery was exercised when only a down script was inspected.

---

# 30. ORM and Query Builder Usage

These are tools, not substitutes for understanding storage behavior.
No ORM is globally required or prohibited. Inspect generated behavior where
correctness or performance matters, including N+1 queries, broad updates or
deletes, implicit transactions, unexpected cascades, eager-loading explosions,
missing tenant predicates, and unsafe raw-query escape hatches.

---

# 31. Observability

Operationally important behavior may require query latency, errors, connection
pool usage, contention, deadlocks, replication lag, migration progress,
storage growth, and slow-query visibility. Choose signals proportional to risk.

Avoid high-cardinality telemetry and sensitive-data leakage. Do not log raw
sensitive queries or parameters without explicit justification and appropriate
protection; secrets must never be logged. Release connections and related
resources on failure and cancellation as required by the backend standard.

---

# 32. Production Change Safety

Before high-risk production changes, define the change window, expected locks,
runtime, traffic impact, compatibility, backup/recovery, monitoring, abort
criteria, responsible operator, and required approval. Identify what stopping
safely means when an operation is only partially complete.

AI agents must not infer production authorization from repository access,
tool capability, task assignment, credentials, migration existence, review
PASS, CI success, or merge permission. None independently grants production
database execution authority. AGENTS.md and explicit project deployment/release
governance control production changes; releases remain under human control.
Preparing and testing a migration is distinct from executing it in production.

## Automatic Migration Execution

Determine whether merge, deployment, application startup, or a release/deployment
pipeline automatically applies pending migrations. The PR/change record for
every production-affecting migration must clearly state whether merging or
deployment (including startup of the deployed application) can trigger database
execution, and identify the trigger or gate. Reviewers must be able to assess
this before merge; if the execution path is unknown, resolve it before enabling
possible automatic execution.

For destructive or otherwise high-risk migrations that can be applied
automatically, obtain and record all required production/destructive-change
authorization **before the point that makes automatic production execution
possible**. That point may be merge, deployment, startup, or enabling a pipeline
stage; do not wait for a later manual step that the architecture does not have.

If authorization is not in place, prevent automatic production execution by
gating, excluding, disabling, separating the migration, or an equivalent control
under the project's approved deployment architecture. This means controlling
the migration execution path, not disabling integrity or security safeguards.
If no safe gate exists, do not proceed past the triggering point; report the
blocker and follow existing task governance.

Explicitly defined and authorized automated deployment/migration architectures
remain permitted under AGENTS.md. No universal execution or gating mechanism is
mandated here. Merge may trigger that authorized automation, but merge permission
alone is not approval of the production database change.

---

# 33. Testing Expectations

Test applicable constraints, uniqueness, relationships, tenant isolation,
transactions, concurrency, query behavior, migrations, backfills, deletion,
and sensitive-data exposure. Use negative tests where correctness/security
depends on rejected behavior, including cross-tenant access where applicable.

Never claim tests passed unless executed. Report expected checks separately
from actual commands and results, with failures and gaps. Never bypass,
weaken, skip, disable, or delete tests merely to make CI pass. Legitimate
corrections of invalid tests follow existing agent and security rules.

---

# 34. Review Expectations

Follow [standards/code-review.md](code-review.md) and its
[severity model](code-review.md#severity-levels); do not create another one.
Review applicable domain/data correctness, schema semantics, constraints,
tenant isolation, sensitive data, queries, indexes/performance, transactions,
concurrency, locks, compatibility, migrations, backfills, destructive changes,
recovery, and tests. High-risk production/destructive changes require deeper
review; routine changes do not need every category.

Independent review is required under existing governance. PASS grants no
human approval, merge, production, or DONE authority.

---

# 35. Governance and Scope Drift

Missing data rules, unresolved deletion/retention, missing tenant ownership,
unclear migration compatibility, missing security decisions, unresolved
architecture, or materially expanded scope must not be answered by invention.
Stop affected work and report to the Tech Lead.

Ordinary defects use normal correction/re-review; readiness-invalidating
issues use [task-standard BLOCKED recovery](task-specification.md#blocked-recovery).
Who may record a blocker, request changes, verify readiness, or mark DONE
is governed only by that standard. Recording BLOCKED grants no authority
to resolve governance questions or resume implementation.

Significant architecture decisions follow the
[ADR standard](architecture-decisions.md). AI agents may propose, never
approve. Required decisions must be APPROVED with repository evidence;
PROPOSED, rejected, missing, or obsolete required decisions prevent readiness
until the applicable approval and recovery requirements are met.

For security issues discovered at READY FOR HUMAN APPROVAL, follow
[the security standard](security.md#issues-found-during-ready-for-human-approval):
stop advancement, report, and do not silently modify/push approval-ready
content. Preserve independent review and required revalidation.

---

# 36. Database Checklist

Apply before implementation and during review. Mark `N/A` with a reason
where genuinely inapplicable. Not every project uses relational storage,
migrations, replicas, ORMs, multi-tenancy, or soft delete.

| Area | Check where applicable |
| ---- | ---------------------- |
| Ownership and identity | Data owner, identifiers, lifecycle, tenant ownership explicit |
| Schema and types | Domain semantics, precision, dates/instants, structured values appropriate |
| Nullability/defaults | Absence meaningful; defaults valid for new and historical data |
| Integrity/relationships | Constraints or explicit alternatives; cascades and orphan prevention reviewed |
| Indexes/performance | Access and uniqueness supported; costs and representative execution assessed |
| Queries | Safe construction, bounded results, deliberate ordering, scoped access |
| Transactions | Atomic boundary and failure behavior explicit; external effects coordinated separately |
| Concurrency/locking | Races, isolation, locks, deadlocks and bounded whole-operation retries assessed |
| Tenant isolation | Trusted context across reads/writes/joins/jobs/derived data; negative tests |
| Sensitive data | Minimized; copies, retention, access, encryption, and redaction considered |
| Credentials | No runtime superuser; owner access justified; least privilege; separate migration access; no secrets in clients/logs |
| Migrations | Reproducible history; realistic data, partial failure, deployment order assessed |
| Compatibility | Required consumers remain supported; expand/migrate/contract where useful |
| Backfills | Bounded, restartable, observable; concurrent writes and correctness verified |
| Destructive changes | Data/consumers identified; recovery and production authorization established |
| Rollback/recovery | Actual feasible strategy; new writes and external effects considered |
| Backups | Scope/protection/restore tests; project-supplied objectives; no assumed recovery |
| Replicas | Lag, consistency-sensitive reads, failover understood; not assumed backup |
| Deletion | Domain semantics, filtering, restoration, uniqueness, retention and copies addressed |
| Audit | Required context, protected history, no secrets; not mistaken for backup |
| Seeds/test data | Purpose, isolation and repeatability clear; no unsafe production copies/fixtures |
| Migration tests | Safe representative data; production-derived copies approved/protected; prior schema and failure/recovery exercised |
| ORM/query behavior | Generated queries, cascades, tenant filters and transaction behavior understood |
| Observability | Useful signals; no secrets, unnecessary sensitive data, or uncontrolled cardinality |
| Production safety | Automatic triggers disclosed; authorization before enabling execution or safe gate; impact, operator and recovery defined |
| Tests/security | Real storage semantics and negative paths tested; actual evidence; security baseline preserved |
| Governance | No invented rules; applicable ADRs approved; lifecycle and review delegated |
