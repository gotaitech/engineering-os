# Architecture

<!--
Reusable CURRENT APPROVED architecture context. Replace placeholders only with
confirmed, approved facts and references. Keep entries brief; link detailed
contracts, runbooks, and standards. Optional fields may be omitted; use
"N/A — <reason>" where inapplicable and "TBD / Not specified" for unknowns.
A placeholder is not an approved design. Adjust links if copying elsewhere.
-->

## 1. Architecture Summary

<Brief overview of the current approved system: major parts and how they relate.
Do not choose an architecture style to fill the template.>

- Architecture baseline / last verified: <Date, repository revision or reference>
- Related project context: <Repository path to the filled PROJECT document>

<!-- Distinguish approved structure from actual deployed state where they differ.
Record confirmed implementation/deployment gaps without relabeling them approved;
report conflicts. Future proposals belong in ADRs/open questions, not this summary. -->

## 2. Architecture Authority

[AGENTS.md](../AGENTS.md) is authoritative. The Human Tech Lead retains final
architecture authority. This document describes current approved structure and
operation; it does not approve architecture or replace requirements/tasks.

Material decisions are governed by applicable APPROVED ADRs under the
[ADR standard](../standards/architecture-decisions.md). AI agents may propose,
never approve. Update this view when approved changes affect it; do not silently
rewrite APPROVED ADR history. If this document, implementation evidence, or a
diagram conflicts with an APPROVED ADR or another instruction, stop and report
the conflict to the Tech Lead for resolution through existing governance.
Neither PROJECT nor ARCHITECTURE overrides an ADR.

This is not an ADR, proposal backlog, decision history, or implementation task.
Keep decision rationale in ADRs and bounded execution/acceptance criteria in
[TASKs](TASK.md). Task readiness/statuses, review, and production authority remain
owned by their existing standards/workflows.

## 3. System Context

<Current system boundary, known human/system actors, external systems, and major
trust relationships. Link actor definitions from PROJECT rather than copying
business requirements. Optional diagram/reference in any project-chosen format.>

## 4. Major Components

| Existing component | Responsibility / owned behavior or data | Interfaces / dependencies | Runtime context / reference |
| --- | --- | --- | --- |
| <Name> | <Ownership> | <Connections> | <Where it runs / detailed documentation> |

<!-- Include only components that exist in the approved design. Frontends, mobile
apps, APIs, workers, schedulers, stores, caches, queues, and storage are possibilities,
not requirements. -->

## 5. Component Boundaries / Dependency Direction

<Important allowed/prohibited dependencies, ownership boundaries, shared
modules/libraries, and cross-component communication, with applicable references.
Document current rules; no architecture pattern or folder layout is mandated.>

## 6. Data Architecture

<Where applicable: current stores, major domains and ownership, consistency,
transaction boundaries, replicas/read models, retention/deletion rules, and
backup/recovery references. Identify boundaries with external side effects;
a database transaction is not assumed to include them. Otherwise N/A — reason.>

Follow the [Database standard](../standards/database.md) and
[Backend standard](../standards/backend.md). Link detailed schemas and data flows
rather than duplicating those standards.

## 7. Multi-Tenant Architecture

<If applicable: source of trusted tenant context, actual isolation/enforcement
boundaries, scoped components/data, authorized cross-tenant operations, and
context propagation through caches, jobs, and integrations. Otherwise N/A.>

