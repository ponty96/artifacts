# Understand the changes

## Establish the review scope

Read applicable repository guidance. Resolve the target repository and the requested PR, branch comparison, or local changes. Record the actual base and head revisions for a PR or branch. Use the platform's current patch for PR line anchors. For local work, identify which staged, unstaged, and relevant new files belong to the requested change.

Read the PR description and referenced intent. Check existing inline reviews and replies so you can avoid duplicate feedback. Skim the file list, then read the human-written diff line by line. Use a summary for generated output when you can trace it to the changed source.

Trace the changed code through the entry points, callers, interfaces, schemas, configurations, and tests needed to explain it. Follow an unchanged path only when it can confirm or rule out an effect of the diff. Compare the base when a concern could predate the change. Record any requested scope limit or part that needs a specialist review.

## Draw the behavior map

Produce a high-level Mermaid `flowchart` or `sequenceDiagram` before editing. Include the main entry points, changed responsibilities, and relevant downstream calls or side effects. Mark changed components in their labels. Show significant branches, writes, and error paths when they affect the contract.

Keep the diagram at the level of behavior. Record the invariants the diagram cannot show in a few lines below it. An invariant is a rule that must stay true. Examples include the order of writes, transaction boundaries, authorization decisions, and the public response shape.

Carry forward the original diagram and invariants. Record the observed flow separately from any requirement that still needs confirmation. [Google review guide](https://google.github.io/eng-practices/review/reviewer/looking-for.html#every-line)
