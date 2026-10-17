# Project

<!--
Reusable project context, not a filled project specification. Replace placeholders
with confirmed facts and source references. Keep only useful detail; use
"N/A — <reason>" for inapplicable sections and "TBD / Not specified" for unknowns.
Optional fields may be omitted. Unknown does not mean inapplicable.
When copying this template, adjust relative links to the chosen document location.
-->

[AGENTS.md](../AGENTS.md) is authoritative; the Human Tech Lead retains final
authority. If sources conflict, stop and report them to the Tech Lead rather
than resolving the conflict yourself. This document gives project context; it
does not approve architecture, change task gates, or grant production authority.

| Artifact | Purpose |
| --- | --- |
| PROJECT | What we build, for whom, and under which project constraints |
| ARCHITECTURE | Current approved structure and operation |
| ADR | Why a material architecture decision was made, with approval evidence |
| TASK | A bounded piece of implementation/review work and its acceptance criteria |

## 1. Project Identity

| Field | Confirmed value / reference |
| --- | --- |
| Project name | <Name> |
| Short description | <One or two sentences> |
| Repository / repositories | <Locations, where applicable> |
| Product / service type | <Project-supplied description> |
| Project owner | <Existing accountable person/role, if defined> |
| Technical owner / Human Tech Lead | <Name or reference> |
| Project lifecycle / stage | <Project-supplied stage, or Not specified; not a task status> |

## 2. Purpose

<Problem being solved, intended outcome, and why the project exists. Keep this
at project level; feature acceptance criteria belong in specifications/tasks.>

## 3. Users / Actors

<List only known human or system actors and their goals/relationship to the
project. Reference authoritative actor definitions where available; do not invent
users, administrators, operators, services, or external systems.>

## 4. Scope

### In Scope

- <Project-level boundary / source>

### Out of Scope

- <Explicit exclusion / source>

<!-- Project boundaries complement, rather than replace, each task's scope. -->

## 5. Core Capabilities

- <High-level capability / relevant specification reference>

<!-- Summarize the product; an exhaustive feature catalogue is not required. -->

## 6. Business / Domain Context

<Stable terminology, major domain concepts, and confirmed business rules that
matter across tasks, with source references. Link detailed requirements instead
of duplicating every task requirement. Put unconfirmed behavior in Section 18.>

## 7. Constraints

| Applicable constraint | Confirmed limitation / authoritative source |
| --- | --- |
| <Business, regulatory, approved technology, hosting, data, security, operations, budget/time, or integration constraint> | <Actual constraint and reference, or Not specified> |

<!-- Do not invent constraints or introduce technology choices here. Material
architecture choices reference applicable APPROVED ADRs and their evidence. -->

## 8. Non-Functional Requirements

| Applicable quality | Project-supplied requirement / target | Source |
| --- | --- | --- |
| <Security, availability, performance, scalability, accessibility, compatibility, privacy, auditability, observability, recoverability, or maintainability> | <Confirmed requirement, or TBD / Not specified> | <Reference> |

<!-- Include relevant qualities only. Do not invent numeric targets, compliance
levels, SLAs/SLOs, or recovery objectives to complete the table. -->

## 9. Security / Data Classification Context

<Project-approved sensitive-data categories, authentication/authorization context,
privacy/regulatory requirements, and major trust boundaries. Link detailed
security requirements and the current architecture. No secrets or credentials.>

Follow the [Security standard](../standards/security.md); this summary does not
replace it or establish a new security policy.

## 10. Multi-Tenancy

<Multi-tenant: confirmed yes/no, or TBD. If applicable: tenant/workspace meaning,
high-level isolation expectations, and intentionally supported cross-tenant
operations with requirement references. Otherwise N/A — reason.>

Detailed trusted context and enforcement belong in architecture/security/data
design. Do not invent an isolation mechanism from the product label.

## 11. External Systems / Integrations

| Known system | Project-level dependency / purpose | Reference |
| --- | --- | --- |
| <Actual integration, if any> | <What the project needs from it> | <Contract / documentation> |

