# Production-Agent Evaluation Harness

The rental-email agent communicates with prospective tenants without a person reviewing every response before it is composed. That makes evaluation a deployment control, not a demo score.

## Current documented suite

| Measure | Current state |
|---|---:|
| Evaluation cases | 35 |
| Criteria | 18 |
| Meta-tests of scorers and gates | 66 |
| Safety gates | 12 at a required 100% |
| Documented clean cases | 34 of 35 |

The remaining documented failure was traced to an over-specified harness assertion rather than unsafe agent behavior. That distinction is important: the evaluation system can be wrong too, so its rules and scorers need tests of their own.

## Design principles

### Test what is deployed

The adapter loads the deployed agent instructions directly rather than maintaining a copied evaluation prompt. A change to the real instructions therefore changes what the suite evaluates.

### Keep quality and safety separate

Quality criteria—such as resolving the correct unit, answering the prospect's question, and preserving the leasing funnel—use a 90% target. Safety criteria—such as fair-housing behavior, invented availability, and required disclosures—are gates at 100%.

A combined score could conceal the failure that matters most. High tone or helpfulness scores do not compensate for a safety violation.

### Prefer deterministic evidence

The scorer hierarchy is:

1. deterministic string and structural assertions;
2. contextual compliance patterns;
3. grounded checks against property and workflow facts;
4. limited model-based judging for subjective criteria such as tone and responsiveness.

Model judges do not grade safety.

### Test the tests

The 66 meta-tests feed each scorer known-good and known-bad examples. This has caught patterns that missed ordinary human phrasing, fixture behavior that made a route unreachable, and assertions that punished correct behavior.

## What production data changed

The first cases were based on the specification. Reviewing actual message patterns exposed conditions the specification did not capture well:

- the same prospect identifier could be rendered more than once in a notification;
- listing sources and aggregators did not always identify themselves consistently;
- duplicate inquiry volume was higher than the original test set assumed;
- an application notification could resemble a new lead while requiring the opposite action;
- real listing URLs did not match an early scorer's assumed format.

Those findings changed the cases, parsers, routing categories, and meta-tests. The lesson was not merely to add more examples; it was to derive cases from the operating environment rather than from the intended interface alone.

## A safety case that corrected the harness

One high-risk test involved an assistance animal at a unit whose ordinary pet policy did not allow pets. The correct response needed to distinguish an assistance animal from a pet and avoid applying pet fees or restrictions.

The agent handled the distinction correctly, but a blunt forbidden-phrase assertion rejected the response because it mentioned the ordinary policy before immediately explaining the lawful exception. The assertion—not the agent—was wrong. The fix narrowed the deterministic prohibition to conduct that is wrong under any framing and left contextual refusal behavior to a contextual scorer.

This led to a durable rule for the evaluation system: a safety gate must be strict about the actual prohibited outcome without rejecting lawful context.

## Known limitations

- This is development-time evaluation, not continuous production monitoring.
- The mocked data layer cannot yet reproduce every timeout, stale record, or partial external-system failure.
- Failure-injection coverage is not yet part of the documented suite.
- Thirty-five cases are useful for regression detection, not a statistically complete accuracy estimate.

The next maturity step is sampled production monitoring and controlled failure injection while keeping deterministic safety gates as the non-negotiable baseline.
