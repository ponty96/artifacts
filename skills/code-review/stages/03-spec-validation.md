# Validate the spec

Read the PR description and any linked ticket, Linear issue, proposal, acceptance criteria, or rollout plan. Use the available connector or repository tools. When material sources conflict, look for a later explicit scope decision. Record what you could confirm and what remains unclear.

Map each relevant requirement to the changed behavior and its evidence. Consider the PR's stated scope and what its merge or deployment would expose. Mark a requirement as satisfied, deferred, unmet, or uncertain. If no spec is available, review against the stated goal and established public contracts.

## Partial delivery rubric

This rubric is non-exhaustive. Use it to test the evidence in both directions before raising a finding.

- Scope. Look for PR wording, a linked follow-up, a stacked PR, or a rollout sequence that supports the proposed split. A broad parent ticket can span several PRs. A title alone gives weak evidence.
- Activation. Trace whether the new behavior is reachable when this PR merges. Check old and new consumers, feature flags and their defaults, job registration, and deployment order where relevant.
- Safety on its own. Check compatibility, useful behavior within the current boundaries, and whether this change leaves a broken path or a data invariant unmet. For a migration, check mixed app versions and the rollout plan. For domain code, check direct tests and existing consumers.
- Explicit obligations. Identify any criterion assigned to this exact change or contract that must hold immediately. Search for a follow-up or scope decision before declaring it missing.
- Counterevidence. Look for existing code that already meets the requirement. Check whether a missing capability could be supplied by a later layer without changing what this PR promises.

An additive migration can be ready before domain code. Domain logic can be ready before endpoints if existing flows stay sound and tests cover the new rule. When the evidence supports an independent increment, treat the remaining work as deferred. If making the split explicit would help, suggest a PR description note or follow-up link as a non-blocking comment.

If scope is ambiguous and the impact matters, ask a focused non-blocking question. Raise an issue only for a confirmed requirement of this PR or a concrete broken path that merging it would introduce.
