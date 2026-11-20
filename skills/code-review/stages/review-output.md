# Deliver the review

Lead with actionable findings in impact order. If there are none, say so. Include the high-level Mermaid diagram from Stage 01 and any relevant rollout or spec uncertainty. Call out a specific good practice when it helps the author or future reviewers. List local fixes and their verification. Give the independent test plan and distinguish tests run from tests proposed.

## Write Conventional Comments

Use one concern per comment. Begin with the [Conventional Comments](https://conventionalcomments.org/) label and a concise subject. Give the trigger, consequence, evidence, and a proposed action when one is known.

```text
issue (blocking): Make duplicate requests share a single write

Two workers can both pass the existence check before either inserts a row.
Exercise two calls with the same request ID and assert that exactly one row is written.
```

Use `issue` for an established defect, `chore` for an applicable repository requirement, `suggestion (non-blocking)` for an optional change, and `question (non-blocking)` for a material uncertainty. Use `nitpick (non-blocking)` sparingly. Keep severity and blocking status distinct.

## Attach the comment to the diff

Use the smallest relevant range in the current patch. Attach new or surviving code on the new side. Attach removed code on the old side when the platform supports that. If the affected line is outside the patch, attach to the closest related changed line and name the actual affected path. If there is no related anchor, put the concern in the review summary. For local reviews, use the available inline review format or a file and line reference. If a reviewer edit shifts a line, label which line belongs to the original staged or branch diff and which line belongs to the working tree.

When the user authorizes posting, re-check the current PR head, diff anchors, and existing feedback before submitting. Post through the native PR review surface. Report a direct link to the submitted review. Keep comments based on local fixes or a newer local diff out of old remote line anchors. For a review draft, return prepared comments with their proposed anchors.

For GPT-authored reviews, put `Reviewed by GPT` in the review body once. Include it in a standalone posted comment when there is no review body. Follow the repository's attribution rules for other reviewers.
