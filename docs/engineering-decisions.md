# Engineering Decisions & Tradeoffs

This document walks through the architectural decisions I made (or directed) and the tradeoffs each one carries. I've tried to give the honest cost alongside the benefit for each, because a decision presented without its downside isn't an engineering decision — it's marketing.

## Why the system uses multiple specialized applications

**Decision:** Seven separate Next.js applications instead of one monolith or a small number of larger apps.

**Benefit:**
- Each user-facing surface deploys on its own cadence. The public marketing site can ship SEO and content changes many times a week without being coupled to a property-management release.
- Fault isolation: an incident in one app doesn't take the others down.
- Each app's Firebase App Hosting deployment is sized independently to its own traffic and resource profile.
- Smaller apps mean smaller build times and a smaller blast radius per change — which matters a lot when most code is AI-generated and every change needs careful review.

**Cost:**
- **Schema drift is real and constant.** The same conceptual entity (a user, a property, an address, a money amount) is represented in multiple apps, and they've drifted. There's a documented canonical vocabulary and a migration backlog specifically because of this. This is the single biggest recurring tax of the multi-app design.
- No single source of truth for shared entities without deliberate discipline. Consistency is enforced by convention and code review, not by the platform.
- Cross-cutting changes (rename a field everywhere) require coordinated, multi-app migrations rather than one edit.

**Would I do it again?** For a solo/small-team operator who needs independent deploy cadences and fault isolation more than they need perfect schema consistency — yes. If I were staffing a team of engineers who could own a shared internal API layer, I'd introduce that layer (see the last section).

## Benefits and risks of sharing Firestore across applications

**Decision:** All apps share one Firestore project, with access expressed through security rules rather than routed through an internal API layer.

**Benefit:**
- Nothing to build or maintain between apps. Data written by one app is instantly, consistently readable by another.
- No serialization/versioning layer to keep in sync.
- Fast to build, which for an AI-assisted solo operation is the dominant constraint.

**Tradeoff:**
- Authorization lives in security rules rather than in a service layer, so the correctness of those rules matters a great deal and they deserve first-class testing.
- Consistency of access policy is a property of the rules, so evolving them is a careful, well-tested activity rather than an afterthought.
- At larger scale I'd introduce an internal API layer so authorization is centralized and apps aren't coupled to each other's raw schema (see the last section).

**Mitigation in place:** emulator-backed security-rules tests for the highest-risk app, run in CI (see [`testing-and-operations.md`](testing-and-operations.md)). This is genuine; broadening that coverage is a stated priority.

## Multi-tenant data isolation

**Decision:** Isolate user data by ownership/relationship fields on documents (this lease belongs to this tenant UID; this property lists these landlord UIDs) rather than by a single tenant-partition key.

**Benefit:**
- Maps naturally to the real relationships (a property genuinely has multiple landlords; a payment genuinely belongs to a chain of property → lease → tenant).
- No rigid partitioning scheme to fight when relationships are many-to-many.

**Cost:**
- Relationship-based access checks add some rule complexity and per-request cost compared to a flat partition key. That's a reasonable price for modeling the real relationships accurately.
- An organization-level isolation concept (for a possible future where multiple brokerages share the platform) is layered on top and only partially applied today. That's appropriate for a single-organization deployment; completing it is a prerequisite for any multi-organization direction, tracked in [`project-status.md`](project-status.md).

## Authentication and role management

**Decision:** Firebase Auth with custom claims for roles — migrating *toward* claims from an older model where roles were stored as Firestore document fields.

**The interesting part is the migration, not the destination.** The system runs a compatibility layer during the transition — new-model checks with fallback to the older approach — as a **dual-write / backfill / cutover** migration deliberately run without a hard switchover, because live user sessions and real data access depend on authorization working correctly at every intermediate step.

**Benefit:** custom claims are checked directly off the auth token, which is faster and cheaper than the older stored-role approach.

**Cost:** carrying a compatibility layer during the transition adds real complexity; completing the migration and retiring the legacy path is remaining work.

**Why I think it's a good example:** migrating a live authorization model with real users and no downtime is exactly the kind of thing that's easy to describe and hard to pull off safely. The intermediate complexity is the *evidence* it was done carefully rather than with a risky big-bang switch.

## Event-driven agent workflows

**Decision:** Decouple event producers (a maintenance request was created) from the AI agent consumer via a Firestore notification queue, fire-and-forget, rather than invoking the agent synchronously.

