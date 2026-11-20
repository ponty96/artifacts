# Prepare the test plan

Run relevant existing tests when the target code and a suitable environment are available. Record each command or manual check and its observed result. If an environment issue prevents a check, report that limit separately from code failures.

For each material proposed improvement or unresolved finding, give the user a check they can run independently before adopting it. Include a concrete command or manual steps, setup and inputs, and the observation that would support or disprove the claim. State whether the check ran. Use an isolated patch or disposable test setup for a side-by-side comparison when the proposed fix is absent.

- Database. Use the same representative data, parameters, and statistics on both sides. Compare result sets as well as plans. Use plain `EXPLAIN` for estimates. Run `EXPLAIN (ANALYZE, BUFFERS)` only in an appropriate test environment because it executes the statement. Compare measured rows and buffer work, and include write or lock effects when relevant. [PostgreSQL EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- Concurrency or cleanup. Use a barrier, an injected failure, or cancellation to exercise the relevant sequence. Assert the invariant, such as one write or all workers exiting. Repeat under representative concurrency when timing matters.
- Metrics or user-visible behavior. Trigger the affected path and say which metric, response, or interface state to observe. Include a baseline or a control when it helps isolate the effect.

Prefer checks that would fail if the concern is real. Test the main changed path and relevant boundaries. Include focused regression coverage for behavior-preserving fixes when existing tests can establish equivalence. [Google review guide](https://google.github.io/eng-practices/review/reviewer/looking-for.html#tests)
