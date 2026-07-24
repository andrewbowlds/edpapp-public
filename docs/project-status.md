# Project Status

An honest, feature-by-feature status of the ecosystem, so nothing in this repo reads as more finished than it is. Development began **April 2025**; the platform is deployed and receives regular updates. I use four labels:

- **Live** — deployed and in production use
- **Production-capable** — fully implemented and working, but with limited current adoption or volume
- **Partial** — implemented but incomplete, or implemented with known remaining work
- **Planned** — a stated direction, not yet built. **Nothing labeled Planned should be read as deployed.**

## Payments & rent — stated precisely

This is the area most prone to overstatement, so it gets its own table:

| Capability | Status | Detail |
|---|---|---|
| Invoice payment via Stripe Checkout | **Live** | The live payment path for the ecosystem |
| Tenant rent charge via Stripe Checkout (in-app) | **Production-capable** | A real tenant rent-charge flow exists end-to-end; this overview does not claim a specific transaction volume |
| Landlord disbursement pipeline (Stripe Connect) | **Production-capable** | Charge → ledger → disbursement exists in code, including a held state for landlords who haven't completed payment onboarding |
| Third-party property-management platform reconciliation | **Live** | Relevant external records are reconciled with EDP through internal operator tooling; relative payment volume is not claimed |
| Plaid bank linking + transaction import | **Live** | Used in the bookkeeping flows |
| Recurring/scheduled automated rent billing | **Planned** | Recurring invoice structures exist; automated scheduling is not wired up |

The honest summary: **rent collection through the app is a production-capable feature that exists and works; this overview does not claim a specific adoption level or transaction volume.**

## By application

### edpmain — public marketing site + lead capture — **Live**

| Feature | Status |
|---|---|
| Public marketing site, listings, blog, neighborhood/city pages | Live |
| 10+ distinct lead-capture forms (contact, cash offer, valuation, flat-fee, wholesale, agent/referral applications, W-9, iBuyer) | Live |
| SEO surface (sitemap, robots, structured data) | Live |
| Stripe Checkout for invoice payment | Live |
| Magic-link SSO handoff to the agent CRM | Live |
| Remaining structured-data/OpenGraph coverage gaps | Planned |

### edpAgentNet — agent CRM, transactions, e-sign packets, AI comms — **Live** (largest app)

| Feature | Status |
|---|---|
| Lead management (creation, claiming, activity tracking) | Live |
| Transaction/deal management (large multi-field deal model, parties, commissions, expenses) | Live |
| Document OCR field detection | Live |
| E-sign packet creation (signing experience delegated to edpEuphoriSign) | Live |
| Twilio voice/SMS + SendGrid email communications | Live |
| Pierce AI voice agent (OpenAI) | Live — see [`ai-agent-system.md`](ai-agent-system.md) |
| In-app Genkit/Gemini AI flows | Live |
| Multi-organization readiness (scoped phone numbers, per-org branding, cross-org lead routing) | Planned/Partial |

### edpmanagement — property management — **Live** (most tested app)

| Feature | Status |
|---|---|
| Property/unit/lease/tenant/landlord management | Live |
| Tenant, landlord, and vendor self-service portals | Live |
| Maintenance request intake and escalation workflow | Live |
| Inspections (move-in/periodic/move-out) and deposit disposition | Live |
| Tenant rent charge + landlord disbursement pipeline | Production-capable (see payments table) |
| Vendor coordination and work orders | Live |
| Public rental listing sync to the marketing site | Live |
| Emulator-backed security-rules test suite + CI gate | Live |
| Tenant screening in-app integration | Planned — currently routed to an external flow |
| Lease e-signing wired to the e-sign product | Planned — a product decision with open questions |
| Organization-scoped storage rules and full org-ID coverage | Partial |

### edpadmin — user provisioning, form library — **Live** (small control plane)

