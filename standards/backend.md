# Backend Standard

This standard defines the minimum engineering baseline for backend services,
APIs, workers, background jobs, and other server-side components developed
under Engineering OS.

- Project architecture and conventions may strengthen or specialize this
  standard. They must not silently weaken Engineering OS governance or
  security requirements.
- It is not framework-, language-, or platform-specific. It does not mandate
  an architecture style (such as Clean Architecture, DDD, hexagonal, or
  microservices), an API style (such as REST or GraphQL), a language, ORM,
  database, queue, container platform, logging library, or observability
  platform.
- Approved ADRs and project architecture take precedence over the
  illustrative examples here.

[AGENTS.md](../AGENTS.md) is authoritative. Security requirements are owned by
[standards/security.md](security.md), task lifecycle by
[standards/task-specification.md](task-specification.md), architecture
authority by [standards/architecture-decisions.md](architecture-decisions.md),
and review by [standards/code-review.md](code-review.md). This standard does
not redefine them. If it conflicts with any of them, stop and report the
conflict to the Human Tech Lead. Do not resolve it yourself.

---

# 1. Principles

- **Correctness before cleverness.**
- **Explicit boundaries** between responsibilities.
- **Maintainable code** that the next engineer or agent can change safely.
- **Predictable behavior**, including under failure.
- **Secure by default** and **least privilege**.
- **Fail safely.**
- **Observable behavior** for operationally meaningful components.
- **Backward compatibility** where consumers require it.
- **Idempotency** where operations can repeat.
- **Testability** of important behavior.
- **Clear ownership of side effects.**

Backend code must not bypass requirements, architecture decisions, security
controls, or validation to make implementation or CI pass. Tool limitations do
not create engineering exceptions.

---

# 2. Requirements Before Implementation

Before implementation, determine where applicable:

| Area        | Determine                                                                 |
| ----------- | ------------------------------------------------------------------------- |
| Behavior    | Business behavior, actors, failure behavior                               |
| Access      | Authorization requirements, tenant scope                                  |
| Contracts   | Input and output contracts, compatibility requirements                    |
| Data        | Data ownership, persistence, transaction requirements                     |
| Side effects| External integrations, side effects, asynchronous behavior                |
| Repetition  | Retry behavior, idempotency, concurrency concerns                         |
| Operations  | Observability requirements, performance constraints supplied by the project |

