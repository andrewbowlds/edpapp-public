# Lessons Learned

Real ones, from building and operating a production system since April 2025 where much of the code was AI-generated. These are the stories I'd actually tell in an interview. All specifics are stripped of identifying detail, and issues described here have been resolved — I'm writing about what I learned, not publishing a map of current weak points.

## Coordination bugs live in the seams

**What happened:** two separate systems ended up sharing responsibility for escalating stale maintenance requests, and their responsibilities overlapped instead of being cleanly partitioned. For a period, both acted on the same records, producing a burst of duplicate escalation messages — including outbound texts that should have gone out once.

**Why it happened:** each system was individually reasonable and individually tested. The bug lived in the *seam* between them — a coordination boundary no single component's logic or tests covered. This is the characteristic failure mode of a system assembled from many independently-correct pieces, and it's exactly where AI-generated code tends to hide problems: each generated piece does what you asked, but nobody asked "what happens when both of these run against the same record?"

**The fix:** cleanly partition responsibility between the two systems, add per-record cooldowns and caps so no record can generate a flood, and default new outbound automation to a validated dry-run before enabling it.

**The lesson:** in an event-driven system, the tests that matter most are about *interactions and idempotency*, not individual component correctness. And when actions are outbound and reach real people, the cost of a coordination bug isn't a stack trace — it's a real person getting messaged repeatedly. That permanently reframed how I think about rate limits, cooldowns, and validated-before-live defaults as first-class safety features rather than niceties.

## Money math is where AI-generated code hurts most

**What happened:** several bookkeeping bugs surfaced only through operating the billing side against real data — misrouted internal accounting, double-counted ledger entries, and a statement generator returning empty results when it should have returned a full period.

**Why it happened:** money logic is full of plausible-looking-but-wrong implementations. An AI assistant will confidently produce a function that sums transactions in a way that looks right, reads right in review, and is subtly wrong on a real edge case (a refund, a held amount, a period boundary). Those are precisely the cases you don't think to specify, so they don't get generated correctly and don't get caught in a quick read.

**The lesson:** for money and dates, "it looks right and the demo worked" is not evidence of correctness. This category of logic needs adversarial test inputs (refunds, holds, boundaries) or reconciliation against a source of truth. I now treat any AI-generated financial function as guilty until reconciled.

## Important constraints belong in code, not only in instructions

**What I learned:** when a system includes an AI agent taking actions, it is tempting to express business constraints as instructions to the model, because that's fast and it usually works. "Usually" is the trap. An instruction to a model is a strong suggestion, not an enforcement mechanism.

**The lesson, and the priority it set:** any constraint that genuinely matters — a limit, a confirmation, an approval step before something reaches a customer — belongs in server-side, testable enforcement, with model instructions as a secondary layer rather than the only one. This is now the first thing I look for when evaluating any AI system that touches production: where is the real boundary, and is it enforced in code? Recognizing that in my own system is what made **strengthening capability-scoped authorization, server-side policy enforcement, and human-approval controls** the top engineering priority in the AI-agent workflow — and that work is in progress, which is exactly how I describe it in [`ai-agent-system.md`](ai-agent-system.md) rather than claiming it's finished.

## Migrating a live system teaches patience

**What happened:** the authorization model needed to move toward Firebase custom claims, and field names across apps needed to converge on a canonical standard. Both are deliberately still in progress.

**Why it's slow on purpose:** real users have active sessions and cached mobile data. A hard cutover breaks in-flight reads; a bulk rename breaks a mobile client that's a review cycle behind the web. So every change is dual-write → backfill → cutover, spread across multiple deploys, carrying both old and new state for a while.

**The lesson:** intermediate-state complexity in a live system is not necessarily sloppiness — it can be the *evidence* of doing migrations safely. I learned to prefer a slow, boring, reversible migration over a clever fast one, and to read that kind of complexity in someone else's codebase as a possible sign of care rather than mess.

## AI-assisted development is a review discipline

The meta-lesson: generating the code was never the hard or valuable part. The value was knowing what to build, specifying it precisely, and then *reviewing the output adversarially* — reading generated code with specific suspicion about where AI assistants tend to be plausibly wrong (edge cases, money, dates, coordination seams, and constraints expressed as intentions rather than enforcement). The bugs that got through got through where my review was too trusting. The system works, with real users and real workflows, because the review was mostly good — and it's better now because I know exactly where it wasn't.

That's the job I'm applying for, described from the inside.
