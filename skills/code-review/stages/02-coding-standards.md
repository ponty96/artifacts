# Apply coding standards

## Find the rules for each changed path

Start from the target repository root even if the review starts in a subfolder. Honor any explicitly supplied standards file. Match Markdown filenames without regard to case. Treat `CODING_STANDARDS.md`, `coding-standards.md`, and `coding standards.md` as equivalent names.

Check the repository root, the ancestors of changed files, and their `ai-rules/` directories. Look in nested `ai-rules/` directories when they apply to a changed path. Read relevant standards that repository guidance points to, including a style guide with a different name. Use the document's stated scope first and prefer the more specific rule when it overrides a broad rule. If equally applicable files disagree, identify the conflict and leave that fix as a suggestion.

Record the sources you used. If there is no applicable document, follow existing conventions and use Google for general review criteria such as design, complexity, naming, comments, and consistency. Treat preferences as preferences. If the diff itself changes a standards document, read both revisions and check which rules the change intends to adopt. [Google review guide](https://google.github.io/eng-practices/review/reviewer/looking-for.html#style)

## Apply and verify routine fixes

When the review mode permits local edits and the target revision is available, apply clear fixes automatically. Use the smallest patch that implements the rule. Keep the patch in the changed code and any directly affected glue. Preserve unrelated local edits. For a remote PR, use an appropriate local checkout or a safe isolated copy of the reviewed head.

Review the resulting patch against the Stage 01 diagram and invariants. Confirm that the same callers can use the same contracts and that ordering, failure behavior, access decisions, and side effects still match. Run the relevant existing formatter, type check, or tests when available. A diagram establishes the flow. Use the diff and tests to check details it cannot capture.

If a fix requires a behavior change or the evidence cannot establish equivalence, leave it as a review finding or suggestion. Revert only the reviewer-owned edit if a check shows it alters the intended behavior. Record the standards rules applied, paths edited, and verification results. For later analysis, use the selected review snapshot with the reviewer edits applied. If other working-tree edits affect a staged-only review, verify the combined patch in isolation. Leave the index untouched unless the user asks to stage changes. Keep the original review snapshot for comment anchors.
