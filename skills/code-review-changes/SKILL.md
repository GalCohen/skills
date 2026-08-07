---
name: code-review-changes
description: Delegate a fresh-eyes review of code changes to two parallel sub-agents—one for correctness and project conventions, and one for concise, durable code comments and developer documentation—then triage and apply their findings.
disable-model-invocation: true
license: Internal
metadata:
  version: "1.1"
  category: quality
---

# Code Review Changes

You wrote this code, so you cannot review it cold. Delegate two deliberately disjoint reviews in parallel, then use your implementation context to triage their findings:

- **Reviewer A — core code review:** correctness, edge cases, architecture fit, project conventions, security, accessibility, and missing tests.
- **Reviewer B — comment and documentation hygiene:** comments and developer-facing documentation affected by the diff, with an emphasis on clarity, concision, accuracy, and resistance to drift.

Separate scopes give both reviewers enough attention to find meaningful issues without duplicating each other's work. They must also write to different report paths so parallel completion cannot overwrite a report.

## Workflow

1. **Resolve the user's OS temp directory and current branch name.** Sanitize the branch name for filenames by replacing `/` with `-`; use `detached-head` if no branch name is available.
2. **Spawn both review sub-agents before waiting for either one.** Use the prompt templates below and launch both in the same response/tool batch when the environment supports it. This concurrency is intentional.

   - If sub-agents are unavailable, invoke two independent agents through the available CLI.
   - If no delegation mechanism exists, perform both reviews yourself as separate passes and write both reports in the specified formats.

3. **Wait for both reviewers to finish.** Reviewer A writes `{{temp-dir}}/code-reviews/{{branch-name}}.md`; Reviewer B writes `{{temp-dir}}/code-reviews/{{branch-name}}-comments.md`.
4. **Read both complete report files.** Do not rely on the returned summaries alone.
5. **Triage every finding against your implementation context.** Decide whether to address, defer, or dismiss each item. Scope boundaries, explicit user decisions, intentional trade-offs, and changes made after a reviewer inspected the diff are legitimate reasons not to apply a finding.
6. **Apply accepted core-review fixes first.** Correctness changes can invalidate line numbers, comment wording, and documentation assumptions.
7. **Re-check Reviewer B's findings against the updated diff, then apply the accepted comment and documentation cleanup.** Do not apply stale verdicts mechanically.
8. **Validate the result.** Run the focused tests, lints, builds, or documentation checks appropriate to the accepted fixes. If validation cannot run, explain why.
9. **Report back to the user.** Include both report paths; findings addressed, deferred, or dismissed; brief reasons for non-addressed findings; and validation performed.

Apply accepted fixes without pausing for routine confirmation. Stop and ask only when a finding requires a product decision, expands scope, risks data loss, introduces a breaking API change, or conflicts with prior user instructions.

## Reviewer A prompt — core code review

Fill in all `{{...}}` placeholders before spawning the agent.

```text
You are an expert code reviewer. Review all uncommitted changes on the current branch and write actionable findings to a markdown report. Report only; do not edit source files.

Another reviewer is auditing comments and developer documentation in parallel. Leave wording, redundancy, clarity, and staleness to that reviewer. Mention a comment or document only when it is materially misleading about a behavioral issue you are reporting.

Steps:

1. Run `git status --porcelain`, `git diff`, and `git diff --staged` so modified, staged, and untracked files are all considered. Inspect untracked source files directly because they do not appear in a normal diff.
2. Read every affected source file in full; the diff alone can hide important surrounding context.
3. Read the repository's applicable `AGENTS.md` files and any directly relevant project guidance or conventions they reference.
4. Review the changes as a coherent solution, not only as isolated lines.
5. Write the report to `{{temp-dir}}/code-reviews/{{branch-name}}.md`. Create the directory if needed and overwrite an existing report for this branch because the latest diff is authoritative.

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

Fill in all `{{...}}` placeholders before spawning the agent.

```text
You are an expert reviewer auditing only code comments and developer-facing documentation affected by the uncommitted changes on the current branch. Another reviewer is covering logic, architecture, naming, security, accessibility, and tests in parallel; stay out of that scope. Report only; do not edit source files.

Your goal is durable context, not more prose. Keep comments and documentation that explain why, constraints, invariants, non-obvious behavior, or externally imposed quirks. Remove or revise material that merely narrates the code, duplicates a signature, describes the change history, is unnecessarily long, or will silently become false after routine edits.

Steps:

1. Run `git status --porcelain`, `git diff`, and `git diff --staged`. Inspect untracked files directly because they do not appear in a normal diff.
2. Build the in-scope set:
   - Comments, doc comments, docstrings, and developer documentation added or modified by this branch.
   - Pre-existing comments or documentation whose adjacent code, referenced symbol, behavior, configuration, or example changed in the diff. Treat these as possible stranded documentation.
   - Exclude untouched material with no concrete dependency on the changed code.
3. Read every affected file in full. Read enough surrounding source to determine whether each item adds information the code cannot express and whether it is still accurate.
4. Rule on every in-scope item as Keep, Revise, or Remove using the rubric below.
5. Write the report to `{{temp-dir}}/code-reviews/{{branch-name}}-comments.md`. Create the directory if needed and overwrite an existing report for this branch.

For code comments and docstrings, quote the exact text, including comment markers where present. For longer documentation, quote only the smallest exact excerpt needed to locate the issue and include its heading or line number. Never invent or silently paraphrase the text being reviewed.

### Keep

Keep material that supplies durable information the implementation cannot readily express:

- Why this approach is necessary or why an obvious alternative is wrong
- An invariant, ordering rule, unit, threading constraint, or cross-call-site contract
- A non-obvious edge case or domain rule
- An external platform, protocol, dependency, or compatibility workaround, ideally anchored to a version or issue
- A warning that prevents a plausible future edit from breaking behavior
- Public API guidance that callers genuinely need and cannot infer from the signature and types

### Revise

Revise when the underlying information is valuable but its expression is weak. Always provide exact replacement text or a precise documentation edit.

- Replace a description of what the code does with the reason or constraint that matters
- Compress bloated prose while preserving essential context
- Remove brittle anchors such as line numbers, positional phrases, duplicated constants, counts, or symbol names that a routine edit can orphan
- Correct documentation that no longer matches behavior
- Anchor TODO or FIXME notes to a stable issue, owner, or removal condition; otherwise recommend removal
- Prefer an assertion, type, named abstraction, test, or clearer API when it can enforce the claim better than prose; explain that the comment should disappear with that code change

### Remove

Remove material that adds noise or future liability:

- Restates or walks through self-explanatory code
- Re-spells a function, property, parameter, or type signature without adding a contract or constraint
- Narrates file structure or obvious control flow
- Describes the change rather than the enduring behavior; version control already records history
- Contains commented-out code
- Duplicates a test name or stable documentation elsewhere
- Is generic template, scaffolding, or AI-filler prose
- Is obsolete, unresolvable, or so brittle that it is more likely to mislead than help

### Drift test

For every Keep or Revise candidate, ask whether a symbol rename, new enum case, changed constant, reordered block, altered default, or fixed upstream bug could make it false without drawing an editor's attention. If so, re-anchor it to the actual dependency or recommend removing it. Concise is not enough; the text must also resist silent drift.

### Out of scope

Do not rule on license headers, generated-file banners, formatter or linter directives, build-configuration directives, navigation markers, user-facing copy, or documentation unrelated to changed behavior.

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
