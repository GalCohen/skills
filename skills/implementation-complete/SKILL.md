---
name: implementation-complete
description: Mandatory completion gate for software implementation work. ALWAYS invoke before declaring an implementation done, complete, ready for review, ready for the next phase, or fully validated; when the user asks to verify, validate, finalize, or wrap up code changes; and when finishing any coding task, even without an explicit validation request. If task-relevant uncommitted changes have not received a fresh-eyes review in the current task, invoke code-review-changes once before validation, then resume this gate. Do not use for research, diagnosis, planning, or review-only tasks that made no implementation changes.
license: Internal
metadata:
  version: "1.1"
  category: quality
---

# Implementation Complete Checklist

This is the final owner of implementation completion. Its job is to settle review findings first, validate the resulting code once it is stable, and report exactly what is and is not proven.

Use this one-way lifecycle:

`implementation → code-review-changes (once, if needed) → validation → final declaration`

Do not turn it into `review → fix → review` or `validation → fix → review`. Review and validation fixes remain inside the same completion cycle.

## 0. Establish state and discover the toolchain

Before running commands:

1. Define the task-relevant change set. Use the conversation, `git status`, staged and unstaged diffs, and untracked files. Preserve unrelated pre-existing user changes and exclude them from review and cleanup.
2. Determine whether `code-review-changes` has already completed for this task in the current conversation. Do not infer this from temporary report files or timestamps.
3. Discover the repository's actual validation workflow. Read applicable `AGENTS.md` or equivalent instructions, package and build manifests, CI configuration, and contributor documentation.
4. Map the applicable sections below to concrete project commands. Prefer the project's CI-equivalent or documented commands. Do not assume a language, framework, package manager, or tool, and do not install missing tooling merely to satisfy this checklist.

Record checks that are inapplicable, unavailable, prohibitively environment-dependent, or blocked by a known pre-existing failure. A skipped check is not a passed check.

## 1. Fresh-eyes review precondition

If the task-relevant uncommitted changes have not been reviewed in this task, invoke `code-review-changes` now and follow it through both reports, triage, and accepted fixes. Resume at section 2 when it returns.

Skip this precondition when:

- the diff is trivial, such as formatting-only changes, a pure rename with no behavioral effect, or a clean revert;
- the user explicitly declined review;
- no implementation changes were made; or
- the current task already completed the review workflow.

One completed review satisfies this precondition for the task. Edits made while applying accepted findings or fixing later validation failures do not reset it. Never invoke `code-review-changes` recursively or automatically for a second pass. A materially new implementation or an explicit user request may warrant another review, but that is a separate deliberate decision outside this completion cycle.

## 2. Change-set completeness and cleanup

Inspect the settled diff as a coherent solution:

- Confirm the requested behavior is implemented and task-relevant new files are included.
- Look for debug logging, temporary scaffolding, placeholder data, commented-out code, accidental generated output, and secrets.
- If files, components, screens, commands, or features were removed or renamed, search before deleting supporting assets, localization entries, configuration, fixtures, or documentation that may now be orphaned.
- Do not delete unrelated or uncertain files. Escalate when ownership or intended compatibility is unclear.

## 3. Shared contracts, dependencies, and boundaries

When the change affects a shared interface, public API, type, schema, serialization contract, configuration key, database shape, protocol, event, or command-line surface:

- Search the entire repository for direct and indirect consumers, including callers, implementations, tests, mocks, factories, fixtures, generated bindings, examples, and documentation.
- Check compatibility and migration behavior, not only compilation. String keys, reflection, persistence, and wire formats can fail without a compiler error.
- Verify new files belong to the correct target, package, workspace, or build graph.
- Verify new dependencies are declared in the appropriate manifest or lockfile and respect repository architecture boundaries.
- Keep visibility as narrow as the actual cross-module contract permits.

## 4. Documentation and durable comments

- Update user, contributor, API, configuration, migration, and changelog documentation when the changed behavior requires it.
- Keep agent-facing project guidance accurate.
- Ensure examples and commands still work when the project provides a way to check them.
- Preserve comments that explain non-obvious constraints or decisions; avoid comments that merely narrate code. The comment reviewer handles detailed hygiene, so do not repeat that entire audit here.

## 5. Static checks and formatting

Run every applicable project-provided formatter check, linter, type checker, static analyzer, schema validator, code generator consistency check, or equivalent CI step.

- Do not introduce new violations.
- Do not broaden scope to clean unrelated pre-existing warnings unless required for a meaningful result or requested by the user.
- If a formatter or generator modifies files, inspect the resulting diff and rerun the checks it can affect.

## 6. Build or compile verification

Run the broadest practical project build or compile command required by repository guidance or CI. For monorepos and modular projects, include affected dependents rather than compiling only edited files.

If a full build is unavailable in the current environment, run the strongest supported substitute and record the limitation. Do not describe an unrun build as successful.

## 7. Tests

Run the project's full test suite when it is available and permitted. A focused test run is useful for fast feedback but does not replace the full suite at completion.

- Include configured integration, end-to-end, snapshot, contract, migration, or platform tests when repository guidance or CI treats them as part of completion.
- Verify new or changed behavior has appropriate tests. Do not manufacture low-value tests merely to satisfy a checkbox.
- If the project tracks coverage, run the configured coverage check and meet its threshold. Do not add coverage tooling unprompted.
- Distinguish failures caused by the task from confirmed pre-existing or environmental failures. Investigate enough to support that distinction.

## 8. Runtime or artifact checks

Perform applicable checks that static validation cannot cover, such as launching the changed application, exercising a CLI path, rendering changed UI, validating a package or generated artifact, checking migrations, or running a documented smoke test.

Use the repository's own workflow and tools. Skip this section when no meaningful runtime or artifact check applies, and say so.

## 9. Fix-and-rerun rule

When any section reveals a task-related defect:

1. Fix it without restarting the review workflow.
2. Rerun the failed check.
3. Rerun any later or dependent checks the fix could invalidate.
4. Inspect the final task-relevant diff once more for accidental changes.

Continue until every applicable check passes or a genuine blocker remains. Stop for user input only when resolution requires a product decision, materially expands scope, risks data loss, creates a breaking contract, needs unavailable credentials or infrastructure, or conflicts with prior instructions.

## 10. Final declaration

Only declare the implementation complete when:

- the review precondition was satisfied or legitimately skipped;
- every applicable, runnable validation passed after the final edits; and
- no known task-related defect or unresolved required decision remains.

Report:

- review status, including why it was skipped if applicable;
- checks run and their outcomes;
- checks not run and why;
- confirmed pre-existing or environmental failures;
- any deferred or dismissed review findings; and
- remaining risk or recommended manual verification.

Do not say "all tests passing" unless the full applicable suite actually ran and passed. When required validation is unavailable or blocked, state that the implementation is not fully verified rather than converting missing evidence into success. When the remaining issue is known and external to the task, describe the work as completed with that explicit validation limitation instead of making an unqualified claim.

The invariant is simple: review the implementation before comprehensive validation, validate the settled result, and never let either phase recursively restart the other.
