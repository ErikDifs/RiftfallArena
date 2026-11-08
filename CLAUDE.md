# Riftfall Arena — how sessions run

Game code lives in the open Studio place and is edited through the `Roblox_Studio` MCP ([ADR-0005](docs/adr/0005-game-code-lives-in-studio-edited-through-mcp.md)). This repo holds the docs, ADRs and a `.rbxl` snapshot. `CONTEXT.md` is the glossary: use its terms.

## Starting a session

1. Read `CONTEXT.md`, the current ticket and the previous ticket's open points in `docs/milestone-1-tickets.md`, the ADRs the ticket names, and `docs/code-structure.md`.
2. Check the latest commit and the current published checkpoint in the ticket file's Checkpoints table.
3. Call `list_roblox_studios` (the id changes when Studio reopens) and `get_studio_state`.

## Ticket loop

1. **Settle** the ticket's open points with the developer before building. Record what's settled in the ticket.
2. **Build** in focused edits, one system at a time, with specs in `ServerStorage.Tests`.
3. **Test** with `RunAll`, then the ticket's checks in Play. Read [`docs/testing/studio-mcp-quirks.md`](docs/testing/studio-mcp-quirks.md) first. It's the canonical testing reference, so record new quirks there.
4. **Record** the result in the ticket, in the style of earlier tickets, and commit.
5. **Stop.** Work waits for the developer's go-ahead after every ticket. The developer publishes; record the version in Checkpoints.

Always ask before: uploading any asset (animations included) to the developer's account, publishing, or anything else that leaves the machine.

## Who does what

The main session holds the design context, so it does the work that needs judgment. Subagents get the work that is heavy on tokens but light on judgment, which also keeps the main context clean.

| Work | Who | Model |
|---|---|---|
| Settling open points, design, ADRs, planning edits | main session | Opus |
| Gameplay logic and anything touching security or authority (`Rules`, server systems, `Protocol`) | main session | Opus |
| Reviewing a subagent's work before it's accepted | main session | Opus |
| Long Studio test runs: Play sessions, real input, logs, forged requests | `studio-tester` | Sonnet |
| Exactly specified edits: specs from a written list, Config tweaks, cosmetic client code, renames | `studio-implementer` | Sonnet |
| Read-only lookups: where something is used, what a script does, finding a doc passage | `scout` | Haiku |

Rules for delegating:

- **A subagent result is evidence, not approval.** The main session remains responsible for reviewing delegated work, accepting implementation, and deciding whether a ticket is complete.
- **One agent drives Studio at a time.** Studio's Play state, camera and input are shared, so the main session doesn't touch Studio while `studio-tester` or `studio-implementer` runs, and they never run together.
- **Hand over a complete brief:** exact scripts, the behaviour wanted, the checks and the expected results. A subagent starts with none of this conversation.
- **Ask for compact reports:** a results table and anything unexpected, not raw logs.
- Don't delegate small things. A two-call lookup is cheaper done directly than briefed.
- Escalate rather than downgrade: if a subagent's result looks wrong, the main session checks it itself.
