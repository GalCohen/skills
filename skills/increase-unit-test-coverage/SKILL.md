---
name: increase-unit-test-coverage
description: Make one small, high-value increase in unit-test coverage. Use only when explicitly invoked or by a scheduled maintenance routine; do not use for ordinary feature or bug-fix testing.
disable-model-invocation: true
license: Internal
metadata:
  version: "1.0"
  category: testing
---

# Increase Unit Test Coverage

Make a single, focused, reviewable improvement to unit-test coverage. The goal of each run is to
increase confidence in a meaningful part of the codebase, not to maximize a percentage with
low-value tests.

If the project uses an issue tracker and the invocation identifies an assignee, assign any issue
created by this workflow to them. Otherwise, leave it unassigned.

## 1. Establish coverage evidence

Read repository guidance and discover the existing test and coverage workflow from its manifests,
CI configuration, contributor documentation, and test tooling. Run the broadest practical
coverage-enabled test command already supported by the project. Reuse an existing coverage report
when it is current enough to represent the code under examination.

Record the applicable before figures at the project, package, module, file, or function level. Do
not add coverage tooling, alter thresholds, or treat an incomplete report as a full-codebase
measurement just to run this skill.

## 2. Select one worthwhile gap

Choose one small target that has meaningful untested behavior. Favor code that is broadly reused,
has a material coverage gap, protects important data or business behavior, contains meaningful
branching or error handling, or is widely depended on.

Before selecting it, search tests across the whole repository. Coverage for a package or target
can miss tests that exercise the code indirectly from another package, application, integration,
or end-to-end test. Do not choose code solely because a narrow report labels it uncovered.

Skip targets where the remaining gap is primarily:

- presentation-only code that is better covered by UI or end-to-end tests;
- generated code, mocks, fixtures, trivial serialization/deserialization, or boilerplate;
- dead or obsolete code; or
- behavior with no useful test seam unless creating one would be a disproportionate refactor.

## 3. Avoid duplicate work and stop at decision boundaries

Before changing code, check open pull requests and, if the project uses an issue tracker, active
issues for work that modifies the same area or already covers the identified behavior. Select
another target if the work would duplicate an active effort.

Read the implementation and its callers closely enough to establish the expected behavior. If the
work reveals a likely defect or requires a human product, design, or architecture decision, stop
the coverage run. Do not silently encode the behavior as a test or fix it as part of this run. If
the project uses an issue tracker, file an issue with the evidence; otherwise call out the blocker
and ask for direction. Do not open a coverage PR.

## 4. Add a focused test

Add the smallest useful set of unit tests that demonstrates existing, intended behavior. Keep
production changes limited to testability improvements that are necessary and proportionate.
Follow the repository's test conventions, including its naming, fixtures, test doubles, and
assertion style.

The change must remain easy to review. Do not broaden it into a refactor, retrofit unrelated code,
or add superficial assertions that only exercise lines without guarding behavior.

## 5. Verify the result

Run the focused tests and the relevant project validation. Re-run the coverage workflow when
practical to capture the after measurement; otherwise state precisely why the before/after metric
cannot be compared. Review the final diff for accidental changes and confirm the tests fail for a
meaningful regression when the project makes that practical.

Use the repository's completion workflow, including any installed completion or review skill,
before declaring the change ready.

## 6. Track and propose the work

If the project uses an issue tracker, create an issue linking the coverage gap, target, rationale,
validation, and before/after figures. Apply the team's relevant improvement and area labels when
they exist; do not invent labels. Use the project's established issue workflow, such as Linear,
Jira, or GitHub Issues. Link the issue from a pull request containing only this coverage
improvement.

Push a branch and open a PR, linking the issue when one exists, unless the run was explicitly told
not to or lacks GitHub access. Do not merge the PR.

## Final report

Report the selected target and why it was valuable, the test behavior added, coverage evidence,
validation performed, and links to the issue and PR when created. If the run stopped because
of a defect, a required human decision, duplicate work, or unavailable coverage or tracker access,
say so plainly and identify the next safe action.
