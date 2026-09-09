---
name: code-review-changes
description: Fresh-eyes review of task-relevant uncommitted changes, followed by informed triage and accepted fixes. Invoke once an implementation is believed working and before implementation-complete, when asked to review or sanity-check the current diff, or before commit or PR creation if the task is unreviewed. Skip active iteration, trivial diffs, and reviews of committed ranges, branches, or existing PRs.
license: Internal
compatibility: Requires the `code-comments` skill for the comment and documentation review.
metadata:
  version: "1.3"
  category: quality
---

# Code Review Changes

You wrote this code, so you cannot review it cold. Delegate two deliberately disjoint reviews in parallel, then use your implementation context to triage their findings:

- **Reviewer A — core code review:** correctness, edge cases, architecture fit, project conventions, security, accessibility, and missing tests.
- **Reviewer B — comment and documentation hygiene:** comments and developer-facing documentation affected by the diff, with an emphasis on clarity, concision, accuracy, and resistance to drift.

Separate scopes give both reviewers enough attention to find meaningful issues without duplicating each other's work. They must also write to different report paths so parallel completion cannot overwrite a report.

This review is a bounded phase, not a loop. In the normal completion flow, `implementation-complete` calls this skill at most once, this skill returns after applying accepted findings, and `implementation-complete` validates the settled result. Review fixes and validation fixes do not automatically trigger another review.

## Workflow

1. **Confirm this run is needed and define its scope.** Review once when the task's implementation is believed working. Include task-relevant staged, unstaged, and untracked files. Preserve and exclude unrelated pre-existing user changes; tell both reviewers the exact exclusions. Do not self-invoke again for the same task merely because review or validation produced edits. A later run requires an explicit user request or a genuinely new implementation phase.
2. **Resolve fresh report paths.** Resolve the operating system's temporary directory and current branch name. Sanitize the branch name for filenames by replacing `/` with `-`; use `detached-head` if no branch name is available. Build the two literal absolute paths described below and clear stale files at those exact paths before delegating.
3. **Spawn both review sub-agents before waiting for either one.** Use the prompt templates below and launch both in the same response/tool batch when the environment supports it. This concurrency is intentional.

   - If sub-agents are unavailable, invoke two independent agents through the available CLI.
   - If no delegation mechanism exists, perform both reviews yourself as separate passes and write both reports in the specified formats.

4. **Wait for both reviewers to finish.** Each reviewer writes only to its assigned report path.
5. **Read both complete report files.** Do not rely on the returned summaries alone. A missing current-run report is a failed review, not an empty report; retry that reviewer once or report the failure.
6. **Triage every finding against your implementation context.** Decide whether to address, defer, or dismiss each item. Scope boundaries, explicit user decisions, intentional trade-offs, unrelated user-owned changes, and changes made after a reviewer inspected the diff are legitimate reasons not to apply a finding.
7. **Apply accepted core-review fixes first.** Correctness changes can invalidate line numbers, comment wording, and documentation assumptions.
8. **Re-check Reviewer B's findings against the updated diff, then apply the accepted comment and documentation cleanup.** Do not apply stale verdicts mechanically.
9. **Return control to the caller.** If `implementation-complete` invoked this review, resume its next validation section without invoking this skill again. If this was a standalone review request, run only focused checks needed to avoid handing back obviously broken edits, then report both paths, dispositions, and checks. The full completion gate remains responsible for comprehensive validation.

Apply accepted fixes without pausing for routine confirmation. Stop and ask only when a finding requires a product decision, expands scope, risks data loss, introduces a breaking API change, or conflicts with prior user instructions.

## Review budget and repeat policy

- Count a review as performed only after both current-run reports were read and triaged.
- One completed run satisfies the review precondition for the current task. Accepted review edits do not invalidate it.
- Fix build, lint, type-check, or test failures inside `implementation-complete`; they do not reset the review precondition.
- Do not use report timestamps or leftover files to infer review state. Track it in the current conversation/task state.
- If conversation history was compacted, use its summary or handoff as evidence. Any indication that the review workflow completed or its reports were read and triaged satisfies the budget even if details were compressed. Do not repeat a review merely because exact state was lost; if there is no evidence either way and a non-trivial uncommitted task diff remains, run one review.
- Run again only when the user explicitly asks or when the work has become a materially different implementation rather than a fix to reviewed work. Never begin that later run automatically from inside this skill.

