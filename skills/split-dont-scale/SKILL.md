---
name: split-dont-scale
description: "Model and effort routing for coding agents. Use when choosing the model or effort of a subagent, worker or dispatched ticket, or when a task seems to need Opus or xhigh to implement: that is the signal to split it into Sonnet-sized tickets."
---

# Split, don't scale

Implementation runs on Sonnet. The larger models plan, split, review and orchestrate; they write no ticket's code. A ticket that seems to need Opus or `xhigh` to implement is **too big**: split it, then dispatch the slices to Sonnet.

## Routing

| Role | Model | Effort |
|---|---|---|
| Implement a ticket (code, tests, its docs) | Sonnet | `medium`; `high` when it touches persisted data, auth or concurrency, or after one failed attempt |
| Chores: exports, merging green PRs, cleanup, closing tickets | Haiku | `medium` (Sonnet `low` if Haiku fumbles the tooling) |
| Split and plan: grill, spec, tickets, ADR | Opus | `high` |
| Review a branch, one reviewer for both axes | Opus | `high` |
| Orchestrate: dispatch, gate plans, read reports | Opus | `high` |
| Chart a foggy multi-week effort | Fable | `high`, short session, decisions only |

`xhigh` and `max` are planning settings. Reaching for them on implementation is the split signal.

Cache reads cost the same per token on Sonnet and Opus: the swap halves output and cache-write cost, never context cost. Keep worker sessions short and their context small; that lever is worth as much as the model.

## The Sonnet test

A ticket is Sonnet-sized when every line holds:

- **One seam**: one module or layer plus its tests, with the files to touch named.
- **Decided**: every design choice is already in the ticket, spec or ADR; the implementer only executes.
- **Observable**: each acceptance criterion names a command that goes red, then green.
- **One session**: about 150 tool calls; large docs read by section, never whole.
- **Edges written**: blockers declared, and no parallel ticket touches the same files.

A ticket that fails a line goes to Split, never to a larger model.

## Split

Route by what is missing:

- Open decisions: `/grill-with-docs` (or `/grill-me`), then `/to-spec`.
- Clear spec, ticket too wide: `/to-tickets`, tracer-bullet slices, each one vertical path through one seam.
- Fog, no visible destination: `/wayfinder`, on Opus or Fable.
- Unsure which route: suggest `/ask-matt` to the human (it is user-invoked; an agent reads its `SKILL.md` instead).

Without those skills, write the slices yourself in the same shape. Split is done when every slice passes the Sonnet test and its edges are written.

## When Sonnet fails a ticket

Read the failure before relaunching:

- Spec gap or false premise: fix the ticket; re-split if it grew.
- Too wide: split the remainder into new tickets.
- Same failure twice on a ticket that passes the test: stop and ask the human. An Opus implementation runs only on their explicit go, recorded as an exception in the ticket.

## Dispatch

- Claude Code CLI: `claude --model sonnet --effort medium`.
- Claude Code subagent: `model: sonnet` in its frontmatter, or the `model` parameter of the Agent tool.
- Orca: `orca orchestration worker-start --agent claude --model sonnet --effort medium`.

Write the chosen model and effort into the ticket's spec, so the worker and its commit trailer name the same model.