Client-side tenant filtering is not isolation. Follow the
[Security](../standards/security.md#6-multi-tenant-isolation) and
[Database](../standards/database.md#15-multi-tenant-data-isolation) standards;
reference the governing decisions and detailed enforcement design.

## 8. Authentication / Authorization Architecture

<If applicable: authentication authority/flow, session/token boundary,
authorization enforcement at trusted boundaries, service identity, and privileged
or admin paths. Distinguish identity from permission. Link approved design and
security documentation; never include credentials/tokens/secret values.>

Follow the [Security standard](../standards/security.md). UI controls and
client-provided roles/tenant IDs are not authorization proof.

## 9. API / Interface Architecture

| Existing interface | Owner / consumers | Approved protocol/style and contract location | Compatibility / versioning expectations |
| --- | --- | --- | --- |
| <Interface> | <Ownership> | <Reference> | <Project-supplied requirements or Not specified> |

<!-- No REST, GraphQL, RPC, event model, or version scheme is mandated. -->

## 10. Integration Architecture

<Known external flows: direction, trust boundary, authentication mechanism
category (not credentials), retry/idempotency expectations, failure handling,
and reconciliation where applicable. Link contracts/runbooks and approved ADRs.
Do not invent provider behavior or verification mechanisms.>

## 11. Async / Messaging Architecture

<If applicable: producers/consumers, queues/topics/events, actual delivery and
ordering assumptions, idempotency, retry/dead-letter behavior, and failure/recovery
ownership. Otherwise N/A. Do not assume exactly-once guarantees or require messaging.>

## 12. Frontend / Client Architecture

<If applicable: client applications, routing/navigation, state ownership,
backend integration boundary, platform considerations, and existing design-system/
component source. Otherwise N/A. No framework or state library is mandated.>

Follow the [Frontend standard](../standards/frontend.md); link detailed client
design rather than duplicating the standard or PROJECT's support matrix.

## 13. Runtime / Deployment Architecture

<Current topology: runtime components, process/container/server/function model
as actually used, network boundaries, gateway/load balancing, environments,
scaling, and stateful/stateless distinctions where applicable. Link deployment
configuration and diagrams. No cloud, container, orchestration, VM, serverless,
or on-premises model is required.>

## 14. Release / Deployment Trigger Model

Authorization is permission for a production change. A trigger/execution mechanism
performs an already-authorized change; documenting it does not grant authority.

| Change / target environment | Actual trigger and execution path | Gate / authorized operator or automation | Production authorization mechanism / evidence reference |
| --- | --- | --- | --- |
| <Applicable application, migration, configuration, flag, or infrastructure change> | <Actual manual action or event, with configuration reference> | <Existing controls> | <Approved process and applicable authorization record, or TBD> |

<!-- Identify actual manual deployments, merge/tag/release triggers, environment
approval gates, app-startup migrations, and automated infrastructure apply where
they exist. Explicitly record which events can cause production execution. -->

Follow the [Release workflow](../workflows/release.md#8-automated-deployment) and
[automatic migration rules](../standards/database.md#automatic-migration-execution).
Required authorization must precede the production trigger, including merge,
tag/release creation, deployment/startup migrations, and IaC auto-apply. If absent
or ambiguous, stop/report and prevent automatic execution through the approved
gating architecture. Explicitly authorized automation remains allowed; it cannot
create its own authority. Credentials, access, CI, review PASS, or merge permission
alone authorize nothing. Use release records for candidate-specific execution
evidence rather than making this current architecture view a release log.

## 15. Infrastructure Architecture

<If applicable: current hosting/runtime platform, networking, DNS/load balancing,
storage, compute, managed services, and IaC location. Link the approved design and
operational documentation; no secrets. IaC repository access is not production
authority. Reference Section 14 for triggers instead of duplicating their rules.>

## 16. Configuration / Secrets Architecture

<Configuration categories, sources, environment-specific handling, ownership,
secret-management mechanism, rotation owner/process reference, and client-safe
versus trusted-server boundary. Never include secret values or credentials.>

## 17. Observability Architecture

<Existing logs, metrics, traces, audit events, alerts, health/readiness signals,
correlation identifiers, ownership, and references where applicable. No vendor
is mandated; diagnostics must not expose secrets/sensitive internals.>

## 18. Failure / Resilience Architecture

<Existing timeout/retry behavior, failure containment, graceful degradation,
failover, queue recovery, and dependency-failure behavior where relevant. Identify
owners/runbooks and known limits; do not invent mechanisms not present.>

## 19. Backup / Recovery Architecture

<Where applicable: backup scope, restore approach, dependencies, limitations,
runbook location, and project-supplied recovery objectives (or Not specified).
Record restore-verification evidence/reference and date, or Not verified.>

Backup existence is not proven recovery; never claim restore was verified without
actual test evidence. Do not invent RTO/RPO. Follow
[database recovery guidance](../standards/database.md#23-backup-and-restore);
rollback is not assumed safe after data changes or external side effects.

## 20. Security Architecture Summary

<Concise map/reference of current trust boundaries, sensitive-data flows,
privileged paths, external exposure, tenant boundaries, and enforcement locations.
Summarize or cross-reference Sections 7–8 and diagrams; avoid duplicated details.>

The [Security standard](../standards/security.md) remains authoritative for
security. This summary does not introduce a competing security standard or
approve security-sensitive decisions/accept risk.

## 21. Compatibility / Evolution

<Current compatibility constraints: rolling deployments, mixed versions,
long-lived mobile clients, API/event consumers, and schema evolution where
applicable. Link contracts and approved strategies. Do not invent future designs
or supported versions; unresolved changes belong in Section 25/ADRs.>

## 22. Architecture Diagrams

| Optional view | Diagram / repository reference | Scope / baseline |
| --- | --- | --- |
| <Useful system-context, component, deployment, data-flow, or trust-boundary view> | <Link> | <What it depicts and when verified> |

<!-- Diagrams and format are optional. Use the project's choice; no C4/UML/Mermaid
or other tool is required. A contradictory diagram cannot override written
approved architecture/ADR evidence; stop/report and resolve the conflict. -->

## 23. Approved ADR References

| ADR / repository path | Title | Status in record | Architecture area | Current relevance |
| --- | --- | --- | --- | --- |
| <Applicable ADR link> | <Title> | <APPROVED; verify record/evidence> | <Area> | <What it governs today> |

Link decisions instead of copying their rationale/history. A table entry cannot
approve an ADR. Use the currently applicable APPROVED replacement when an ADR is
SUPERSEDED. Historical references, if useful, must be explicitly labeled as
historical and point to the replacement; do not present them as current authority.
Required missing, PROPOSED, REJECTED, or obsolete decisions block dependent task
readiness under the task/ADR standards. If none are required, state N/A with reason.

## 24. Known Architecture Risks / Constraints

- <Known current limitation/risk — evidence, impact, owner/reference>

<!-- No speculative risk inventory is required. Recording a risk does not accept
it or authorize security/production exceptions. -->

## 25. Open Architecture Questions

| Unresolved question / gap | Affected area / tasks | Owner / decision or evidence needed |
| --- | --- | --- |
| <Unknown; no proposed design presented as approved> | <Impact / criticality> | <Reference or pending clarification> |

An unresolved question is not an approved decision. AI agents must surface
material unknowns rather than invent facts from placeholders/TBDs. Follow
[task readiness / BLOCKED recovery](../standards/task-specification.md#blocked-recovery)
and ADR governance before dependent implementation. No direct BLOCKED → IN PROGRESS
or self-approval is authorized by completing this document.

## 26. Architecture Validation / Maintenance

<Who maintains this view within existing project roles; last verification and
relevant implementation/approval references. No new organizational role required.>

Review/update affected sections after approved ADR, major component/data-store,
trust-boundary, topology, or integration changes. Reconcile the approved baseline
with actual implementation/deployment evidence; identify gaps explicitly. New
proposals stay outside current-state sections until approved. No documentation
churn is required for trivial implementation changes that do not affect architecture.
Updates follow normal [Git](../standards/git.md) and
[independent review](../standards/code-review.md) governance; updating this file
is not architecture approval. Material ADR changes use new ADRs and auditable
supersession under the ADR standard, not silent history edits.

## 27. Architecture Checklist

Use proportionally; N/A with a reason is valid where inapplicable. TBD is a known
gap, not evidence a gate passed. This checklist grants no approval or DONE status.

- [ ] Current approved summary, context, components, boundaries, and dependencies are clear.
- [ ] Applicable data/tenant design and authentication/authorization boundaries are referenced.
- [ ] APIs, integrations, async flows, and clients reflect actual approved design.
- [ ] Runtime, deployment triggers/gates, and infrastructure are explicit; authority remains separate.
- [ ] Configuration/secrets, observability, resilience, recovery evidence/limits, and security are covered where relevant.
- [ ] Compatibility constraints and optional diagrams agree with the written approved baseline.
- [ ] Applicable APPROVED ADRs are linked; obsolete decisions are not used as current authority.
- [ ] Known risks, deployment gaps, and open questions are explicit; no invented facts or secret values.
- [ ] Maintenance references support the current view without copying ADR history or implementation tasks.
