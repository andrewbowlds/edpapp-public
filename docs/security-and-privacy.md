# Security & Privacy

This document describes the system's security and privacy approach at a structural level, appropriate for a public overview. It deliberately excludes anything operationally sensitive: no rule excerpts, no project IDs, no credential names, no infrastructure hostnames, and no catalog of specific weaknesses. Where safety work is in progress, I describe it as an active priority rather than as a finished control.

## Authorization model

Authorization is expressed through **Firestore security rules**, with **Firebase Authentication** as the identity provider across every app. The role model is **mid-migration from an older stored-role approach toward Firebase custom claims**, using a compatibility layer during the transition so that live sessions and real access keep working at every step. Running that migration as a careful dual-write/backfill/cutover — rather than a risky big-bang switch — is itself a piece of the security story: evolving an authorization model on a system with real users without breaking access.

Because authorization is expressed in rules rather than a service layer, those rules are treated as first-class, testable code. The highest-risk app has **emulator-backed security-rules tests** (described below), and broadening that coverage is a stated priority.

## Data isolation

Isolation between users' data is expressed through **ownership and relationship fields** — a lease is associated with its tenant, a property with its landlords — and access is resolved from those relationships rather than a single partition key. Different roles receive different scopes appropriate to their relationship with the data: company staff, landlords, tenants, and vendors each see the subset relevant to them, and a separate demo-account mode is confined to demo data.

An organization-level isolation concept (relevant to a possible future multi-organization direction) is layered on top and is only partially applied today. That is appropriate for the current single-organization deployment; completing it would be a prerequisite for serving multiple organizations, and it's tracked as such in [`project-status.md`](project-status.md).

## Secrets management

- Secrets are kept out of source control. The repository's ignore rules cover environment files, private keys, service-account credentials, and log files.
- Server-side secrets used by serverless functions are injected as managed secrets rather than committed.
- Client-exposed configuration (public Firebase web config, Stripe *publishable* keys) appears in build config by necessity — these ship in the browser bundle by design and are not secrets. I note the distinction so "public key that's meant to be public" isn't confused with a leaked secret.

## Internal tooling designed with least privilege

The internal MCP tooling used to operate business data through an AI assistant was built **with access controls in mind**: staged read/write/approve modes, an approval step for destructive operations, and — in a permission-scoped variant intended for restricted access — collection-level access restrictions plus field-level redaction of sensitive values before any data is returned to the assistant. I describe this at the level of "designed with least-privilege controls" rather than publishing its configuration; the point for a hiring manager is that scoping an AI agent's data access was treated as a design requirement, not an afterthought.

## Audit trails

- Agent communications and actions are recorded to dedicated log collections with actor/action/timestamp information and surfaced to administrators through internal views. This is queried in practice.
- The e-signature product maintains a **tamper-evident audit trail** implemented as a hash chain: each entry's hash incorporates the previous entry's hash, so any retroactive alteration of an earlier entry breaks the chain. Combined with ESIGN-consent capture, identity verification, and completion certificates, this is an audit-grade signing process. I describe it as tamper-evident and audit-grade rather than asserting legal enforceability, which the implementation itself does not claim.

## Testing the security model

The property-management app — the highest-risk app, because it governs tenant/landlord/vendor data and touches money workflows — has **emulator-backed security-rules tests**: they load the actual rules into the Firestore emulator and assert that role/ownership scenarios succeed or fail as intended, running in CI on every push and pull request. For a rules-based authorization model, testing those rules directly against the emulator is the highest-leverage security test available, and it's genuinely in place. Extending this style of coverage across more of the system is a stated priority.

## Security-relevant priorities (stated honestly)

Rather than publish a list of specific gaps, I'll state the direction of the work, which is what a hiring manager actually needs to know I can reason about:

- **Strengthening capability-scoped authorization and server-side policy enforcement** for the AI-agent system — moving important business constraints and approval steps into application-code enforcement — is the top priority.
- **Completing the authorization-model migration** and retiring the compatibility layer.
- **Completing organization-level isolation** before any multi-organization direction.
- **Broadening automated security and authorization test coverage** across more apps.
- **Adding dependency-vulnerability scanning** to CI.

I frame these as priorities I understand and can rank, because the roles I'm targeting are exactly about being the person who can look at a live system, understand where the security investment should go next, and sequence it correctly — without turning a public document into a map of where to push.