Do not invent missing business rules. Material unresolved requirements block
READY FOR IMPLEMENTATION under
[standards/task-specification.md](task-specification.md#ready-for-implementation).

---

# 3. Architecture Boundaries

Keep clear responsibility boundaries. Conceptual boundaries may include:

- Transport or interface (HTTP handlers, message consumers, CLI entry points)
- Application or use-case coordination
- Domain or business logic
- Persistence or data access
- External integrations and adapters
- Infrastructure concerns

These are concepts, not required folder names or framework layers. Project
architecture and approved ADRs take precedence.

Avoid:

- Business rules hidden in transport handlers
- Persistence details leaking throughout the code
- Framework-specific objects becoming universal domain contracts
- Uncontrolled global state
- Circular dependencies
- Utility modules that become dumping grounds

---

# 4. Dependency Direction

Dependencies follow intentional architecture boundaries. Core business
behavior should not depend unnecessarily on HTTP frameworks, database
drivers, queue implementations, vendor SDKs, or deployment details.

Where practical, isolate infrastructure-specific behavior behind explicit
boundaries. Do not add abstraction for its own sake: a direct implementation
is acceptable when an abstraction gives no meaningful architecture, testing,
replacement, or ownership benefit.

---

# 5. Interface Contracts

Where backend interfaces or APIs exist, define their contracts explicitly:

- Request and response shape
- Required and optional fields
- Validation and authorization
- Error behavior
- Pagination, filtering, and sorting
- Versioning or compatibility, where required
- Idempotency, where required

Rules:

- Do not expose internal database models as public contracts unless
  explicitly justified.
- Do not silently change contract semantics.
- Changes to public or external contracts require a compatibility impact
  review (see [Compatibility](#22-compatibility-and-versioning)).

---

# 6. Input Validation

Validate untrusted input at a trusted boundary, following
[standards/security.md](security.md#7-input-validation). Consider type,
required fields, allowed values, length, range, identifiers, nested
structures, collection sizes, and pagination limits.

Distinguish:

- **Syntactic validation** — is the input well-formed?
- **Business validation** — is the operation valid in the current domain
  state?

Valid input does not mean the actor is authorized. Avoid contradictory
validation rules duplicated across layers; each rule has one clear owner.

---

# 7. Authentication and Authorization

[standards/security.md](security.md#5-authorization) is authoritative.
For backend code:

- Enforce authorization at a trusted backend boundary. Transport, UI, and
  client checks do not replace it.
- Authorize the actual action on the actual resource, including ownership and
  tenant scope where applicable.
- Never treat client-provided roles, ownership claims, or tenant IDs as
  authorization.

---

# 8. Error Handling

Errors are explicit and predictable. Distinguish where useful:

| Category              | Example                                              |
| --------------------- | ---------------------------------------------------- |
| Validation            | Malformed or out-of-range input                      |
| Authentication        | Missing or invalid credentials                       |
| Authorization         | Authenticated actor not permitted                    |
| Not found             | Resource absent, or hidden from this actor           |
| Conflict              | Duplicate, version mismatch, invalid state transition |
| Dependency failure    | External system returned an error                    |
| Transient failure     | Timeout or temporary unavailability                  |
| Internal failure      | Unexpected defect                                    |

- Never swallow errors silently.
- Never convert a failure into a successful response to keep a workflow
  moving.
- Do not expose sensitive internal details to untrusted clients. Keep useful
  internal diagnostic context, without secrets.
- Prefer structured error classification over matching error strings.

---

# 9. Transactions and Consistency

Where several persistent changes must succeed or fail together, define the
transaction boundary deliberately. A transaction:

- Covers the required consistency boundary
- Is no longer than necessary
- Rolls back correctly on failure
- Never silently commits part of a business operation

A database transaction does not cover external APIs, message brokers, file
storage, email, or other third-party side effects.

When database state and external or asynchronous side effects must stay
consistent, choose an approach suited to the requirement, such as a
transactional outbox, an explicit state machine, compensating actions, or
reconciliation. This standard does not mandate one. Distributed transactions
across services require explicit architectural justification, normally an
ADR.

---

# 10. Idempotency

Identify operations that may be retried or delivered more than once, such as
payment-like operations, webhook handling, message consumption, job
execution, create commands, and external callbacks.

Where repeated execution could cause incorrect state or repeated side
effects, implement idempotency and define:

- The idempotency key or identity
- Its scope
- Persistence and lifetime, where applicable
- The response to a repeated request
- Behavior when duplicates arrive concurrently

"Duplicates are unlikely" is not idempotency.

---

# 11. Concurrency

Where shared mutable state or competing requests exist, check for:

- Lost updates and stale reads
- Duplicate creation and uniqueness races
- Race conditions and double processing
- Conflicting state transitions

Use a mechanism suited to the architecture, such as database constraints,
optimistic concurrency, locking, atomic operations, serialization, or
idempotency.

Application-level "check, then insert or update" logic is not safe under
concurrency on its own. Concurrency fixes need meaningful tests where
feasible.

---

# 12. Database Interaction

[standards/database.md](database.md) is the authoritative database standard,
including schemas and migrations. In addition to it and to
[standards/security.md](security.md#14-database-security), backend code:

- Uses parameterized queries or an equivalent safe mechanism.
- Never uses database superuser credentials as the application credential.
- Makes data ownership and tenant scope explicit in data access.
- Avoids unbounded queries.
- Reviews query counts and expensive access patterns.
- Preserves transaction correctness.
- Handles not-found, duplicate, and conflict outcomes deliberately.
- Never depends on unspecified row ordering.
- Uses database constraints for invariants that must hold under
  concurrency, where appropriate.

---

# 13. External Integrations

Every external call defines:

- Timeout behavior
- Failure handling
- Authentication and credentials
- Retry policy, where applicable
- Idempotency implications
- Observability
- Contract expectations

Never assume an external dependency succeeds.

Retries:

- Are not automatic for every failure. Retry only when the operation is safe
  to repeat and the failure is plausibly transient.
- Are bounded.
- Use backoff and jitter where appropriate, to avoid retry storms.

Never log external credentials or secrets. Incoming webhooks and events
follow [standards/security.md](security.md#16-external-integrations-and-webhooks).

---

# 14. Asynchronous Processing

Where queues, workers, events, or background jobs are used, define:

- Message or job ownership
- Payload contract
- Tenant context, where applicable
- Retry behavior and maximum attempts
- Poison message or dead-letter handling, where supported or required
- Idempotency
- Ordering assumptions
- Concurrency behavior
- Observability
- Failure recovery

Assume at-least-once delivery wherever the infrastructure may redeliver,
unless the platform's actual contract guarantees otherwise. Do not claim
exactly-once processing unless the architecture provides it end to end.

Background work preserves the authorization and data-isolation context it
needs. Do not silently lose the security context when work moves off the
request path.

---

# 15. Time and Scheduling

Time-dependent behavior states its assumptions:

- Timezone, and whether the value is an instant or a local calendar date
- UTC storage for instants, where appropriate
- Business-local timezone and daylight-saving behavior, where relevant
- Clock boundaries, expiration, scheduling, and retries

Do not convert a business date that represents a local calendar date into a
UTC instant. Never depend implicitly on the server's local timezone. Tests
control time where time affects business behavior.

---

# 16. Configuration

- Configuration is explicit and environment-aware.
- Configuration is separate from secrets; secrets follow
  [standards/security.md](security.md#9-secrets-and-credentials).
- Required configuration is validated at startup or another appropriate
  boundary, and the service fails clearly when it is missing or invalid.

Avoid:

- Hidden production defaults
- Insecure default credentials
- Silently missing critical configuration
- Environment-specific values hardcoded in application logic

Development conveniences must not silently become production configuration.

---

# 17. Logging

Follow [standards/security.md](security.md#17-logging-and-audit). Use
structured, contextual logging where supported. Useful context includes
correlation or request ID, operation, actor and tenant identifiers where
appropriate, resource identifier, outcome, dependency, and duration.

Never log passwords, tokens, or secrets. Do not log sensitive payloads
without explicit need.

Log a failure once, where it is handled, rather than at every layer. Logs
help diagnose behavior; they must not become an uncontrolled store of
sensitive data.

---

# 18. Observability

For operationally meaningful components, consider:

- Logs, metrics, and traces
- Health and readiness indicators
- Dependency health
- Error rates, latency, and retry behavior
- Queue depth, where queues are used

Measure meaningful system behavior, not arbitrary numbers.

- Health endpoints must not expose sensitive internal diagnostics publicly.
- Distinguish process health from readiness where the platform requires it.
- A transient dependency failure should not automatically kill a healthy
  process unless the architecture requires that behavior.

---

# 19. Performance and Resource Safety

Avoid premature optimization, but check for obvious resource risks:

- Unbounded queries, collections, request bodies, or concurrency
- Connection and resource leaks
- N+1 access patterns and repeated external calls
- Expensive work in synchronous request paths
- Uncontrolled threads, tasks, or workers
- Memory-heavy buffering

Base performance requirements on actual project needs where possible. Never
trade correctness or security for speculative optimization.

---

# 20. Resource Lifecycle

Release resources such as transactions, database connections, files, network
responses, locks, timers, and worker resources on every path, including
errors.

Propagate cancellation and timeouts where the platform supports them and
doing so is meaningful. Long-running work stops when its owning operation is
cancelled, unless it is intentionally detached by design.

---

# 21. Pagination and Query Safety

Collection endpoints avoid unbounded retrieval when datasets can grow.
Define, where applicable:

- Default and maximum page size
- Stable ordering
- Cursor or offset behavior (neither is mandated)
- Filtering constraints

Pagination, filtering, and sorting never bypass authorization or tenant
scoping.

---

# 22. Compatibility and Versioning

Backend changes consider their consumers: web and mobile clients, other
services, external partners, workers, stored messages and events, and
scheduled jobs.

Review breaking changes to APIs, events, schemas, configuration, and behavior
consumers rely on.

- Make backward compatibility requirements explicit where they exist.
- Follow the project's compatibility strategy.
- Do not add versioning for appearance only.

---

# 23. Testing

Testing is risk-based. Where applicable, cover:

- Business logic and validation
- Authorization and tenant isolation
- Persistence and transactions
- Concurrency and idempotency
- External failures and retries
- Asynchronous processing
- Error mapping
- Compatibility-sensitive behavior

Prefer deterministic tests. Mocks must not hide important integration
behavior: use integration tests where correctness depends on the real
behavior of database constraints, transaction semantics, routing and
middleware, serialization, queues, or external protocols.

Never report tests as passed unless they were executed. Never weaken tests to
make CI pass.

---

# 24. Testability

Design important business behavior to be testable without unnecessary
infrastructure.

- Place boundaries around meaningful dependencies and side effects, not
  around every function.
- Do not add interfaces or mocks solely to make something testable when
  simpler options exist.
- Make time, random values, external calls, and other nondeterministic
  inputs controllable where deterministic tests depend on them.

---

# 25. Code Quality

Prefer clear naming, cohesive functions and modules, explicit behavior,
minimal hidden side effects, consistent error handling, and focused
responsibilities.

Avoid:

- Oversized handlers, controllers, or service objects
- Duplicated business rules
- Unexplained magic values
- Deep implicit coupling
- Dead code and commented-out implementation
- Speculative abstraction
- Generic helper packages with unclear ownership

Follow the project's language and framework conventions where they do not
conflict with Engineering OS governance.

---

# 26. Security Integration

[standards/security.md](security.md) is authoritative for security. Backend
work considers, where applicable: authentication, authorization, tenant
isolation, secrets, sensitive data, injection, file handling, external
trust, logging, and dependency security.

This standard creates no weaker backend-specific alternative to any security
requirement.

---

# 27. Review Expectations

Backend review follows [standards/code-review.md](code-review.md) and its
[severity levels](code-review.md#severity-levels). This standard does not
define another severity model.

Backend review examines, where applicable: requirement correctness,
architecture boundaries, interface contracts, authorization, tenant scope,
transaction correctness, concurrency, idempotency, error behavior, external
dependency failure, asynchronous semantics, observability, performance and
resource safety, tests, and compatibility.

---

# 28. Governance and Scope Drift

If backend implementation reveals missing business rules, unresolved
architecture, invalid assumptions, missing security decisions, incompatible
contract requirements, or materially expanded scope, do not invent a
solution.

- Ordinary implementation defects follow the normal correction and re-review
  path.
- Readiness-invalidating discoveries follow BLOCKED recovery.

Both paths, and who may make each transition, are defined in
[standards/task-specification.md](task-specification.md#transitions). This
standard does not redefine them.

---

# 29. Backend Checklist

Use before implementation and during review. Mark a row `N/A` with a reason
when it does not apply. Not every backend has a database, queue, HTTP API,
multiple tenants, or external integrations.

| Area                 | Check                                                                                   |
| -------------------- | --------------------------------------------------------------------------------------- |
| Requirements         | Behavior, actors, side effects, and failure behavior defined; nothing invented          |
| Architecture         | Boundaries respected; business rules have clear ownership, not obscured by transport; approved ADRs followed |
| Interface contract   | Shapes, errors, and compatibility explicit; no silent semantic change                    |
| Validation           | Untrusted input validated at a trusted boundary; syntactic vs business validation clear  |
| Authorization        | Enforced in the backend for the actual action and resource                              |
| Tenant isolation     | Tenant from a trusted source; data access and jobs scoped                                |
| Transactions         | Boundary deliberate; no partial commits; external side effects not assumed covered      |
| Concurrency          | No unprotected check-then-act; constraints or locking where needed                      |
| Idempotency          | Repeatable operations have a defined key, scope, and repeat behavior                    |
| Database access      | Parameterized; least-privilege credentials; bounded; deliberate conflict handling        |
| External integrations| Timeouts; bounded, safe retries; failure handling; credentials protected                |
| Async processing     | At-least-once assumed; idempotent consumers; retry and dead-letter defined; context kept |
| Time                 | Timezone and date semantics stated; no implicit server timezone; time controllable in tests |
| Configuration        | Validated; separate from secrets; no insecure or hidden production defaults             |
| Logging              | Structured; no secrets or unnecessary sensitive data; no duplicate logging              |
| Observability        | Meaningful signals; health endpoints not leaking internals                              |
| Performance          | No unbounded work; no leaks; no N+1 or repeated calls in hot paths                      |
| Compatibility        | Consumers identified; breaking changes reviewed and explicit                            |
| Tests                | Risk-based; deterministic; integration tests where real behavior matters; only executed results reported |
| Security             | [Security checklist](security.md#26-security-checklist) applied                          |
