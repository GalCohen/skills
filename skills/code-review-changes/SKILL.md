---
name: code-review-changes
description: Delegate a fresh-eyes code review of code changes.
disable-model-invocation: true
license: Internal
metadata:
  category: quality
---

# Code Review Changes

You wrote this code, so you can't review it cold — delegate, then triage. Here is the task: delegate review of uncommitted changes to a sub-agent, then triage its findings against your own implementation context.

## Workflow

1. **Spawn the review sub-agent**. Pass the prompt template in the next section.

- If you cannot spawn agents, you have two options:
  - use the CLI to invoke an agent with the provided prompt template, filling in the placeholders manually.
  - manually run the steps in the sub-agent prompt template yourself, then write a report in the same format.

2. **Wait for it to finish.** It will write its report to `<user's temp directory>/code-reviews/[branch-name].md` and return a short summary.
3. **Read the report file.** Don't rely on the sub-agent's summary alone — open the actual markdown file.
4. **Triage the findings against your implementation context.** For each item, decide: address it, defer it, or dismiss it (with reason). Items the sub-agent couldn't know about — scope boundaries, explicit user decisions, intentional trade-offs — are legitimate dismissals.
5. **Apply the accepted fixes immediately.** Do not pause for user confirmation after triage. The implementor owns the decision about what to address. Only stop and ask the user if a finding requires a product decision, changes scope, risks data loss, introduces a breaking API change, or conflicts with prior user instructions.
6. **Validate the result.** Run the focused tests/lints/build checks appropriate to the accepted fixes. If validation cannot run, say why.
7. **Report back to the user.** Include the review report path, which findings you addressed, which findings you deferred or dismissed with brief reasons, and validation performed.

## Sub-agent prompt template

Spawn an agent with this prompt. Fill in the placeholders (`{{...}}`) before sending.

```
You are an expert code reviewer. Review all code changes and write findings to a markdown file in the user's OS temp directory.

Steps:

1. Run `git status --porcelain` to list modified and new files. Also run `git diff` and `git diff --staged` to see actual changes, not just whole-file contents.
2. For each modified file, examine the full file (the diff alone can hide context about surrounding code).
3. Run `git branch --show-current` to get the branch name for the output filename.
4. Evaluate the changes against the project's standards:
   - `AGENTS.md` at the repo root — project overview, module layout, development guidelines, etc..
5. Write the report to `<user's temp directory>/code-reviews/{{branch-name}}.md` using the template below. Create the directory if it doesn't exist. Sanitize the branch name for filesystem use (replace `/` with `-`). If a report already exists for the current branch, overwrite it — the latest diff is what matters.

Focus on things a human reviewer would actually raise:
- Is this the right solution to the problem?
- Correctness bugs and edge cases (nil handling, off-by-one, concurrency races, memory leaks, retain cycles)
- Violations of project conventions (wrong module, wrong architecture layer, bypassing shared infrastructure)
- Missing tests for new business logic
- Public API exposure that should be internal
- Accessibility regressions in new UI
- Security issues (logged secrets, unsafe deserialization, insecure defaults)

Avoid:
- Style nits that a formatter or linter would catch (unless the project doesn't have those tools configured)
- Speculative "you could also do X" refactors with no concrete benefit
- Praise-only comments — strengths are fine to note briefly, but the point of the report is actionable feedback
- Repeating the same finding across many files — consolidate into one entry with a file list

Report format:

# Code Review: [branch-name]

**Date**: [YYYY-MM-DD]
**Reviewed Files**: [count]

## Executive Summary
[2-3 sentences: what the change appears to do, and overall assessment]

## File-by-File Analysis

### `path/to/File.swift`
**Status**: New | Modified
**Risk**: Low | Medium | High

- Finding with `file:line` reference and concrete suggestion
- Next finding

[Repeat per file. Skip files with no findings rather than padding.]

## Cross-Cutting Concerns
[Issues that span multiple files — architectural fit, module boundaries, missing tests, etc. Omit if none.]

## Action Items
Ranked by priority. Each item should be specific enough to act on without re-reading the diff.

- [ ] [High] ...
- [ ] [Medium] ...
- [ ] [Low] ...

## Risk Notes
[Anything that could cause a production incident, data loss, security issue, or user-visible regression. Omit if none — do not invent risks.]

After writing the file, return a short summary (under 150 words): the report path, file count reviewed, and the top 3 findings by priority.
```

## Triaging the sub-agent's findings

When you read the report, you'll know things the sub-agent can't:

- **Scope**: the user asked for X; the sub-agent may flag missing Y that is intentionally out of scope.
- **Prior decisions**: the user already weighed in on a trade-off earlier in the conversation.
- **Intentional departures**: the code deviates from a pattern for a specific reason (legacy constraint, experimental flag, etc.).
- **Staleness**: the sub-agent may flag something you've already fixed since it last ran.

For each finding, pick one:

- **Address** — the sub-agent is right; fix it now in this same turn.
- **Defer** — valid but out of scope for this change; note it for a follow-up (or surface to the user).
- **Dismiss** — not applicable here; briefly record why in your reply to the user so they can push back if they disagree.

Do not blindly apply every finding. You decide which ones are correct and worth acting on.

After triage, proceed directly to implementation for all **Address** items. Do not ask for confirmation unless the fix requires a new product or scope decision.
