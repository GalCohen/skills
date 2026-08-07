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
- A quality gate for agents to work through when they finish their work. (build the app, run the tests, remove unused files, etc.)
- Triggers automatically as needed.

### Code Review Changes

- Invoke manually with `code-review-changes`
- Has the main implementation agent spawn two sub-agents in parallel: one reviews correctness and project conventions, while the other audits comments and developer documentation for clarity, concision, accuracy, and resistance to drift. The implementation agent then triages both reports using its original context and applies the accepted fixes.
- Triggred manually only.

### Hand Off

- Invoke manually with `handoff`
- Generates a doc describing all the work so far, findings, next steps, etc. that can be handed off to a human or agent and saves it locally so that it can be committed to version control. Perfect capturing everything after a long debugging or research session where you want just the right context to persist and aren't sure the regular context compaction will do. Plus, this way you can view the file, edit it, commit it, etc.
- Triggered automatically when telling the agent to hand off / pass along everthing to another agent.

### Complete Pull Request

- Invoke manually with `complete-pull-request` or provide a PR and ask the agent to finish, finalize, revive, unblock, or shepherd it.
- Independently validates that the problem is still real and the proposed direction is correct before changing the PR. If valid, it updates the branch with its base, resolves conflicts and necessary feedback, fixes relevant CI issues, runs the project validation, and reports whether the PR is genuinely merge-ready.
- Stops before mutation when the PR premise is obsolete, the approach is wrong, or a product/security/scope decision is required.