<!-- Do not assume identity, payment, messaging, email, storage, or other vendors.
Technical flows and material choices belong in ARCHITECTURE/ADRs. -->

## 12. Environments

| Known environment | Purpose / constraints | Documentation |
| --- | --- | --- |
| <Project-defined environment> | <Use and relevant data/access constraints> | <Reference; no credentials> |

<!-- Do not require local/dev/test/staging/production as a universal model. -->

## 13. Delivery / Release Context

<Confirmed deployment model (manual/automated where applicable), release cadence
only if supplied, and whether merge, tag, release creation, or another event can
trigger deployment. Link the architecture's actual trigger/gate model and the
project's production authorization mechanism/reference. Unknowns remain TBD.>

Follow the [Release workflow](../workflows/release.md). Recording automation or
a trigger here does not authorize its use. Required production authorization
must precede its production execution trigger; review PASS, CI, repository
access, credentials, and merge permission do not supply that authority.

## 14. Compatibility Context

<Project-supplied supported clients, browsers/devices, API consumers, mobile
version considerations, and backward compatibility expectations with references.
Do not invent a support matrix. Use Not specified for missing requirements.>

## 15. Project Conventions

| Applicable convention / command | Confirmed value | Repository reference |
| --- | --- | --- |
| <Approved stack, directory structure, naming, build/test/local development command, or documentation location> | <Actual convention or command; no secrets> | <Source> |

Conventions specialize Engineering OS without silently weakening governance.
Technology facts link applicable ADRs where a material decision was required.
Listed validation commands are expected instructions, not evidence they ran or
passed. Actual results belong in task/handoff records under the
[task standard](../standards/task-specification.md#validation).

## 16. Source of Truth / Key References

| Context | Authoritative repository path / durable reference |
| --- | --- |
| Requirements | <Location> |
| Current architecture | <Location of the filled architecture document> |
| Applicable ADRs | <Location / index; decisions follow the ADR standard> |
| Tasks / tracking | <Repository task location; related tracker references if used> |
| API contracts / design guidance | <Locations, including design system if applicable> |
| Operations / external documentation | <Runbooks and authoritative external documents, where applicable> |

Important project state must be persisted or referenced in repository documents,
not only in chat. Apply the source-of-truth rules in AGENTS.md; this table creates
no new precedence order. Material architecture decisions follow the
[ADR standard](../standards/architecture-decisions.md), using
[the ADR template](ADR.md). This document is not a decision log and cannot
override an APPROVED ADR.

## 17. Known Risks / Limitations

- <Known project-level risk/limitation, evidence, impact, owner or reference>

<!-- Do not speculate merely to fill the section. Recording a risk is not its
acceptance or permission to bypass security, readiness, or production rules. -->

## 18. Open Questions

| Question / assumption awaiting confirmation | Impact / criticality | Who must answer / reference |
| --- | --- | --- |
| <Unknown, explicitly distinguished from confirmed requirements> | <Affected scope/tasks; critical or non-critical where known> | <Owner / evidence needed> |

AI agents must not fill TBDs or placeholders by invention. Material unknowns
that affect implementation follow
[task readiness and BLOCKED recovery](../standards/task-specification.md#blocked-recovery)
and ADR governance where relevant. A completed template is not task readiness
verification or approval.

## 19. Project Context Checklist

Use proportionally. Mark N/A with a reason or Not specified where legitimate;
critical missing information still blocks affected task readiness.

- [ ] Identity, purpose, actors, project scope, and capabilities are understandable.
- [ ] Stable domain context and confirmed constraints have source references.
- [ ] Applicable quality/security/data/tenant requirements are recorded without invented targets.
- [ ] Known integrations, environments, delivery triggers, and compatibility expectations are clear.
- [ ] Conventions and authoritative references let another human/AI start without hidden chat history.
- [ ] Architecture detail and decision rationale remain in ARCHITECTURE and ADRs respectively.
- [ ] Known risks, assumptions, and questions are explicit; no secrets or implied production authority.