**Benefit:**
- The Cloud Function that handles a new maintenance request stays fast and reliable — it notifies humans and drops a queue item, and it's done. It never blocks on, or fails because of, the AI agent.
- The agent can be slow, be down, or be reworked entirely without affecting the core data-write path.

**Tradeoff:**
- The decoupling means the producer isn't tightly coupled to the agent's outcome, which is the point. A more mature version would add consumer-side delivery observability and dead-letter handling — a reasonable evolution as the agent system matures.

## Human-in-the-loop boundaries

Described at the right level in [`ai-agent-system.md`](ai-agent-system.md). As a decision: strengthening the enforcement of business constraints and approval steps for agent-initiated actions — moving them into server-side, testable code — is the highest-priority direction in the AI-agent system, and I treat it as an explicit priority rather than a solved problem. I call this out as a decision I'd continue to invest in, and I describe it as a direction rather than cataloguing current specifics.

## Observability and failure handling

**What exists:** structured audit logging for agent actions and communications; internal views surfacing agent activity to administrators; a health-check endpoint in the e-sign app; retry/backoff on the e-sign email job queue; and validated-before-live defaults on some automated jobs.

**What would mature next:** centralized cross-app observability, richer alerting, and consumer-side monitoring of the agent pipeline. This is appropriate work for the next stage of scale rather than a present-day emergency.

## AI-generated code review and validation

Because most line-by-line code is AI-generated, the review and validation discipline *is* the engineering, and I treat it that way:

- **I read the generated code before it ships**, specifically hunting for the failure modes AI assistants are prone to: plausible-but-wrong business logic, silent handling of an error that should surface, off-by-one or boundary errors in money/date math, and confident implementations of a spec I underspecified.
- **The highest-risk logic gets tests** — the property-management app's security rules and payment logic have real test coverage precisely because that's where an AI-generated mistake would be most expensive.
- **Real bugs got through anyway**, and I document them as lessons ([`lessons-learned.md`](lessons-learned.md)) rather than pretending the review caught everything. A coordination bug in an early notification workflow and several bookkeeping-math bugs are the honest evidence of both the risk and the eventual catch — all since resolved.

## Schema consistency across applications

**Decision:** Maintain a written canonical vocabulary (camelCase field names, money in dollars except at the Stripe boundary where cents get a `Cents` suffix, phones in E.164, emails lowercased, dates as timestamps serialized to ISO 8601 at API boundaries, `<entity>Id` naming, soft-delete via an archived flag) and a migration backlog, rather than letting each app invent its own conventions.

**Tradeoff:** the canonical standard is real and documented, but the *current state* of the data doesn't fully conform to it — the backlog exists precisely because there's drift to migrate. I'd rather have a documented standard and a tracked gap than an undocumented mess, but I'm not going to claim the data is already clean. It's being cleaned, deliberately, one dual-write/backfill/cutover at a time.

## Migration strategy for a live system

**Decision:** Never bulk-rename fields on a live system. Every schema change follows dual-write (write both old and new field), backfill (populate the new field on historical documents), cutover (switch reads to the new field, then stop writing the old one).

**Why:** the system has real users with active sessions and cached mobile-app data. A bulk rename would break in-flight reads and stale mobile caches. The mobile constraint is especially real — even with TestFlight distribution, a review cycle means the mobile client can lag a web change by days, so web and mobile field changes have to be coordinated in lockstep or the old client breaks.

**Cost:** every migration is several deploys spread over time instead of one commit, and the intermediate state (both fields present) is carried in the data and the code for a while. That intermediate complexity is visible all over the codebase — and it's the correct price for not breaking production.

## What should change if usage increases substantially

If this went from one brokerage and a few dozen users to many brokerages and thousands of users, the priorities in order:

1. **Introduce an internal API layer** between apps so authorization is centralized rather than expressed per-app, and apps aren't coupled to each other's raw schema.
2. **Complete the custom-claims migration** and retire the compatibility layer, so authorization is one fast token check.
3. **Modularize the authorization rules by domain** so they're testable and reviewable in pieces.
4. **Move key AI-agent business constraints and approval steps into server-side, tested enforcement** — the highest-value direction, and one that only matters more with scale.
5. **Broaden automated test coverage** to the apps that currently have less of it before iterating on them at speed.
6. **Complete the organization-level isolation model** as a prerequisite for any multi-organization direction.
7. **Add cross-app observability and alerting**, including monitoring of the agent pipeline.
