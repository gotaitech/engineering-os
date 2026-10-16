# Security Standard

This standard defines the minimum security engineering baseline for software
designed, implemented, reviewed, tested, and maintained under Engineering OS.

- Project-specific security requirements may strengthen this standard. They
  must not silently weaken it; any weakening is a
  [security exception](#25-security-exceptions).
- This standard sets expectations and decision boundaries. It does not
  mandate a framework, authentication protocol (such as JWT or OAuth), cloud
  provider, orchestration platform, or encryption algorithm, and it does not
  invent regulatory requirements.
- It does not replace threat modeling for high-risk systems,
  project-specific security architecture, or approved ADRs.

[AGENTS.md](../AGENTS.md) is authoritative. Task lifecycle is owned by
[standards/task-specification.md](task-specification.md), architecture
authority by [standards/architecture-decisions.md](architecture-decisions.md),
review by [standards/code-review.md](code-review.md), and Git handling by
[standards/git.md](git.md). If this standard conflicts with any of them,
stop and report the conflict to the Human Tech Lead. Do not resolve it
yourself.

---

# 1. Principles

- **Secure by default.** The default configuration and behavior are the safe
  ones.
- **Least privilege.** Actors, services, and credentials get only the access
  they need.
- **Deny by default** for authorization. Access is granted explicitly, never
  assumed.
- **Explicit trust boundaries.** Know where untrusted data and actors enter.
- **Defense in depth.** No single control is the only protection for
  high-risk assets.
- **Minimize sensitive data.** Collect, expose, log, and retain only what is
  required.
- **Validate untrusted input** at trust boundaries.
- **Server-side authorization** at a trusted enforcement boundary.
- **Auditable security-sensitive behavior**, proportional to risk.
- **Fail safely.** Failure denies access rather than granting it.
- **Scale with risk.** Depth of controls, testing, and review grows with the
  impact of failure.

Security controls must not be disabled to make implementation, tests, CI, or
deployment succeed. Tool limitations do not create security exceptions.

---

# 2. Task Preparation

Security is identified when the task is prepared, not first discovered in
review. In the task's **Security Considerations**, identify where applicable:

- Authentication requirements
- Authorization rules, roles, and permissions
- Tenant boundaries
- Sensitive data
- External trust boundaries and integrations
- Secrets
- File uploads
- Privileged and destructive operations
- Audit requirements
- Regulatory or compliance constraints supplied by the project
- Abuse and misuse cases

Mark categories `N/A` with a reason where they do not apply.

Do not invent missing security-sensitive business rules (for example, who
may perform an action). Missing rules are open questions. An unresolved
security-sensitive decision blocks READY FOR IMPLEMENTATION until the
Tech Lead approves it, as required by the
[readiness gate](task-specification.md#ready-for-implementation).

---

# 3. Architecture Decisions

Security-sensitive architecture may require an ADR under
[standards/architecture-decisions.md](architecture-decisions.md), for
example:

- Authentication architecture and identity provider strategy
- Authorization model
- Tenant isolation architecture
- Secrets architecture
- Encryption and key-management architecture
- Trust boundaries
- Security-sensitive external integrations

AI agents may propose; only the Human Tech Lead approves. A small security
implementation detail inside an approved design does not need its own ADR.

---

# 4. Authentication

Where authentication is required:

- Use established, maintained mechanisms and libraries.
- Authenticate at trusted, server-controlled boundaries.
- Protect authentication state from tampering and disclosure.
- Handle expiration and revocation appropriately.
- Do not expose credentials or tokens in URLs, logs, errors, or client code.
- Do not invent custom cryptography or password algorithms.
- Prevent bypasses, such as debug logins, test backdoors, or fallback paths
  that skip authentication.

Passwords, where stored, use an established password hashing scheme designed
for password storage, as chosen by the project or platform standard.

Authentication proves identity. It does not grant permission.

---

# 5. Authorization

Enforce authorization server-side or at another trusted enforcement
boundary. Never rely solely on:

- Hidden UI elements or disabled buttons
- Client-side route guards
- Client-provided roles, ownership claims, or tenant IDs

Apply least privilege and deny by default. Every protected operation answers:

1. **Who** is the actor?
2. **What** action are they attempting?
3. **Which** resource are they acting on?
4. Are they **permitted** to perform this action on this resource?

Changes to authorization require deeper review and negative tests.

---

# 6. Multi-Tenant Isolation

For multi-tenant systems:

- Tenant context comes from a trusted source, such as the authenticated
  identity, never from an arbitrary client-supplied value.
- Tenant isolation is enforced at trusted, server-side boundaries.
- Queries and resource access are scoped to the tenant.
- Background jobs carry and enforce the correct tenant context.
- Caches do not serve data across tenants.
- Files and object storage preserve tenant isolation.
- Logs, events, and queues do not leak data across tenant boundaries.

An authenticated user in Tenant A must not reach Tenant B's resources by
changing an identifier. Test cross-tenant access attempts.

Tenant isolation architecture may require an approved ADR.

---

# 7. Input Validation

Treat all external input as untrusted. Validate, as appropriate:

- Type, format, length, and range
- Allowed values (prefer allow-lists where practical)
- Identifiers
- File metadata
- Structured payloads

Reject invalid input safely. Validation does not replace authorization.

Never build database queries, shell commands, templates, file paths, or other
interpreted expressions by concatenating untrusted input. Use parameterized
database access or an equivalent safe mechanism.

---

# 8. Output and Injection Safety

Encoding and escaping must match the context where the output is
interpreted:

| Context       | Concern                                                   |
| ------------- | --------------------------------------------------------- |
| HTML          | Context-aware encoding; avoid unsafe raw HTML insertion   |
| JavaScript    | Do not inject untrusted data into executable script       |
| SQL           | Parameterized queries                                     |
| Shell         | Avoid shells; pass arguments as arrays, not strings       |
| URLs          | Encode components; validate redirect targets              |
| Templates     | Use auto-escaping; avoid rendering untrusted templates    |
| Logs          | Neutralize control characters and line breaks             |
| HTTP headers  | Reject or encode line breaks                              |

"Sanitize everything" is not a control. Name the context and the mechanism.

---

# 9. Secrets and Credentials

Never commit:

- Passwords
- API keys
- Access or refresh tokens
- Private keys and signing secrets
- Production credentials
- Secret environment files

Use the project-approved secret and configuration mechanism. Do not place
secrets in source code, client-delivered bundles, or logs.

If a secret is exposed or committed:

1. Report it immediately to the Tech Lead, without reproducing the secret.
2. Assume that deleting it from the latest file is insufficient.
3. Expect that rotation or revocation may be required.
4. Expect that repository history or build artifacts may need remediation.

Git remediation follows [standards/git.md](git.md#sensitive-data), coordinated
by the Tech Lead. Do not automatically rewrite history. Rewriting history does
not revoke an exposed credential.

---

# 10. Sensitive Data

Identify sensitive data from project context, such as:

- Personal data
- Credentials and authentication or session data
- Financial information
- Health information
- Customer confidential information
- Security-sensitive operational data

Apply data minimization: collect, process, expose, log, and retain only what
is required.

- Do not expose fields merely because the database or API model contains
  them. Return only what the consumer needs.
- Do not invent regulatory classifications. Use those supplied by project
  requirements; if none are supplied and the data appears regulated, ask.

---

# 11. Encryption and Cryptography

- Use established platform or library cryptography. Never implement custom
  cryptographic algorithms.
- Never hardcode cryptographic keys.
- Decide, as required by the project:
  - Encryption of data in transit
  - Encryption of data at rest
  - Key ownership, storage, and rotation
  - Maintenance of algorithms and libraries

Encryption does not solve authorization or tenant isolation. Security-sensitive
cryptographic architecture may require an ADR.

---

# 12. Sessions and Tokens

Where sessions or tokens are used:

- Protect them from unnecessary exposure.
- Define expiration behavior.
- Define invalidation or revocation where required.
- Never log token values.
- Use secure transport.
- Consider replay risk.
- Store them appropriately for the target platform. The storage mechanism is
  a project decision; this standard does not mandate one.

Do not place authorization decisions solely in untrusted client state.

## Browser-Managed Credentials

This subsection applies only when browser authentication uses cookies or
other credentials the browser attaches automatically. It does not mandate
cookie sessions, and it does not apply to mobile, API, or other non-browser
token architectures where these controls are irrelevant.

- **Cookie attributes.** Set `Secure`, `HttpOnly`, and `SameSite` as
  appropriate for the application's architecture and cross-site
  requirements. A relaxed `SameSite` value needs a stated reason.
- **CSRF.** Protect state-changing operations against cross-site request
  forgery whenever the browser attaches credentials automatically. Use the
  framework's or project's established mechanism; this standard does not
  mandate one. `SameSite` reduces CSRF risk but does not replace CSRF
  protection in every architecture (for example, cross-site deployments,
  same-site subdomains, or older clients). Safe methods (such as GET) must
  not change state.
- **Session fixation.** Do not accept session identifiers supplied before
  authentication as authenticated sessions.
- **Rotation.** Issue a new session identifier after authentication, and
  rotate or invalidate the session after meaningful privilege or
  security-context changes (for example, role change, password change, or
  step-up authentication), where appropriate.
- **Logout and invalidation.** Logout invalidates the session server-side,
  not only in the browser. Define how sessions are invalidated after
  revocation events.
- **Testing.** Where applicable, include negative tests for missing or
  invalid CSRF protection, fixation attempts, and use of a session after
  logout or invalidation.

---

# 13. API Security

Where APIs exist:

- Authenticate where required, and authorize each protected operation.
- Validate input and constrain resource access.
- Use appropriate status codes; do not return sensitive error detail.
- Apply rate limiting or abuse controls based on risk.
- Protect privileged and administrative endpoints.
- Bound pagination and query sizes where unbounded requests create risk.

An unpublished or obscure URL is not access control.

---

# 14. Database Security

Where databases exist:

- Application credentials are least-privilege. A database superuser is never
  the default application credential.
- Use parameterized queries or an equivalent safe mechanism.
- Protect sensitive data.
- Review destructive operations and migrations.
- Preserve tenant and data-ownership boundaries.
- Control which credentials can run migrations.
- Never expose database credentials to clients.
- Consider the security of backups where applicable.

---

# 15. File Uploads

Where file uploads exist, consider:

- Allowed file types, with content validation where security depends on the
  actual type. Do not trust the filename extension or client-supplied
  Content-Type alone.
- Size limits
- Filename and path safety
- Storage isolation, including tenant isolation
- Authorization for both upload and download
- Malware scanning, based on project risk
- Preventing uploaded content from becoming executable application code
- Private or signed access where appropriate

---

# 16. External Integrations and Webhooks

For external systems:

- Protect integration credentials.
- Validate payloads.
- Define the trust boundary.
- Use timeouts and handle external failures safely.
- Do not leak internal or sensitive errors to the external party.

## Webhook and Event Authenticity

An incoming webhook or event must have its authenticity and integrity
verified before it can perform a security-sensitive or protected state
change. A payload is not trusted merely because it arrived at the endpoint,
or because the provider offers no request signature.

**Where the provider offers verification:**

- Use the provider's documented verification mechanism. Do not invent a
  provider-specific signature scheme.
- Reject requests that fail verification.
- Test failed and invalid verification.

**Where direct verification is unavailable:**

- Do not silently trust the payload.
- Use an approved alternative trust mechanism where feasible, such as:
  - authenticated reconciliation with the provider's API,
  - fetching the authoritative event or resource state from the provider
    before acting, or
  - another project-approved trusted verification path.

If no adequate trust mechanism exists and the event can affect protected or
security-sensitive state, the design is unresolved security architecture or
requirements. The task is BLOCKED until an approved decision exists (an ADR
where the decision is significant).

**For all webhooks and events:**

- Protect against replay and duplicate delivery where applicable, using
  idempotency, event identifiers, or timestamps as the provider supports.
- Include negative tests for failed authenticity verification and replayed
  events where applicable.

---

# 17. Logging and Audit

- Never log secrets.
- Avoid logging sensitive personal or customer data unless explicitly needed.

Security-sensitive operations may require audit records, proportional to
project risk, such as:

- Authentication and security events
- Permission changes
- Privileged actions
- Destructive actions
- Security configuration changes

Audit records capture actor, action, time, and resource where applicable,
without exposing secrets. Application users must not be able to silently alter
security audit history unless the design explicitly permits it and is
approved.

---

# 18. Error Handling

Security-related failures fail safely: access is denied, not granted.

Do not expose to untrusted clients:

- Stack traces
- Credentials or secret values
- Internal infrastructure details
- Unnecessary database details

Keep enough internal diagnostic information to troubleshoot, without logging
secrets. Never convert an authorization failure into success to keep the
application usable.

---

# 19. Dependencies and Supply Chain

- Use maintained dependencies from trusted sources. Avoid unnecessary ones.
- Review security impact when introducing a dependency, making a major
  upgrade, relying on an abandoned package, or receiving a vulnerability
  finding.
- Lock or pin dependency versions as appropriate for the ecosystem.
- Do not upgrade blindly; consider compatibility and behavior changes.
- Assess known vulnerabilities by severity, exploitability, exposure, and
  project context, and report them as AGENTS.md requires.

Do not suppress or hide vulnerability scanner findings to make CI pass. A
finding judged not applicable is recorded with its reason.

---

# 20. Infrastructure and Configuration

Where applicable:

- Minimize exposed services and ports.
- Apply least privilege.
- Keep secrets separate from source.
- Never ship default or insecure credentials.
- Use secure transport.
- Restrict privileged and administrative interfaces.
- Review what is publicly exposed.
- Protect production configuration.

Development conveniences (debug modes, permissive CORS, disabled
verification) must not silently reach production. Containers, orchestration
platforms, cloud services, and private networks are not secure by themselves.

---

# 21. Frontend and Client Security

Browser, mobile, and other client applications are untrusted enforcement
environments.

- Never place server secrets, privileged credentials, or authoritative
  authorization logic solely in client code.
- UI permission checks improve usability; they do not replace server-side
  authorization.
- Avoid unnecessary persistence of sensitive data on the client.

---

# 22. Security Testing

Security testing is risk-based. Where applicable, test:

- Unauthenticated access
- Unauthorized access and ownership violations
- Cross-tenant access
- Invalid input
- Privilege escalation
- Expired or revoked authentication
- Sensitive-data exposure
- Abuse of destructive or privileged operations

High-risk security behavior needs negative tests, not only happy-path tests.
Never report security tests as passed unless they were executed.

---

# 23. Security Review

Security review follows [standards/code-review.md](code-review.md). Changes
involving any of the following require deeper security review:

- Authentication or authorization
- Tenant isolation
- Secrets or cryptography
- Sensitive data
- Privileged operations
- Destructive migrations
- Externally exposed infrastructure

Findings use the code-review
[severity levels](code-review.md#severity-levels). This standard does not
define a separate severity model. Never accept a security bypass because tests
or builds pass.

---

# 24. Security Findings and Governance

A security defect that can be fixed within the approved task is an
**ordinary correction**: CHANGES REQUESTED → IN PROGRESS → IN REVIEW, with
re-review.

If implementation or review finds an unresolved security requirement, missing
approval for security-sensitive architecture, or an invalid trust assumption,
it is **readiness-invalidating**:

IN REVIEW → CHANGES REQUESTED → BLOCKED → resolve the requirement,
architecture, or security decision → required Tech Lead approval → readiness
verified again → READY FOR IMPLEMENTATION → IN PROGRESS → validation →
IN REVIEW

During implementation, the Assigned Agent moves or reports the task as BLOCKED
directly. Do not invent business or security policy to unblock a task.
Transition ownership is defined in
[standards/task-specification.md](task-specification.md#transitions).

## Issues Found During READY FOR HUMAN APPROVAL

Lifecycle ownership, including who may make each transition, is defined
only by [standards/task-specification.md](task-specification.md#transitions).
This section adds no security-specific transition rules.

If a new security issue is found while a task is READY FOR HUMAN APPROVAL:

1. **Stop** advancement toward approval and merge.
2. **Report** the issue to the Human Tech Lead.
3. **Do not** modify and push the approval-ready content before the
   lifecycle action is decided.

**Ordinary security correction** — requirements, architecture, and readiness
remain valid, but the code needs a security fix. The lifecycle action is
taken by whoever the task standard assigns it to (for a task in READY FOR
HUMAN APPROVAL, requesting changes is the Tech Lead's decision). Where
correction is required, the task follows the task standard's
CHANGES REQUESTED path: correction → affected validation → independent
re-review → READY FOR HUMAN APPROVAL again only after the applicable gates
pass.

**Readiness-invalidating security issue** — the finding exposes unresolved
security requirements, invalid trust assumptions, missing approval for
security-sensitive architecture, invalid acceptance criteria, or another
readiness failure. Whoever discovers it may record or move the task to
BLOCKED where the task standard permits, and reports it. BLOCKED does not
authorize implementation. Recovery follows the task standard's BLOCKED
recovery:

resolve the blocker → required requirement, architecture, or security
approvals → readiness verified again → READY FOR IMPLEMENTATION →
IN PROGRESS → validation → IN REVIEW → READY FOR HUMAN APPROVAL

Actions the task standard reserves to the Human Tech Lead remain the
Tech Lead's, including the applicable CHANGES REQUESTED decisions, human
approval, merge, and DONE.

---

# 25. Security Exceptions

Urgency does not authorize:

- Bypassing authentication or authorization
- Disabling tenant isolation
- Exposing secrets
- Disabling security validation
- Ignoring critical security findings

## What Authorization Does Not Do

Explicit Human Tech Lead authorization for a security exception does **not**:

- Amend requirements or acceptance criteria
- Supersede an ADR or approve new architecture
- Make contradictory repository specifications satisfied
- Waive AGENTS.md security prohibitions (committing secrets, exposing
  credentials, disabling security controls to make tests pass, trusting
  client-provided authorization)

Human authorization also does not make an unsafe action technically safe.
If a requested exception would require an AGENTS.md prohibited action, or
creates unacceptable risk, stop and report it instead of implementing it.

## Exceptions That Conflict With Specifications

If a proposed exception conflicts with an approved requirement, acceptance
criterion, architecture decision, or other readiness condition: **STOP.**

The affected specification is changed through its normal governance, not
through the exception. Where applicable:

- Update the requirement or task, including acceptance criteria, under
  [standards/task-specification.md](task-specification.md).
- Write a new or replacement ADR and supersede the old one under
  [standards/architecture-decisions.md](architecture-decisions.md).
- Obtain the required Tech Lead approval.
- Document the security impact and risk.
- Verify readiness again.

Implementation proceeds only after those changes are approved. There is no
separate exception path around the task or ADR standards.

## Temporary Operational Exceptions

An exception that does not conflict with any specification and is not
prohibited still requires explicit Tech Lead authorization, recorded in the
task or pull request with:

- Reason
- Scope
- Duration or expiry, where applicable
- Risk
- Compensating controls
- Owner (a human)
- Remediation or follow-up

See also the exception rules in
[workflows/feature.md](../workflows/feature.md#exceptions).

---

# 26. Security Checklist

Use before implementation and during review. Mark a category `N/A` with a
reason when it genuinely does not apply.

| Category        | Check                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------- |
| Authentication  | Established mechanism; trusted boundary; expiry and revocation; no bypass paths             |
| Browser sessions| Cookie attributes; CSRF on state changes; fixation prevented; rotation; server-side logout   |
| Authorization   | Server-side; deny by default; actor, action, resource, and permission checked per operation |
| Tenant isolation| Tenant from a trusted source; queries, jobs, caches, files, and logs scoped; cross-tenant tests |
| Input           | Untrusted input validated; no unsafe concatenation; parameterized access                    |
| Output          | Encoding matches each output context                                                        |
| Secrets         | None in source, bundles, or logs; approved secret mechanism; exposure reported              |
| Sensitive data  | Identified; minimized; not over-exposed by API or logs                                      |
| Database        | Least-privilege credentials; destructive changes reviewed; migration privileges controlled  |
| Files           | Type, size, and path validated; storage isolated; upload and download authorized            |
| Integrations    | Webhook authenticity verified or approved alternative, else BLOCKED; replay handled; timeouts |
| Logging / audit | No secrets; sensitive data minimized; required audit events recorded                        |
| Dependencies    | Maintained; versions locked; vulnerabilities assessed, not hidden                           |
| Infrastructure  | Minimal exposure; no default credentials; production configuration protected                |
| Testing         | Negative tests for high-risk behavior; only executed results reported                       |
| Governance      | Security-sensitive decisions approved; required ADRs approved                               |
| Exceptions      | Recorded; no conflict with requirements or ADRs (else update specs first); no prohibited action |
| Late findings   | New security issue at READY FOR HUMAN APPROVAL reported to the Tech Lead before merge         |