This budget prevents `review → fix → review` recursion while preserving a deliberate fresh-eyes pass before final validation.

## Report paths

Resolve `{{report-path-a}}` and `{{report-path-b}}` before spawning either reviewer and replace the placeholders in their prompts with literal absolute paths:

- Reviewer A: `{{os-temp-dir}}/code-reviews/{{sanitized-branch}}-review.md`
- Reviewer B: `{{os-temp-dir}}/code-reviews/{{sanitized-branch}}-comments.md`

Create the parent directory and remove any existing files at these two exact paths before the run. Fixed names are intentional: the reports are temporary evidence for the current triage, and clearing them prevents a failed reviewer from appearing to have succeeded.

## Reviewer A prompt — core code review

Fill in all `{{...}}` placeholders before spawning the agent, including the task scope and literal report path.

```text
You are an expert code reviewer. Review the task-relevant uncommitted changes described below and write actionable findings to a markdown report. Report only; do not edit source files.

Review scope: {{review-scope}}
Explicit exclusions: {{review-exclusions}}

Another reviewer is auditing comments and developer documentation in parallel. Leave wording, redundancy, clarity, and staleness to that reviewer. Mention a comment or document only when it is materially misleading about a behavioral issue you are reporting.

Steps:

1. Run `git status --porcelain`, `git diff`, and `git diff --staged` to understand the working tree, but review only the stated scope. Inspect in-scope untracked files directly because they do not appear in a normal diff. Do not critique excluded user changes.
2. Read every affected file in full; the diff alone can hide important surrounding context.
3. Read the repository's applicable `AGENTS.md` files and any directly relevant project guidance or conventions they reference.
4. Review the changes as a coherent solution, not only as isolated lines.
5. Write the report to `{{report-path-a}}`. Use exactly this literal path; do not derive another filename.

Focus on issues a human reviewer would actually raise:

- Whether the implementation solves the requested problem completely and at the right layer
- Correctness bugs and edge cases, including error handling, boundaries, concurrency, resource lifetime, and state consistency
- Violations of repository conventions or architecture boundaries
- Missing or inadequate tests for new or changed behavior
- Unnecessarily broad public API exposure
- Accessibility or user-experience regressions
- Security and privacy risks such as leaked secrets, unsafe parsing, missing authorization, or insecure defaults

Avoid:

- Comment and documentation cleanup assigned to the parallel reviewer
- Style nits a configured formatter or linter should catch
- Speculative refactors without a concrete correctness, maintenance, or performance benefit
- Praise-only observations
- Repeating the same root cause for every affected file; consolidate it once with all relevant locations

Use this report format:

# Code Review: [branch-name]

**Date**: [YYYY-MM-DD]
**Reviewed Files**: [count]

## Executive Summary
[Two or three sentences describing the change and overall risk.]

## Findings

### [High | Medium | Low] Concise finding title
- **Location**: `path/to/file:line`
- **Problem**: [What is wrong and when it matters.]
- **Recommendation**: [A concrete correction.]

[Repeat by descending severity. Omit this section and state that no actionable findings were found when appropriate.]

## Cross-Cutting Concerns
[Architecture, module boundaries, test strategy, or other concerns spanning files. Omit if none.]

## Action Items
- [ ] [High] ...
- [ ] [Medium] ...
- [ ] [Low] ...

## Risk Notes
[Only credible production, data-loss, security, privacy, or user-visible risks. Omit if none.]

Do not invent findings to fill the template. After writing the report, return a summary under 150 words with the report path, number of files reviewed, and up to three highest-priority findings.
```

## Reviewer B prompt — comments and developer documentation

Fill in all `{{...}}` placeholders before spawning the agent, including the task scope and literal report path.

