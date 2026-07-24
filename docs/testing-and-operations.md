# Testing & Production Operations

Written the same honest way as the rest of this repo: the testing story here is genuine but uneven, and I think an accurate picture is more useful to a hiring manager than a polished one.

## Testing coverage

Approximate automated test coverage per app:

| App | Coverage | Kind of testing |
|---|---|---|
| edpmanagement | Strongest | Emulator-backed Firestore **security-rules tests**, API-route authorization tests, business-logic unit tests (payment matching, pre-screening) |
| edpAgentNet | Moderate | Unit tests of library functions (lead lifecycle, message parsing, referral logic) with a fake-Firestore helper, a component test, a lightweight smoke spec |
| edpmain | Light | Unit tests of pure helper functions |
| edpadmin / edpbilling / edpEuphoriSign | Minimal to none | Coverage here is a stated priority |

**What this says honestly:** coverage is concentrated where the risk is highest — the app governing tenant/landlord/vendor data and money workflows has the most meaningful tests, including authorization-level ones. But it isn't uniform, and extending coverage to the remaining apps (particularly those touching money and documents) is real prioritized work rather than something I'll imply is already done.

## The testing that matters most

The property-management app's **security-rules tests are the piece I'd point a hiring manager at first.** They:

- Load the actual authorization rules into the Firebase Firestore emulator
- Assert concrete role/ownership scenarios with explicit pass/fail expectations
- Skip cleanly when the emulator isn't running, so they don't produce false failures
- Run in CI on every push and pull request

For an architecture where authorization is expressed in rules, testing those rules directly against the emulator is the highest-leverage test available, and it's genuinely in place.

## CI/CD

CI exists at the individual-app level (each app is its own repository within a submodule structure), not as one unified pipeline:

- **edpmanagement** — a real quality gate. On push to main and on pull requests, CI provisions the toolchain (including the Firestore emulator) and runs typecheck → security-rules tests → build. A change that breaks types, breaks a rules test, or breaks the build fails CI. All configuration values in CI are placeholder stubs, not real secrets.
- **edpbilling** — a build-and-deploy workflow to Firebase App Hosting on merge. Adding a test gate in front of this deploy is on the priority list.
- **edpmain / edpmanagement** — additional scheduled "daily maintenance" workflows (an AI-assisted maintenance agent that opens automated PRs), since reduced to manual-trigger-only after the scheduled version was judged not worth its noise. A small but honest example of turning off automation that wasn't earning its keep.
- Remaining apps have no CI workflows yet.

**Honest read:** CI enforcement is real but concentrated. Making it uniform is a known next step.

## Tooling and quality gates

- **TypeScript everywhere**, with typecheck scripts — the type system is a meaningful correctness gate given that much of the code is AI-generated.
- **ESLint** across apps, with the public site configured for zero-tolerance on warnings.
- **Pre-commit hooks** on the public site.
- **Dependency-vulnerability scanning** is not yet wired into CI — a stated priority.

## Deployment

- **Firebase App Hosting** (Cloud Run-backed) is the deployment target for all six web apps, each with resource configuration tuned to its own traffic profile.
- Two apps retain a legacy hosting rewrite configuration alongside App Hosting — evidence of a real migration between hosting models rather than a clean single target.
- **Cloud Functions (2nd gen)** deploy for the property-management and billing apps, with a typecheck build step as a predeploy hook.
- The **iOS app** ships through Xcode and is distributed via **TestFlight**, with release cadence gated on Apple's review process; the **Android app** is a Gradle project with no CI wired up yet, consistent with its limited-internal-use status.

## Operational reality

This is a system I operate, not just one I built. Since April 2025 that has meant:

- Being the person on call for it. When an early notification workflow produced duplicate outbound messages due to a coordination bug between two systems, I diagnosed it and designed the fix (see [`lessons-learned.md`](lessons-learned.md)).
- Reconciling real money. Several bookkeeping and ledger bugs were found and fixed through operating the billing side against real data — the kind of thing tests alone don't catch.
- Running data migrations against live production data using dual-write/backfill/cutover, coordinating web and mobile in lockstep because the mobile client can lag a web change by a review cycle.
- Turning off automation that wasn't worth its noise, and validating new automated outbound workflows before enabling them.

**What good operations would add** at higher scale: centralized cross-app observability, richer alerting, monitoring of the agent pipeline, and a test gate in front of every production deploy rather than just some.
