# Frontend Standard

This standard defines the minimum engineering baseline for user interfaces,
web and mobile applications, client-side application logic, and
frontend-to-backend integration developed under Engineering OS.

- Project-specific frontend standards may strengthen or specialize this
  baseline. They must not silently weaken governance, security,
  accessibility, contract, or correctness requirements.
- It is framework- and platform-neutral. It does not mandate a framework or
  runtime (such as React, Next.js, Vue, Angular, Svelte, React Native,
  Flutter, or native mobile), a state-management library, a CSS system, a
  component library, SPA or server-side rendering, mobile-first or
  desktop-first design, or offline support. It does not invent supported
  browsers, devices, locales, or analytics requirements.
- Approved ADRs and project architecture take precedence over the
  illustrative examples here.

[AGENTS.md](../AGENTS.md) is authoritative. Security is owned by
[standards/security.md](security.md), backend and API behavior by
[standards/backend.md](backend.md), task lifecycle by
[standards/task-specification.md](task-specification.md), architecture
authority by [standards/architecture-decisions.md](architecture-decisions.md),
and review by [standards/code-review.md](code-review.md). This standard does
not redefine them. If it conflicts with any of them, stop and report the
conflict to the Human Tech Lead. Do not resolve it yourself.

Frontend authorization and validation are user experience. They are never the
trusted security boundary.

---

# 1. Principles

- **Correctness before visual cleverness.**
- **Preserve user intent.** Do not lose input or perform actions the user did
  not ask for.
- **Clear ownership of state.**
- **Predictable behavior** and **explicit UI states**.
- **Accessible by design.**
- **Secure interaction** with trusted backend boundaries.
- **Responsive or adaptive** behavior where the project requires it.
- **Maintainable components.**
- **Observable failures.**
- **Progressive complexity.** Start simple; add structure when needed.
- **Testability.**
- **Compatibility** proportional to project requirements.

Frontend code must not bypass requirements, authorization assumptions,
validation, accessibility, security controls, or tests to make the UI appear
functional or CI pass. Tool limitations do not create engineering exceptions.

---

# 2. Requirements Before Implementation

Determine where applicable:

| Area        | Determine                                                                         |
| ----------- | --------------------------------------------------------------------------------- |
| User        | Actor, user goal, acceptance criteria                                             |
| Flow        | Screen, route, or flow; information hierarchy; required actions; navigation      |
| Access      | Permissions that affect presentation                                              |
| Data        | Data required; validation behavior                                                |
| States      | Loading, empty, error, success, disabled or unavailable, destructive-action behavior |
| Reach       | Responsive or adaptive requirements, accessibility requirements, compatibility    |
| Project-supplied | Localization, analytics or telemetry, offline or intermittent connectivity   |