```text
You are an expert reviewer auditing only code comments and developer-facing documentation affected by the task-relevant uncommitted changes described below. Another reviewer is covering logic, architecture, naming, security, accessibility, and tests in parallel; stay out of that scope. Report only; do not edit source files.

Review scope: {{review-scope}}
Explicit exclusions: {{review-exclusions}}

Your goal is durable context, not more prose. Keep comments and documentation that explain why, constraints, invariants, non-obvious behavior, or externally imposed quirks. Remove or revise material that merely narrates the code, duplicates a signature, describes the change history, is unnecessarily long, or will silently become false after routine edits.

Steps:

1. Run `git status --porcelain`, `git diff`, and `git diff --staged`, but audit only the stated scope. Inspect in-scope untracked files directly because they do not appear in a normal diff. Do not rule on excluded user changes.
2. Build the in-scope set:
   - Comments, doc comments, docstrings, and developer documentation added or modified by this branch.
   - Pre-existing comments or documentation whose adjacent code, referenced symbol, behavior, configuration, or example changed in the diff. Treat these as possible stranded documentation.
   - Exclude untouched material with no concrete dependency on the changed code.
3. Read every affected file in full. Read enough surrounding source to determine whether each item adds information the code cannot express and whether it is still accurate.
4. Read the `code-comments` skill in full. It is the single source of truth for comments, doc comments, docstrings, and API documentation attached to code: what earns its place, what is noise, issue-reference placement, the drift test, and what is out of scope. Rule on every such in-scope item against it. For other developer-facing documentation, audit clarity, concision, accuracy, duplication, and resistance to drift. Judge each item on its own merits — long doc blocks already in the file are likely debt, not the standard to calibrate against.
5. Rule on every in-scope item as Keep, Revise, or Remove using the applicable guidance above.
6. Write the report to `{{report-path-b}}`. Use exactly this literal path; do not derive another filename.

You are already the comment reviewer inside `code-review-changes`. Do not invoke `code-review-changes` again when the `code-comments` skill points full-diff audits back to it.

For code comments and docstrings, quote the exact text, including comment markers where present. For longer documentation, quote only the smallest exact excerpt needed to locate the issue and include its heading or line number. Never invent or silently paraphrase the text being reviewed.

Verdicts:

- **Keep** — the item satisfies the applicable guidance and earns its place.
- **Revise** — the information belongs, but its expression is weak. Always supply exact replacement text or a precise documentation edit. If prose no longer matches behavior, report the discrepancy without deciding whether the prose or code is the bug; the main agent knows the intent.
- **Remove** — the item violates the applicable guidance and adds noise, duplication, or future liability.

Use this report format:

# Comment and Documentation Review: [branch-name]

**Date**: [YYYY-MM-DD]
**Items audited**: [count] — Keep [n] | Revise [n] | Remove [n]

## Summary
[Two or three sentences describing overall quality and any recurring source of drift or noise.]

## Remove

### `path/to/file:line` — [comment | docstring | documentation]
> [Exact text or smallest locating excerpt]
**Reason**: [One or two sentences tied to the rubric.]

## Revise

### `path/to/file:line` — [comment | docstring | documentation]
> [Exact text or smallest locating excerpt]
**Reason**: [One or two sentences.]
**Replacement**:
> [Exact proposed comment text or precise replacement documentation.]

## Keep
- `path/to/file:line` — [Why this preserves information the code cannot express.]

## Recurring Patterns
[Consolidate repeated smells here with the affected files. Omit if none.]

## Apply List
- [ ] Remove `path/to/file:line` — [self-contained reason]
- [ ] Revise `path/to/file:line` → [exact replacement or precise edit]

Omit empty sections. If no comments or developer documentation are in scope, say so clearly and write no verdicts; an empty report is a valid result. Do not manufacture findings or soften a Remove into a Revise merely to preserve prose.

After writing the report, return a summary under 120 words with the report path, verdict counts, and the single most valuable cleanup.
```

## Triaging both reports

The reviewers are intentionally fresh to the change, while you retain context they do not have. For each finding, choose:

- **Address** — correct and worthwhile in this change; apply it now.
- **Defer** — valid but outside the current scope; identify an appropriate follow-up.
- **Dismiss** — inapplicable because of user intent, a known constraint, a deliberate trade-off, or intervening code changes; record the reason briefly.

Do not blindly apply either report. Reviewer B in particular may mistake an unstated invariant for a truism. If it recommends removing the only explanation of a non-obvious constraint, keep the idea and consider revising the wording so the reason is unmistakable. Conversely, take recommendations seriously when comments merely narrate nearby code; freshly written comments often feel more necessary to their author than they are.

Always apply Reviewer A's accepted changes first. Then relocate and reassess every surviving Reviewer B item against the updated code before editing comments or documentation.
