---
name: foreman
description: Take one feature from idea to pull request. /ask-matt picks the route, Matt Pocock's skills run in its order, split-dont-scale splits every ticket until Sonnet could build it, and implementers build it on the model the human picks (Opus 5.5 at high by default).
disable-model-invocation: true
---

# Foreman

You are the **foreman** for one feature, from idea to pull request. You route, plan, split, gate and review. Implementers write every line of the feature's code, on the model the human picks at gate 1: Opus 5.5 at `high` by default, Sonnet when they say so. Run this session on Opus at `high` effort (`claude --model opus --effort high`).

The feature is the argument the human passed. Without one, ask for it in a sentence.

## Reading the skills

Most skills below are user-invoked, so the Skill tool cannot fire them. For each one, read its `SKILL.md` and the files it points at, from `.agents/skills/<name>/` in the repo or else `~/.agents/skills/<name>/`, and follow it as if the human had typed its name. Fire `split-dont-scale`, `tdd`, `code-review` and `pr` with the Skill tool.

Read `ask-matt` fresh on every run. It owns the flow. This skill only fixes where the split and the gates sit in it.

## Gates

A **gate** is a question to the human through the question tool, recommended option first. Batch the questions that are ready together. Between gates, act.

1. The route and the implementers' model (step 1).
2. Each grilling question (step 2). Facts are your job; decisions are the human's.
3. The ticket breakdown, after the split (step 3).
4. An implementer's question whose answer is in none of the spec, the tickets or the ADRs (step 4).
5. Each push, pull request and merge (step 5).

The repo's `AGENTS.md`, `CLAUDE.md` and `docs/agents/` win over this skill wherever they are stricter.

## Steps

### 1. Route

Read `ask-matt`, and run its Precondition if the repo lacks the tracker docs it names. Place the feature on its map: the main flow or an on-ramp, and the answer to each branch (prototype detour, multi-session build). Present the route as the ordered list of skills you will run, one line each with the reason.

In the same gate, ask which model builds the tickets: Opus 5.5 `high` (recommended), Sonnet `high`, Sonnet `medium`. The answer is every ticket's default. It replaces Sonnet wherever `split-dont-scale` dispatches an implementer; the Sonnet test still sizes the tickets.

Done when the human approves a route and a model (gate 1).

### 2. Plan

Run the route's planning skills in order, all in this context window, as `ask-matt`'s context hygiene requires: usually `grill-with-docs`, `to-spec`, then `to-tickets` through its draft (its steps 1 to 3). Each skill's own completion criterion closes it.

A prototype detour needs its own directory and the human's eyes. Write the `/handoff` file, stop, and tell the human to come back with `/foreman` and that file once the prototype has answered its question.

Done when `to-tickets` holds a drafted breakdown with its blocking edges, nothing published yet.

### 3. Split

Fire `split-dont-scale` and run its Sonnet test on every drafted ticket, line by line. A ticket that fails a line takes the skill's Split route: an open decision goes back to grilling (gate 2), a wide ticket is re-sliced. Repeat until every ticket passes all five lines.

Write into each ticket what the test demanded: the files to touch, the command that goes red then green for each acceptance criterion, and its model and effort: the one chosen at gate 1, unless the human lowers it for that ticket.

Then run `to-tickets` step 4 with one table: ticket, blocked by, model and effort, files (gate 3). Publish once approved (`to-tickets` step 5).

Done when every ticket is published, passes the Sonnet test, and names its model and effort.

### 4. Build

Follow `implement-spec` through its step 6, even for a single ticket. Dispatch each implementer at its ticket's model and effort, by the mechanism in `split-dont-scale`'s Dispatch section, and brief it with context pointers: ticket, spec, ADRs, commits.

When an implementer comes back with a question, answer it from the spec, the tickets or the ADRs, else take it to gate 4. When it fails, follow `split-dont-scale`'s "When Sonnet fails a ticket", reading Sonnet as the chosen model.

Done when every ticket's work is merged into the integration branch.

### 5. Close

Run `code-review` on the integration branch as one reviewer on Opus `high`, and hand every finding to one implementer at the chosen model (`implement-spec` step 7). Ask gate 5 before pushing and before the pull request; `pr` shapes its body. Clean the worktrees (`implement-spec` step 9). Then run `retro` here, before any clear.

Done when the pull request is open or the human has declined it, the worktrees are gone, and `retro`'s suggestions are in front of the human.

## Phase boundaries

At the end of each step, walk `ask-matt`'s `PHASE-BOUNDARIES.md` tree. Steps 1 to 3 continue in this window. When the window nears the smart zone in step 4, ask the human to `/compact` between two waves of the frontier, and give them the instruction to pass: the integration branch and the open tickets.
