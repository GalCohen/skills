# skills

My personal collection of agent skills

## Current Skills

### Implementation Complete

- Invoke manually with `implementation-complete`
- A quality gate for agents to work through when they finish their work. (build the app, run the tests, remove unused files, etc.)
- Triggers automatically as needed.

### Code Review Changes

- Invoke manually with `code-review-changes`
- Has the main implementation agent spawn a sub-agent to review and report back its findings. Then the main implementation agent decides which feedback to address. This way the implementer gets a fresh, unbiased review, while still maintaining the implementation context to push back or apply the correct fixes as needed.
- Triggred manually only.