| Feature | Status |
|---|---|
| Firebase Auth user CRUD with role assignment | Live |
| Shared form-template library (feeds e-sign) | Live |
| Centralized activity-logging endpoint used by all apps | Live |
| Organization creation | Partial — known remaining work distinguishing organization records from rental-tenant records |
| Multi-organization ownership (subscription provisioning, plan limits, custom domains, admin invites) | Planned |
| Automated test coverage | Planned |

### edpbilling — invoices, bills, payments — **Live**

| Feature | Status |
|---|---|
| Invoice creation/lifecycle, vendor bills with markup | Live |
| Payment ledger, Plaid bank-transaction import | Live |
| Chart of accounts / journal entries (double-entry) | Live |
| Its own in-app payment page | Partial — superseded by the primary Checkout path |
| Automated test coverage | Planned |
| Platform-level subscription billing (for a multi-org model) | Planned |

### edpEuphoriSign — e-signature product — **Live**, single-organization

| Feature | Status |
|---|---|
| Token-based signer flow, ESIGN-consent capture, identity verification (access code / one-time code) | Live |
| Sequential/parallel multi-recipient signing order | Live |
| PDF field overlay + flattening | Live |
| Hash-chained tamper-evident audit trail + completion certificates | Live |
| Background job queue (finalization + email) with retry/backoff and health check | Live |
| QR-code public intake flow | Live |
| Multi-organization support | Planned — currently single-organization |
| Automated test coverage | Planned |

**Note on framing:** this is an audit-grade, tamper-evident signing product. I describe it that way rather than asserting legal enforceability — the implementation captures ESIGN consent and maintains a tamper-evident trail, but it does not claim, and I do not claim, a legal-enforceability guarantee.

### edp-mobile-app — **iOS in TestFlight; Android in limited internal use**

| Feature | Status |
|---|---|
| iOS app (Swift/SwiftUI) — role-based views, lead capture, maintenance, transactions, invoice viewing/payment, push notifications | Functionality built and in use; **distributed via TestFlight**, not publicly released on the App Store |
| iOS public App Store release | Planned |
| iOS floor-plan / room-scan feature | Built, in the app's feature set |
| Android app (Kotlin/Compose) — UI/view-model parity with iOS | **Partial — in limited internal use; not production-ready.** Some data-layer repositories are still backed by stub implementations |
| Android store release | Planned |
| Typed push-notification payload schema | Planned |
| Field-naming convergence with the canonical standard | Partial — active migration, coordinated with web because mobile releases lag behind Apple's review process |

## AI-agent system status

Full detail in [`ai-agent-system.md`](ai-agent-system.md).

| Capability | Status |
|---|---|
| Pierce voice agent (OpenAI, live phone, tool-calling) | Live |
| Pierce SMS / email | Live — conversational reasoning runs in an agent runtime outside the application repo |
| Event → agent pipeline (Cloud Functions) | Live on the producing side |
| Time-based escalation for stale requests | Live |
| Audit logging of agent actions | Live |
| Capability-scoped authorization, server-side policy enforcement, human-approval controls | **Active priorities** — being strengthened; not presented as complete |
| Count of production AI agents beyond Pierce | **Not claimed** — the architecture supports additional specialized roles; Pierce is the agent operating today |

## Internal operator tooling status

| Tool | Status |
|---|---|
| Firestore MCP service (staged read/write/approve modes, approval step for destructive operations) | Live — in internal operational use |
| Permission-scoped MCP variant (collection-level restrictions + field-level redaction) | Live — built for restricted access |
| External PM-platform MCP bridge (reconciliation) | Live — internal use |

## The multi-organization direction

Several apps carry readiness backlog items (org-scoped data, per-organization branding, subscription billing, custom domains). To be unambiguous: **the system today runs one brokerage as a single organization.** Serving additional brokerages is a real, documented *direction* with meaningful groundwork in some apps and little in others. It is **Planned**, not shipped, and nothing in this repository should be read as claiming a live multi-organization SaaS product.
