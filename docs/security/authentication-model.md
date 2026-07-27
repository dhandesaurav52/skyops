# SkyOps Authentication Model

## Purpose

This document defines how SkyOps authenticates humans, services, automation, integrations, CLI clients, VS Code extensions, webhooks, and AI tool execution paths. It complements the security architecture and does not define APIs or implementation code.

## Authentication principles

- Authentication identifies an actor; it does not authorize actions.
- Credentials must be short-lived where practical and revocable always.
- Enterprise identity providers are preferred for human access.
- Every authentication event must be observable and auditable.
- Tenant context must be resolved after authentication through organization membership and policy.
- High-risk actions require stronger authentication through MFA or step-up verification.

## Actor types

### Human users

Human users authenticate through local account flows only if enabled, enterprise SSO, or identity-provider initiated flows. A user can belong to multiple organizations, but each organization independently controls membership, role assignment, MFA, session policy, and lifecycle.

### Organization administrators

Organization administrators are human users with privileged permissions. They require MFA and are subject to stronger session, audit, and anomaly monitoring controls.

### Service accounts

Service accounts represent non-human automation owned by an organization, workspace, or project. They use scoped credentials, explicit ownership, expiration policy, and usage audit.

### Internal services and workers

Internal services and workers authenticate with workload identity, mTLS, or short-lived service tokens. Their identity is separate from the original human or automation actor whose request they may process.

### CI/CD runners

Runners use scoped, short-lived credentials bound to organization, project, pipeline, job, and environment context. Runner credentials must not be reusable outside their job or assigned environment.

### CLI clients

CLI clients authenticate through OAuth device flow, browser-based authorization, or scoped API keys for automation. CLI sessions must show current organization and workspace context before destructive actions.

### VS Code extension

The extension authenticates as the user through secure OAuth/OIDC flows. Local token storage must be minimized and protected by operating system credential storage where possible.

### Webhook senders

Webhook senders authenticate through provider signatures, shared signing secrets, public key verification, event IDs, timestamps, and replay protection.

### AI assistant tool executor

AI tool execution uses a distinct execution identity combined with the requesting actor context. The AI executor cannot independently expand permissions beyond the requesting actor and approved policy.

## Authentication lifecycle

1. Actor initiates login or credential presentation.
2. SkyOps validates the credential, provider response, signature, or workload identity.
3. SkyOps resolves actor identity and actor type.
4. SkyOps evaluates organization membership and identity policy.
5. SkyOps creates a session or short-lived token with limited audience.
6. Every subsequent request is authorized independently.
7. Sessions and tokens are refreshed, rotated, revoked, or expired according to policy.

## Enterprise SSO

SSO is organization-scoped. Each organization can configure identity providers, allowed domains, JIT provisioning policy, group mappings, MFA enforcement, and break-glass recovery.

SSO requirements:

- Validate issuer, audience, signature, state, nonce, and expiration.
- Require verified domains before domain enforcement.
- Prefer SCIM for user lifecycle and group membership.
- Treat identity provider group claims as inputs to authorization mapping, not direct unrestricted permissions.
- Audit SSO configuration and login events.

## OAuth2 and OIDC

OAuth2/OIDC is used for identity providers and external integrations.

Rules:

- Use Authorization Code with PKCE for public clients such as CLI and editor extension.
- Use confidential clients only where secrets can be protected server-side.
- Validate all token and authorization response parameters.
- Use least-privilege scopes.
- Encrypt and rotate integration refresh tokens.
- Keep user-delegated credentials separate from organization-level integration credentials.

## MFA model

MFA is required for privileged roles and high-risk actions. Preferred factors are phishing-resistant methods such as WebAuthn/passkeys. Authenticator apps are acceptable for broad compatibility. SMS should not be preferred for privileged access.

Step-up MFA is required for:

- SSO, SCIM, MFA, and session policy changes.
- Owner and security administrator changes.
- API key creation with high-risk scopes.
- Production deployment approvals when policy requires.
- Sensitive data export.
- High-impact AI tool execution.
- Break-glass access.

## Session model

Browser sessions should use secure, HTTP-only, SameSite cookies where possible. Sessions have idle timeout, absolute lifetime, organization policy constraints, revocation support, and device inventory.

Session events requiring revalidation or revocation:

- Password or identity provider reset.
- MFA reset.
- Organization deprovisioning.
- Role or group removal.
- Suspicious login.
- User-initiated logout from all devices.
- Security administrator revocation.

Tenant switching clears tenant-scoped caches and revalidates permissions.

## JWT strategy

JWT access tokens are short-lived, audience-specific, issuer-validated, and key-rotated. Mutable permissions should not be treated as permanently embedded token truth. Sensitive authorization should be resolved server-side or rechecked against current policy.

JWT claims should be minimal and avoid sensitive data. Tenant context may be included only when the token is intended for a specific tenant audience.

## Refresh token strategy

Refresh tokens are high-value credentials.

Requirements:

- Rotation on use.
- Reuse detection.
- Secure storage.
- Revocation by user, administrator, system, or organization policy.
- Expiration and maximum lifetime.
- Binding to client, device, or session where feasible.

## API key and personal token authentication

API keys are suitable for automation where interactive login is impractical.

Rules:

- Keys are scoped to organization, workspace, project, environment, and permissions.
- Keys have expiration and owner.
- Keys are shown only once.
- Verification material is hashed or encrypted according to design needs.
- Last-used metadata is recorded.
- Key creation, use, and revocation are audited.
- Organization policy can restrict creation and maximum lifetime.

## Service-to-service authentication

Internal authentication requirements:

- Workload identity or mTLS preferred.
- Static service secrets avoided.
- Tokens are short-lived, audience-restricted, and environment-bound.
- Callers propagate original actor, tenant, request ID, and trace ID.
- Callees authorize both service identity and delegated actor intent.

## Webhook authentication

Inbound webhooks require:

- Signature verification.
- Timestamp tolerance.
- Replay protection using event ID or nonce.
- Provider-specific payload validation.
- Rate limiting.
- Deduplication.
- Tenant integration lookup without trusting unverified payload fields.

Outbound webhooks require:

- Tenant-scoped signing secrets.
- Secret rotation.
- Delivery audit.
- Retry limits.
- Payload redaction based on configuration.

## Authentication monitoring

Monitor:

- Login failures.
- Impossible travel or unusual device changes.
- MFA enrollment and reset.
- Session refresh anomalies.
- Token reuse detection.
- API key unusual usage.
- Webhook signature failures.
- CLI and extension authentication anomalies.
- Privileged login outside expected patterns.

Authentication monitoring feeds audit logs, security alerts, and incident response workflows.
