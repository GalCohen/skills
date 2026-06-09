---
name: handoff
description: Create a handoff document summarizing work done, what remains, key decisions, known issues, and how to continue. Use whenever the user is wrapping up a session, passing work to another developer or agent, or says things like "write a handoff," "document where we left off," "prepare a context dump," "I'm handing this off," or "summarize what we've done so I can continue later" — even if they never say the word "handoff."
---

The point of a handoff is to let whoever picks this up next — a human or another agent — continue without having to ask you questions or reverse-engineer the current state. They weren't here for the work, so the document has to carry everything they'd otherwise have to reconstruct.

## Ground the handoff in reality first

Before writing, check the actual state rather than relying on memory:

- `git status`, `git diff`, and recent `git log` — what's committed, what's staged, what's still dirty, and on which branch.
- The current todo list or task tracker, if there is one.
- Whether tests/build pass right now, and if not, what's failing.

Describe what you actually find. If something is uncertain or you didn't verify it, say so plainly instead of papering over the gap.

## What to include

Use this structure as a default skeleton — adapt it to the work, but keep it skimmable:

```markdown
# Handoff: [task / feature name]

## Summary

What this work is and where it stands in one short paragraph.

## Current state

- What's implemented and working
- What's in progress (and how far along)
- Current branch, and any uncommitted or unpushed changes

## What remains

The concrete next steps, ideally in priority order.

## Known issues & gotchas

Bugs, edge cases, fragile spots, and decisions that aren't obvious from the code.

## How to continue

The specific files/functions/modules to start in, plus how to run, build, and test.

## References

Links to relevant docs, tickets, PRs, or discussions.
```

A few principles behind the structure:

- **Explain the _why_ behind non-obvious decisions.** The next person can read the code; what they can't recover is why you chose this approach over the alternative, or why a tempting shortcut doesn't work.
- **Link, don't duplicate.** If something is already documented elsewhere (a design doc, a ticket, a README), point to it instead of restating it — restated docs drift out of sync.
- **Be concrete about where to start.** "Continue the auth work" is useless; "pick up in `auth/session.ts:120` where token refresh is stubbed" lets them start immediately.

This isn't an exhaustive checklist — use judgment about what this particular handoff needs. The test is simple: could someone with no memory of this work continue from the document alone?

## Where to save it

Default to a markdown file in the repo in `docs/handoffs/<date>.md`. Report the file path when you're done.