Do not invent missing business or UX rules. Material unresolved behavior
blocks READY FOR IMPLEMENTATION under
[standards/task-specification.md](task-specification.md#ready-for-implementation).

---

# 3. Architecture and Responsibility Boundaries

Keep responsibilities understandable. Conceptual concerns include
presentation, interaction, state, application behavior, data access,
navigation, platform integration, and shared UI primitives. These are
concepts, not required folders or framework layers. No frontend architecture
(such as atomic design, feature-sliced design, MVC, MVVM, Redux, or Clean
Architecture) is mandated.

Avoid:

- Giant page or screen components
- Duplicated business rules
- Network or service details scattered through presentation code
- Uncontrolled global state
- Circular dependencies
- Generic utility or component dumping grounds
- Components with unclear ownership of side effects

Simple components stay simple. Do not add abstraction for architectural
appearance.

---

# 4. Component Design

Components have focused responsibilities. Prefer explicit inputs, predictable
outputs or events, understandable side effects, and composition where useful.

Avoid:

- Premature universal components
- Excessive configuration options
- Hidden global dependencies
- Substantial behavior duplicated across screens
- Components that silently perform unrelated side effects

Do not extract every small markup fragment into its own component. Reuse
should reduce duplication or improve consistency without making behavior
harder to follow.

---

# 5. State Ownership

Every piece of state has a clear owner and scope. Distinguish where
applicable:

| Kind              | Typical owner                                  |
| ----------------- | ---------------------------------------------- |
| Local UI state    | The component that uses it                     |
| Form state        | The form                                       |
| Navigation state  | The route or navigation system                 |
| Server state      | The backend; the client holds a copy           |
| Shared app state  | An explicit shared owner                       |
| Persisted client state | Client storage, with defined lifetime     |

- Keep state as local as practical. Do not put all state in a global store by
  default.
- Do not duplicate authoritative state across client stores without a
  synchronization strategy.
- Derive values instead of storing them separately when storing them would
  allow inconsistency.
- Server-owned data never silently becomes client-authoritative.

---

# 6. Server and Remote State

Remote data accounts for: loading, success, empty, error, stale data,
refetch and retry, concurrent requests, cancellation where supported,
mutation outcome, and cache invalidation or refresh.

- A successful network response does not mean the business operation
  succeeded. Check the operation's actual result.
- When the server rejects a mutation, discard or correct the optimistic
  state; do not keep it.
- Caching reflects correctness and user-experience requirements.

No data-fetching library is mandated.

---

# 7. Loading States

Loading behavior is intentional. Avoid blank unexplained screens, permanent
spinners, avoidable layout shifts, and duplicate submissions while a critical
operation is in progress.

Never show a final success state before the operation has reached the success
the product requires. Long-running operations communicate progress or status
where the project requires it.

---

# 8. Empty States

Distinguish where relevant:

- No data exists
- No search or filter results
- Data unavailable
- User lacks permission
- Data has not loaded yet

Never present an error or authorization failure as an ordinary empty list.
Offer a next action where one exists, but only actions the user is authorized
to perform.

---

# 9. Error States

Errors are understandable and actionable where possible. Distinguish where
useful: validation, authentication, authorization, not found, conflict,
network or dependency failure, rate limiting or temporary failure, and
unexpected internal failure.

- Never expose sensitive backend or internal details.
- Never convert a failure into apparent success.
- Preserve diagnostic context through approved logging or telemetry, without
  secrets or sensitive data.
- Offer retry only when retrying is meaningful and safe.

---

# 10. Forms

Forms define, where applicable: initial values, required and optional fields,
validation, error presentation, submission state, success behavior,
cancellation, and unsaved-changes behavior.

- Client validation improves usability; it does not replace backend
  validation.
- Validation messages tell the user what to correct.
- Never silently discard user input after a recoverable failure.
- Prevent accidental duplicate submission where duplicate effects matter.
  Disabling the submit button is not a substitute for server-side idempotency
  or concurrency protection.

---

# 11. Destructive and High-Impact Actions

Make consequences clear for actions such as delete, revoke, publish, final
submission, transfer, and irreversible configuration changes.

- Use confirmation where the risk warrants it, not for every harmless action.
- Offer undo or recovery only as the product actually supports it.
- Never present an action as reversible when backend or data semantics make
  it irreversible.

---

# 12. Navigation and Routing

Navigation is predictable. Consider, where applicable: deep links, refresh,
browser or platform back behavior, unauthorized and not-found routes,
preserved state, redirects, and the return destination after sign-in.

- Route hiding and navigation guards improve the experience; they never
  replace backend authorization.
- Avoid redirect loops and ambiguous route ownership.

---

# 13. Authentication and Authorization UI

Follow [standards/security.md](security.md#21-frontend-and-client-security).
Frontend authorization is presentation, not the trusted authorization
boundary.

The frontend may hide or disable unavailable actions, redirect
unauthenticated users, and explain insufficient permission. The backend or
another trusted boundary still enforces authorization.

Never treat these as proof of authorization: client-stored roles, route
metadata, hidden buttons, disabled controls, or client-provided tenant IDs.

Do not reveal sensitive information through unauthorized UI state, preloaded
data, caches, logs, or client bundles.

---

# 14. Multi-Tenant Frontend Behavior

Where multi-tenancy exists:

- The active tenant or workspace is clear where the product requires it.
- Switching tenant invalidates or refreshes tenant-scoped client state.
- Caches and persisted client state are tenant-aware and never reuse data
  across tenants.
- Routes and navigation state never expose another tenant's data.
- Optimistic state and background refreshes keep their tenant scope; a
  response for a previous tenant is discarded after a switch.

Client-side tenant filtering is not security isolation. Isolation is enforced
by trusted backend and data boundaries
([standards/security.md](security.md#6-multi-tenant-isolation)). Test tenant
switching and stale-cache scenarios where applicable.

---

# 15. API and Contract Integration

Respect backend contracts
([standards/backend.md](backend.md#5-interface-contracts)). Define where
applicable: request and response shape, required and optional fields, error
contract, pagination, sorting and filtering, and compatibility expectations.

- Do not guess undocumented backend behavior; ask or record an open question.
- Generated types alone do not prove that runtime data is valid.
- Contract changes consider mixed-version clients, especially installed
  mobile apps and cached or offline clients.
- Do not spread backend persistence models through the UI because the API
  resembles them.

---

# 16. Runtime Data Validation

Treat external and runtime data as untrusted in proportion to project risk.
Where runtime contract validation is warranted, validate at an appropriate
boundary; it is not required for every internal value.

Handle missing, malformed, unexpected, and newly added fields as project
requirements define. Never use unsafe type assertions or casts to silence
compiler errors.

---

# 17. Optimistic UI

Use optimistic updates only when rollback and reconciliation are understood.
Define where applicable: the temporary client state, rollback on failure,
reconciliation with the server, concurrent updates, duplicate actions, and
tenant scope.

Do not use optimistic UI where showing temporary success would be dangerous
or misleading. High-impact operations may require server confirmation before
showing final success.

---

# 18. Concurrency and Race Conditions

Requests do not necessarily complete in the order they started. Consider
overlapping operations such as:

- Rapid search requests
- Route changes while requests are pending
- Repeated button presses
- Autosave
- Multiple mutations
- A stale response arriving after a newer one
- A component or screen closing while work is pending
- A tenant switch during a fetch

Never let an older response overwrite newer authoritative state. Use
cancellation, request identity, sequencing, version checks, or another
appropriate mechanism.

---

# 19. Accessibility

Accessibility is part of frontend correctness. Where applicable:

- Semantic controls and appropriate heading and structure semantics
- Full keyboard interaction and visible focus
- Meaningful labels and accessible names
- Form errors that assistive technology can perceive
- Readable text and UI contrast
- Information not conveyed by color alone
- Alternative text for meaningful images
- Announcements for meaningful dynamic changes
- Usable touch and click targets
- Respect for reduced-motion preferences where applicable

Do not recreate native controls with custom elements unless there is a valid
reason and the replacement has equivalent accessible behavior.

Project standards may strengthen these requirements. Do not invent a legal
or formal conformance level; use the one the project supplies.

---

# 20. Responsive and Adaptive Design

Interfaces work across the device and viewport classes the project supports,
not only the developer's screen. Consider content reflow, text expansion, the
on-screen keyboard, safe areas, orientation, touch versus pointer input,
dynamic browser or system chrome, and device capabilities.

Neither mobile-first nor desktop-first is mandated. Supported device classes
come from project requirements.

---

# 21. Localization and Internationalization

Where localization is required:

- User-visible strings follow the project's localization approach.
- Layouts tolerate text expansion.
- Date, time, number, and currency formatting follow the intended locale and
  domain semantics.
- Avoid string concatenation that prevents correct translation.
- Support right-to-left layout if required.

Do not invent supported languages or locales. The browser or device locale is
not necessarily the user's business locale.

---

# 22. Time and Date Presentation

Preserve backend and domain semantics. Distinguish instants, local dates,
local times, and timezone-aware business times.

- Never shift a date-only business value through timezone conversion.
- Show timezone or context where ambiguity would materially affect the user.
- Never depend implicitly on the device's or developer's timezone.
- Tests control time and timezone where relevant.

---

# 23. Client Storage

Use client persistence (cookies, web storage, secure platform storage, local
databases, caches) intentionally.

- Persist only what needs persisting, with a defined lifetime where
  correctness matters.
- Never store credentials or sensitive data in insecure storage when a safer
  approved mechanism is required.
- Logout, account switching, and tenant switching clear or isolate applicable
  persisted state.

Token and session handling follows
[standards/security.md](security.md#12-sessions-and-tokens), including
browser-managed credentials.

---

# 24. File Upload and Download

Where file operations exist:

- Communicate accepted constraints.
- Handle progress and failure where useful.
- Handle retry and duplicate submission appropriately.

Client-side file name, type, and size checks improve usability; they do not
replace backend validation. Downloads must be authorized by the backend; being
able to construct a URL is not access.

---

# 25. Performance

Performance work is proportional to user impact. Consider initial load,
interaction responsiveness, unnecessary rendering or recomputation, oversized
assets, unnecessary network calls, request waterfalls, large lists, memory
leaks, cache behavior, and mobile startup cost.

Measure where practical. Do not add memoization, virtualization, caching,
code splitting, or prefetching just because those techniques exist. Never
trade correctness, accessibility, or security for speculative performance.

---

# 26. Large Collections

Avoid unbounded rendering or fetching where data can grow. Consider
pagination, incremental loading, virtualization, filtering and search, and
stable item identity. Virtualization is not mandated.

Client-side pagination and filtering are never authorization or tenant
isolation mechanisms.

---

# 27. Network and Connectivity

Where connectivity may be unreliable:

- Distinguish network failure from business failure.
- Define retry behavior and avoid retry storms.
- Preserve user input where appropriate.
- Communicate pending and synchronized state if the product supports offline
  work.

Never present a locally queued operation as completed unless the product
explicitly defines that state. Do not add offline support that is not
required.

---

# 28. Security

[standards/security.md](security.md) is authoritative. Frontend work
considers, where applicable: authentication and session handling,
authorization assumptions, tenant isolation, sensitive data, client storage,
injection and output handling, unsafe URLs and navigation targets, file
handling, third-party scripts and dependencies, secrets, and logging or
telemetry.

Secrets that grant trusted access are never embedded in client code or
bundles. This standard creates no weaker frontend alternative to any security
requirement.

---

# 29. Dependencies and Third-Party Code

Add dependencies deliberately, considering maintenance, security posture,
bundle or application size, permissions, and project compatibility.

- Do not add a large dependency for trivial functionality without
  justification.
- Third-party scripts and components never silently receive sensitive data.
- Do not replace established project libraries from personal preference.

Supply-chain rules follow
[standards/security.md](security.md#19-dependencies-and-supply-chain).

---

# 30. Design Consistency

Follow the project's design system or style guidance where it exists, and
prefer existing components that meet requirements. Do not duplicate
near-identical components.

Consistency does not override correct platform behavior, accessibility, or
user clarity. If no design system exists, do not build one as incidental
scope for a small feature.

---

# 31. Platform-Specific Behavior

Respect platform conventions. Web and native platforms differ in navigation,
back behavior, keyboard, focus, permissions, lifecycle, storage,
notifications, deep linking, and safe areas.

Shared code must not force inappropriate behavior on a platform to maximize
reuse. Web behavior is not automatically correct for mobile, or the reverse.

---

# 32. Lifecycle and Cleanup

Clean up event listeners, timers, subscriptions, observers, pending async
work, media, and platform listeners when their owner goes away. Avoid
updating state that no longer exists when the platform makes that unsafe or
misleading. Long-lived subscriptions have a clear owner.

---

# 33. Testing

Testing is risk-based. Where applicable, cover:

- Important user flows and business behavior
- Validation, and loading, empty, error, and success states
- Authorization presentation
- Tenant switching and isolation behavior
- Destructive actions
- Race conditions and optimistic rollback
- Navigation
- Accessibility
- Responsive or adaptive behavior
- Contract integration
- Network and offline behavior where required

Prefer tests that assert observable user behavior over implementation details.
Snapshot tests are never the only evidence for important behavior. Use
integration or end-to-end tests where correctness depends on real interaction
between layers or platforms.

Never report tests as passed unless they were executed. Never weaken tests to
make CI pass.

---

# 34. Testability

Make important behavior testable without coupling tests to implementation
details. Prefer semantic roles and accessible names, or project-approved test
identifiers. Do not expose internals solely for tests when behavior can be
tested through what the user sees. Control time, network, randomness, and
other nondeterminism where it matters.

---

# 35. Failure Containment

Where the platform supports error boundaries or equivalent containment, use
them in proportion to risk, so one component failure does not destroy
unrelated user work. Containment does not replace handling expected failures.
No framework-specific mechanism is mandated.

---

# 36. Telemetry and Analytics

Telemetry is intentional and authorized by the project; do not invent
analytics requirements. Where telemetry exists:

- Never send secrets or sensitive payloads.
- Respect project privacy and security requirements.
- Keep event meaning stable.
- Avoid accidental duplicate events.
- Distinguish user actions from render or view side effects.

Analytics must never trigger business side effects.

---

# 37. Compatibility

Support the browsers, operating systems, devices, app versions, and API
versions the project requires. Do not invent a support matrix, and do not
depend on features outside the supported environment.

Installed mobile and client versions can outlive a backend deployment, so
compatibility-sensitive API changes account for supported older clients
([standards/backend.md](backend.md#22-compatibility-and-versioning)).

---

# 38. Code Quality

Prefer clear naming, focused components and modules, explicit state
ownership, predictable effects, consistent error handling, and readable data
flow.

Avoid:

- Giant components
- Duplicated business rules
- Unexplained magic values
- Hidden side effects
- Excessive configuration complexity
- Dead code and commented-out implementation
- Speculative abstraction

Follow the project's framework and language conventions where they do not
conflict with Engineering OS governance.

---

# 39. Review Expectations

Frontend review follows [standards/code-review.md](code-review.md) and its
[severity levels](code-review.md#severity-levels). This standard does not
define another severity model.

Frontend review examines, where applicable: requirement correctness, user
flow, state ownership, API contracts, loading, empty, error, and success
states, authorization assumptions, tenant behavior, race conditions,
accessibility, responsive behavior, security, performance, compatibility, and
tests.

---

# 40. Governance and Scope Drift

If frontend implementation reveals missing UX behavior, missing business
rules, unresolved permissions, unresolved API contracts, missing
project-supplied accessibility requirements, unresolved architecture, or
materially expanded scope, do not invent the answer.

- Ordinary implementation defects follow the normal correction and re-review
  path.
- Readiness-invalidating discoveries follow BLOCKED recovery.

Both paths, and who may make each transition, are defined in
[standards/task-specification.md](task-specification.md#transitions).
Architecture-impacting frontend decisions follow
[standards/architecture-decisions.md](architecture-decisions.md).

---

# 41. Frontend Checklist

Use before implementation and during review. Mark a row `N/A` with a reason
when it does not apply. Not every frontend has routing, forms,
authentication, multiple tenants, localization, offline support, file
operations, analytics, or a mobile client.

| Area                  | Check                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------- |
| User and goal         | Actor, goal, and acceptance criteria clear; no invented UX or business rules             |
| States                | Loading, empty, error, success, and unavailable states defined and distinguishable      |
| Components            | Focused responsibilities; no giant components or hidden side effects                    |
| State ownership       | Clear owner and scope; no unsynchronized duplicate state                                |
| Remote state          | Network success not mistaken for business success; cache invalidation defined           |
| Forms                 | Validation messages actionable; input preserved on failure; duplicate submits handled   |
| Destructive actions   | Consequences clear; confirmation proportional; reversibility shown truthfully           |
| Navigation            | Deep links, refresh, back, unauthorized and not-found routes handled; no loops          |
| Authorization         | UI checks are presentation only; backend enforces; no data leaked via UI or caches      |
| Tenant behavior       | Switching clears tenant state; caches, storage, and responses tenant-scoped             |
| API contracts         | Contract respected; nothing guessed; mixed-version clients considered                   |
| Runtime data          | Validated where warranted; no unsafe casts                                               |
| Optimistic UI         | Rollback and reconciliation defined; not used where misleading                          |
| Races                 | Stale responses cannot overwrite newer state; cancellation or sequencing used           |
| Accessibility         | Semantic, keyboard-operable, labeled, perceivable errors and changes, not color-only     |
| Responsive            | Supported device classes work; text expansion and input modes handled                   |
| Localization          | Project locales supported; no concatenated translations; formats correct                |
| Time and date         | Instants vs local dates preserved; no implicit timezone                                 |
| Client storage        | Minimal; secure where required; cleared on logout and tenant switch                     |
| Files                 | Client checks for usability only; downloads authorized by backend                       |
| Performance           | Measured where practical; no unbounded rendering; no speculative complexity             |
| Connectivity          | Network vs business failure distinguished; queued work not shown as completed           |
| Security              | [Security checklist](security.md#26-security-checklist) applied                          |
| Dependencies          | Justified; maintained; no sensitive data to third parties                               |
| Design consistency    | Existing components used; platform behavior and accessibility preserved                 |
| Platform behavior     | Platform conventions respected; no forced cross-platform behavior                       |
| Cleanup               | Listeners, timers, subscriptions, and pending work released                             |
| Tests                 | Risk-based; user-observable assertions; not snapshot-only; only executed results reported |
| Telemetry             | Project-authorized; no sensitive data; no business side effects                         |
| Compatibility         | Project support matrix followed; older installed clients considered                     |
| Governance            | No invented rules; ADRs approved where required; lifecycle delegated to task standard   |
