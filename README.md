# skills

My personal collection of agent skills

## Installation

- skills.sh:
```
npx skills add https://github.com/GalCohen/skills
```

- or copy & paste into your `./claude/skills/`,  `./agents/skills/`, etc.

## Current Skills

### Implementation Complete

- Invoke manually with `implementation-complete`
- A mandatory completion gate that discovers and runs the project's own review, build, static-check, test, documentation, cleanup, and runtime validation workflow before an agent declares implementation work done.
- Triggers automatically when implementation work is being finalized. If the task's changes have not been reviewed, it invokes `code-review-changes` once before validating the settled result.

### Code Review Changes

- Invoke manually with `code-review-changes`
- Runs a fresh-eyes review of task-relevant changes, then has the implementation agent triage the findings using its original context and apply the accepted fixes.
- Triggers proactively once the implementation is believed working and before the completion gate, while enforcing a one-review budget that prevents recursive review/fix loops.

### Hand Off

- Invoke manually with `handoff`
- Generates a doc describing all the work so far, findings, next steps, etc. that can be handed off to a human or agent and saves it locally so that it can be committed to version control. Perfect capturing everything after a long debugging or research session where you want just the right context to persist and aren't sure the regular context compaction will do. Plus, this way you can view the file, edit it, commit it, etc.
- Triggered automatically when telling the agent to hand off / pass along everthing to another agent.

### Complete Pull Request

- Invoke manually with `complete-pull-request` or provide a PR and ask the agent to finish, finalize, revive, unblock, or shepherd it.
- Independently validates that the problem is still real and the proposed direction is correct before changing the PR. If valid, it updates the branch with its base, resolves conflicts and necessary feedback, fixes relevant CI issues, runs the project validation, and reports whether the PR is genuinely merge-ready.
- Stops before mutation when the PR premise is obsolete, the approach is wrong, or a product/security/scope decision is required.
