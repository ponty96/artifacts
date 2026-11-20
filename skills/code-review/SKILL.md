---
name: code-review
description: Review a pull request, branch, or local diff. Map the changed behavior in Mermaid, apply repository coding standards with local fixes, validate the spec, and prepare line-level Conventional Comments with a test plan.
---

# Code review

Review the change the author intends to deliver. Use the diff as the primary scope. Apply explicit repository standards first. Use Google's review criteria for the remaining review topics. Aim for better code health and let independently sound increments move forward. [Google review guide](https://google.github.io/eng-practices/review/reviewer/looking-for.html) · [Google review standard](https://google.github.io/eng-practices/review/reviewer/standard.html)

## Run the review

Read each stage when it becomes relevant. Carry the diff revisions, behavior map, evidence, and verification results forward.

1. [Understand the changes](stages/01-understand-changes.md). Identify the exact diff and draw a high-level Mermaid diagram.
2. [Apply coding standards](stages/02-coding-standards.md). Find the rules for the changed paths. Apply clear, behavior-preserving fixes locally when the checkout and review mode allow edits. Verify them against the behavior map.
3. [Validate the spec](stages/03-spec-validation.md). Check intent and acceptance criteria. Evaluate whether a partial implementation is an independent increment.
4. [Look for improvements](stages/04-improvements.md). Follow relevant code paths and validate candidates with evidence.
5. [Classify findings and check dependencies](stages/finding-severity.md). Determine impact, confidence, and whether each finding should block. Perform the dependency check when the diff adds or upgrades a dependency.
6. [Prepare the test plan](stages/05-test-plan.md). Give the user independent checks for proposed improvements and record what you actually ran.
7. [Deliver the review](stages/review-output.md). Include the diagram, local fixes, applicable findings, and the test plan. Attach actionable comments to the current diff.

Respect a read-only request or a review surface that only provides a diff. In that case, prepare exact suggestions for fixes that you can substantiate. Keep local edits distinct from changes already present on a remote PR. Posting requires user authorization. Finish with no findings when the evidence supports that result.
