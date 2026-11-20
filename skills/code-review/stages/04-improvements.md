# Look for improvements

## Follow the diff outward

Start with the changed module. Read the neighboring code, interfaces, callers, and tests needed to evaluate the new behavior. Check how the change fits the wider system and behaves for users. Look for edge cases, unnecessary complexity, clear names, and comments that explain intent. Check whether tests detect regressions and whether affected usage or operation docs are current. For a user-facing change, exercise the affected flow or identify what a demo would establish. [Google review guide](https://google.github.io/eng-practices/review/reviewer/looking-for.html)

When reuse looks plausible, search for exact symbols first. Try one focused search for similar function or variable names, or for a domain term. Follow promising results with matching contracts. Stop when the search has enough evidence to support or rule out the idea. A matching name alone gives weak evidence.

A new module can use an existing implementation or replace an existing one. Before proposing either direction, compare caller contracts, input and output semantics, ordering, failure handling, and ownership. If the replacement would also change unrelated callers, call out that extra scope and consider a follow-up.

## Improvement rubric

This rubric is non-exhaustive. Use it to find weaknesses in a candidate before you report it.

- Change and trigger. Point to the changed path. Describe the input, load, failure, or sequence that makes the problem visible. Check the base and existing feedback.
- Impact. Specify who or what is affected. For defensive code, trace the input boundary and any upstream guarantees before adding checks or removing them.
- Evidence. Prefer a focused test, trace, query plan, or a complete code path that establishes the behavior. Check local conventions and nearby tests. Before claiming a race, show the conflicting operations and a feasible interleaving. Before claiming a leak, show which resource remains live and which exit path skips cleanup.
- Fit. Compare the proposed fix with an example from this repository, an accessible service in the organization, or prior verified reviews. Make sure the constraints match. Call out costs such as an index's write overhead or a metric's label count.
- Confidence. When a potential benefit remains plausible but you cannot verify it or find a close precedent, use a non-blocking suggestion and state what would confirm it. Leave out weak speculation. A complete code trace can establish an issue without a runnable test.

## Four examples from published engineering guides

1. Duplicate writes on retry. Trace whether retries can arrive after a commit before the caller receives the response. Check whether the request identifier and mutation share an atomic boundary. Test a retry and concurrent duplicates with the same identifier. [Amazon Builders' Library](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/)
2. Resource leak on cancellation. Follow a producer when a consumer exits early. Check whether cancellation unblocks the producer and releases its resources. Exercise the early exit and observe worker completion. [Go pipelines and cancellation](https://go.dev/blog/pipelines)
3. Slow database query. Compare plans with representative data, actual parameters, and up-to-date statistics. Check whether selective filters or joins justify an index. Measure rows, buffers, and response time, and consider extra work on writes. [PostgreSQL EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
4. Missing operational signal. Check whether existing metrics show the failure the new path can introduce. Choose a concrete measure such as error rate, tail latency, or resource saturation. Verify that the signal changes under the expected failure or load. [Google SRE monitoring guide](https://sre.google/sre-book/monitoring-distributed-systems/)

## Use prior review evidence

When authorized review memory or an internal retrieval system is available, search it for the failure mode and a verified fix. Treat similar incidents as leads and validate them against the current diff. To support future reviews, record verified lessons in an authorized store when the workflow allows it. Capture the trigger, effect, evidence link, verification method, and limits on reuse. Keep the review usable when no memory is available.
