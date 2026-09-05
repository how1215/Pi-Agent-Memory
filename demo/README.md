# End-to-end demonstration

This demonstration shows a project convention surviving a complete agent
restart. It exercises the same capture, persistence, retrieval, and injection
path used by the Pi adapter.

## Session A: capture a convention

Start Pi with the extension:

```bash
export PI_MEMORY_PATH="$PWD/.pi-memory.json"
pi -e "$PWD/pi-bridge/extension.ts"
```

Tell the agent:

```text
Please remember that this project uses pnpm, not npm, and that the test command
is pnpm test.
```

The agent calls `remember`; the adapter invokes `pi-memory capture`; and the
result is persisted as a JSON observation. Exit Pi completely after the tool
call finishes.

## Session B: retrieve without repeating the answer

Launch the same command again to create a fresh session, then ask:

```text
How do I run the tests for this project?
```

Before the agent starts, the adapter retrieves memories related to the new
question and injects the stored convention. The agent can answer `pnpm test`
without asking the user to restate it.

```mermaid
sequenceDiagram
    participant U as User
    participant P as Pi agent
    participant M as Memory engine
    participant J as JSON store
    U->>P: Remember: use pnpm; tests run with pnpm test
    P->>M: remember(summary, tags)
    M->>J: atomic persist
    Note over U,P: Process exits; a new session starts
    U->>P: How do I run the tests?
    P->>M: inject(query, budget=2000)
    M->>J: load observations
    M-->>P: Relevant memory: use pnpm test
    P-->>U: Run pnpm test
```

## CLI-only smoke test

The storage and retrieval path can also be verified without installing Pi:

```bash
export PI_MEMORY_PATH="$PWD/.pi-memory.json"
python -m memory.cli capture \
  --summary "This project uses pnpm; run tests with pnpm test" \
  --tags "tooling,testing"
python -m memory.cli inject \
  --query "How do I run the tests?" \
  --budget 500
```

For a portfolio-ready recording, run the two sessions above in a clean terminal
and capture both the `remember` tool call and the fresh-session answer.
