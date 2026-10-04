---
name: skill-drift-sweep
description: "Check one agent skill against the codebase in both directions: verify that its claims still hold and that it still describes how the code is actually written. Use only when explicitly invoked or by a scheduled maintenance routine."
disable-model-invocation: true
license: Internal
metadata:
  version: "1.0"
  category: quality
---

# Skill Drift Sweep

Skills go stale quietly. The code moves, the skill keeps saying the old thing, and every agent that
reads it inherits the error — so a single wrong line compounds across every task that loads it.

Pick one skill per run and verify it against the codebase as it stands today. If the project uses an
issue tracker and the invocation names an assignee, assign any issue created by this workflow to
them. Otherwise, leave it unassigned.

## 1. Pick the skill

The rotation covers every first-party skill. Skills an agent loads automatically (without
`disable-model-invocation: true` in frontmatter) come up twice per cycle, because a wrong line in
them propagates into every task in their area; explicit skills come up once.

Use the current worktree or the latest accessible default-branch state. Do not switch branches,
pull, or otherwise disturb a shared worktree merely to select a skill. Selection is deterministic,
with no state between runs. In this repository, let the week number choose:

```bash
pool() {
  for skill in skills/*/SKILL.md; do
    dir=${skill%/SKILL.md}
    { [ -f "$dir/LICENSE" ] || [ -f "$dir/LICENSE.md" ]; } && continue  # vendored
    [ "$1" = auto ] && sed -n '2,/^---$/p' "$skill" | grep -q '^disable-model-invocation: *true' && continue
    basename "$dir"
  done | sort
}

# Every skill once, then the auto-invoking ones a second time.
{ pool all; pool auto; } | awk -v week=$(( $(date +%s) / 604800 )) '{a[NR]=$0} END {print a[(week % NR) + 1]}'
```

The counter is weeks since the Unix epoch, not the ISO week of the year, so the rotation does not
reset at a year boundary. If the invocation names a skill, use it instead of the rotation.

If an open PR or, when the project uses an issue tracker, an active issue from an earlier run already
covers the selected skill, take the next name in the list instead. State which skill you picked and
why before you start.

## 2. Verify it

Read the skill in full, including any relevant files under `references/`.

**Never invoke the skill under review, and never run the commands it documents.** Invoking it does
the job it describes and may trigger builds, mutations, or external work. Check its claims with your
own read-only queries against the codebase instead.

Then check it in both directions. A skill can be wrong in what it says, and just as wrong in what it
leaves out — the first pass treats each claim as a hypothesis to test; the second asks what the skill
never told you.

### Does everything the skill says still hold?

Take each claim in rough order of value:

1. **Code examples.** Compare them with the real declarations: does the type exist, is the method's
   name correct, are the arguments right, and does the example make valid assumptions about results?
   This is the costliest drift, because a wrong example gets copied verbatim.
2. **Named symbols.** Types, interfaces, properties, mock names, packages, and module homes.
   Confirm each is declared, spelled exactly as the skill spells it, and still lives where the skill
   says it lives.
3. **Rules and conventions.** Where the skill says "always X" or "never Y", check what the codebase
   actually does. A rule the code has abandoned is drift even though nothing is misspelled.
4. **Structure and commands.** File paths, directory layouts, package or target names, and documented
   commands. Verify them by reading the relevant scripts and configuration, rather than running them.
5. **Cross-references.** Pointers to other skills, repository guidance such as `AGENTS.md`, and the
   skill's own reference files.

### Would following the skill produce code the team actually writes?

The other direction catches what no claim-by-claim pass can: read how the codebase does this thing
today, then ask whether the skill would have led you there.

Weight recent work most heavily. Inspect recent changes to relevant paths and read the newest
instances of the pattern. Then look for:

- **what the skill never mentions** — a parameter its examples never pass, a helper every call site
  uses, a step every recent instance performs, or a mock or fixture that has become standard;
- **what the skill documents but the code has left behind** — if it teaches one pattern and the code
  overwhelmingly does another, the skill is describing a minority; count both before saying so; and
- **where the skill's advice would now stand out** — if code written strictly to this skill would
  look unlike its neighbours in review, something has moved.

**A gap is a question, not an answer.** Do not write a convention into the skill merely because the
code does it: the code may have drifted, or spread something nobody endorsed, and documenting it
would bless it and push it into every future task that loads the skill. Nor is an omission
necessarily an oversight — it may be deliberate. Findings of this kind go to a person, not into a
diff.

Prefer evidence over inference: a claim is confirmed by a declaration you have read, or by a
read-only query of your own, not by a plausible-looking search result.

### Most mismatches are not drift

A naive symbol or path check is dominated by false positives. Before reporting anything, rule out:

- platform SDK and standard-library names;
- deliberate placeholders and illustrative names;
- prose that happens to be capitalized;
- names that were never owned by the project, such as tool names or third-party APIs; and
- examples that are simplified without being wrong.

Verify each surviving candidate against the code before it reaches the report. **A skill that comes
through both passes intact is a successful run** — say so plainly and stop.

## 3. Decide which side is wrong

A mismatch tells you the two are out of step. It does not tell you which one should move:

- **the skill is stale** — the code moved on and nobody updated the documentation. Fix the skill.
- **the code drifted** — the skill still describes the convention the team wants and the code stopped
  following it. Do not edit the skill to match the code. If the project uses an issue tracker, file
  an issue describing the violation; otherwise report it and leave the skill alone.
- **the skill is silent** — the code has a convention the skill never mentions. Whether it belongs
  in the skill depends on whether the convention is wanted, and the code cannot tell you that. Put
  the question to the assignee.
- **genuinely ambiguous** — the skill documents an intent the code never fully adopted. Defer rather
  than guess.

Deferring is a real outcome, not a failure to finish. You can see what the code does; you usually
cannot see what the team meant by it, and the difference lives with the developer rather than in the
repository. Where the two come apart, write up what you found, both readings, and what each would
cost — then stop and let them choose.

## 4. Ship it

**If the skill was stale**, fix it:

1. Make the corrections.
2. Re-verify each one against the code.
3. If the project uses an issue tracker, create an issue for the work and apply relevant team labels
   when they exist; do not invent labels.
4. Open a pull request containing only the corrections, linking the issue when one exists. Create
   it only when the invocation authorizes the repository's normal external workflow and the agent
   has the required access. Do not merge it.

In the PR body, report **coverage as well as hits**: what you checked and found sound, what you
found wrong, and what you deliberately did not check — including any command you read rather than
ran. A reviewer cannot judge a documentation diff without knowing how much of the skill was
exercised.

**If the code drifted, the skill is silent, or the call is ambiguous**, there is nothing to correct
and no PR to open. If the project uses an issue tracker, file an issue that states the mismatch,
both readings of it, and what changing each side would cost — then stop. Otherwise, report the same
information and the required human decision.

**If some findings are clear-cut and others are not**, keep them apart: the corrections are the pull
request, and the judgment calls are a separate issue or report.

## Final report

Report the selected skill and why it was chosen, the scope checked and deliberately not checked,
confirmed claims, corrections made, and any deferred decisions. Include issue and PR links when
created. State plainly when the skill was verified intact or when missing access prevented the
normal issue or PR workflow.
