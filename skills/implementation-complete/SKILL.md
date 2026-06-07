---
name: implementation-complete
description: Implementation completion checklist for a software project. ALWAYS invoke before declaring "implementation complete", "all tests passing", "done", or "ready for next phase". Also trigger when the user asks to verify, validate, or finalize any implementation, when wrapping up any coding task that modified shared interfaces, types, schemas, or public APIs, or when entirely finishing any coding task and reaching the point of deciding what to do next — even if the user hasn't explicitly asked for validation. This is a required quality gate — do not skip it.
---

# Implementation Complete Checklist

This is a mandatory quality gate. Run through every section below before declaring any task complete. Do not treat this as a formality — its purpose is catching problems that feel done but aren't.

## 0. Discover the Project's Toolchain First

Do not assume `npm`, `bun`, `make`, or any specific tool. Identify how this project
builds, tests, lints, and type-checks by inspecting it:

- `AGENTS.md` or similar agent/skill files may have explicit instructions for validation commands. Follow those if they exist.
- **Manifest / scripts** such as a package.json`(scripts),`Makefile`, `Cargo.toml`, etc.
- **CI config**: `.github/workflows/`, `.gitlab-ci.yml`, `.circleci/`, etc.
- **Contributor docs**: `README`, `CONTRIBUTING.md`, etc.

Map each checklist item below to a concrete command for this project. If a category has no
tooling (e.g. no linter configured), note that and move on — don't invent or install one.

## 1. Orphaned Assets & Files Cleanup

If you removed or renamed any views, screens, pages, or components, check for orphaned resources that are no longer referenced:

- Image, colors, fonts assets that were only used by removed views.
- Localization strings that are no longer referenced.
- Any other supporting files (JSON configs, yaml, plists) tied to removed functionality.
- Search the codebase (using grep, Find Usages, or similar) for the asset/file name across the codebase to confirm it has zero remaining references before deleting.

## 2. Linting & Formatting

- No **new** lint errors. Pre-existing warnings are acceptable; don't expand scope fixing
  unrelated ones unless asked.
- Formatting to match project standards.

## 3. Build / Compile Verification

- The project must build cleanly with no new errors anywhere — not just in files you edited.
- In a modular codebase, compiling just the files you touched isn't enough — changes to shared types or interfaces can break distant parts of the code that still compile but fail at runtime. A full build is required to catch these issues.

## 4. Full Test Suite

- Run the **entire** suite, not just tests for the code you touched. Shared/cross-cutting
  code (config, auth, serialization, shared utilities) can be broken by unrelated changes,
  so partial runs create false confidence.
- No previously-passing test may now be failing.
- Include integration / end-to-end tests if the project has them — unit tests alone don't
  catch wiring failures between components.

## 5. Breaking Changes Protocol

**If you modified any shared interface, type, schema, public function signature, or API
contract:**

- Search the **entire** codebase for all usages — not only the files that fail to build.
- Update every affected site: callers, implementations, tests, mocks/factories, fixtures,
  serialized data, and documentation.
- Critically, find **indirect** usages. Grep for the symbol name,
  string keys, and reflection/serialization references.
- Re-run the full test suite after the updates to catch integration-level breakage.

> A change to a widely-used type can cascade to many files that all still compile but carry
> subtle runtime bugs. The codebase-wide search is what catches those silent failures —
> "it compiles" is not "it works."

## 6. Coverage (if the project tracks it)

If the project has a coverage threshold (check CI config or the test runner config):

- Run coverage separately — it's usually slower than the plain test run.
- Meet or exceed the project's threshold; don't let coverage decrease for code you didn't
  touch.
- For safety-critical or high-stakes systems, treat untested branches as latent production
  failures, not as acceptable gaps.

If the project has no coverage tooling configured, skip this — don't add it unprompted.

## 7. Documentation

- Update relevant docs (`README`, `docs/`, API references, changelog) when behavior or
  public APIs change.
- Keep agent-facing docs (`CLAUDE.md`, `.cursorrules`, etc.) accurate — stale guidance
  causes future agents to make wrong assumptions.
- Add comments only for non-obvious decisions; match the surrounding code's comment density.

## 8. Final Declaration

The point of this gate is catching work that _feels_ done but isn't — don't treat it as a
formality.

**Only after every applicable section above passes** may you declare:

- "Implementation complete"
- "All tests passing"
- "Ready for next phase"

**If any check fails, DO NOT declare completion. Instead:**

1. List every failure.
2. Fix them systematically.
3. Re-run all validation commands.
4. Only then declare completion.

Report outcomes faithfully: if a step was skipped or had no tooling, say so explicitly
rather than implying it passed.

---

**The one rule that's always true:** prefer the full test suite over individual feature
tests, and prefer the project's own CI commands over anything you improvise.
