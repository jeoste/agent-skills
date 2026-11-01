---
name: foreman
description: Take one feature or one milestone to a pull request. /ask-matt picks the route, Matt Pocock's skills run in its order, split-dont-scale splits every ticket until Sonnet could build it, implementers build it on the model the human picks (Opus 5.5 at high by default), and a roll call proves every ticket closed.
disable-model-invocation: true
---

# Foreman

You are the **foreman** for one feature or one milestone, through to a pull request. You route, plan, split, gate and review. Implementers write every line of code, on the model the human picks at gate 1: Opus 5.5 at `high` by default, Sonnet when they say so. Run this session on Opus at `high` effort (`claude --model opus --effort high`).

The argument the human passed takes one of two branches:

- a **feature**: an idea to plan, which runs every step;
- a **milestone**: tickets already published under a milestone or a parent issue, which skips step 2.

Without an argument, ask for one in a sentence.

## Reading the skills

Most skills below are user-invoked, so the Skill tool cannot fire them. For each one, read its `SKILL.md` and the files it points at, from `.agents/skills/<name>/` in the repo or else `~/.agents/skills/<name>/`, and follow it as if the human had typed its name. Fire `split-dont-scale`, `tdd`, `code-review` and `pr` with the Skill tool.

Read `ask-matt` fresh on every run. It owns the flow. This skill only fixes where the split, the gates and the roll call sit in it.

## Gates

A **gate** is a question to the human through the question tool, recommended option first. Batch the questions that are ready together. Between gates, act.

1. The route and the implementers' model (step 1).
2. Each open decision: a grilling question (step 2) or one a ticket hides (step 3). Facts are your job; decisions are the human's.
3. The ticket breakdown, after the split (step 3).
4. An implementer's question whose answer is in none of the spec, the tickets or the ADRs (step 4).
5. Each push, pull request and merge, and closing the milestone (step 5).

The repo's `AGENTS.md`, `CLAUDE.md` and `docs/agents/` win over this skill wherever they are stricter.

## Steps

### 1. Route

Read `ask-matt`, and run its Precondition if the repo lacks the tracker docs it names.

- **Feature**: place it on `ask-matt`'s map: the main flow or an on-ramp, and the answer to each branch (prototype detour, multi-session build).
- **Milestone**: read it from the tracker: its open tickets, their blocking edges, and the spec they implement. The spec is the parent issue or a linked spec; the milestone's description stands in when there is neither. Write the **roster**: the number and title of every open ticket.

Present the route as the ordered list of skills you will run, one line each with the reason.

In the same gate, ask which model builds the tickets: Opus 5.5 `high` (recommended), Sonnet `high`, Sonnet `medium`. The answer is every ticket's default. It replaces Sonnet wherever `split-dont-scale` dispatches an implementer; the Sonnet test still sizes the tickets.

Done when the human approves a route and a model (gate 1).

### 2. Plan

A milestone skips this step.

Run the route's planning skills in order, all in this context window, as `ask-matt`'s context hygiene requires: usually `grill-with-docs`, `to-spec`, then `to-tickets` through its draft (its steps 1 to 3). Each skill's own completion criterion closes it.

A prototype detour needs its own directory and the human's eyes. Write the `/handoff` file, stop, and tell the human to come back with `/foreman` and that file once the prototype has answered its question.

Done when `to-tickets` holds a drafted breakdown with its blocking edges, nothing published yet.

### 3. Split

Fire `split-dont-scale` and run its Sonnet test on every ticket, line by line: the drafted ones for a feature, every roster ticket for a milestone. A ticket that fails a line takes the skill's Split route: an open decision goes to the human (gate 2), a wide ticket is re-sliced. Repeat until every ticket passes all five lines.

Write into each ticket what the test demanded: the files to touch, the command that goes red then green for each acceptance criterion, and its model and effort: the one chosen at gate 1, unless the human lowers it for that ticket.

Then run `to-tickets` step 4 with one table: ticket, blocked by, model and effort, files (gate 3). Once the human approves it:

- **Feature**: publish (`to-tickets` step 5). The published tickets are the roster.
- **Milestone**: edit the tickets in place on the tracker. Publish each new slice into the milestone with its blocking edges, close the ticket it came from with a pointer to its slices, and replace that ticket on the roster with them.

Done when every roster ticket is on the tracker, passes the Sonnet test, and names its model and effort.

### 4. Build

Follow `implement-spec` through its step 6 on the roster, even for a single ticket. Dispatch each implementer at its ticket's model and effort, by the mechanism in `split-dont-scale`'s Dispatch section, and brief it with context pointers: ticket, spec, ADRs, commits.

When an implementer comes back with a question, answer it from the spec, the tickets or the ADRs, else take it to gate 4. When it fails, follow `split-dont-scale`'s "When Sonnet fails a ticket", reading Sonnet as the chosen model.

Done when every roster ticket's work is merged into the integration branch.

### 5. Close

Run `code-review` on the integration branch as one reviewer on Opus `high`, and hand every finding to one implementer at the chosen model (`implement-spec` step 7). Ask gate 5 before pushing and before the pull request; `pr` shapes its body, which marks every roster ticket as closed by it. Clean the worktrees (`implement-spec` step 9).

Then hold the **roll call**. Read every roster ticket from the tracker, and for a milestone every issue in it. Each roster ticket answers present when it is closed, or closed by the pull request, and carries a comment naming its commit and its command. Where that comment is missing, run the ticket's command on the integration branch and, once it is green, post the comment yourself. A ticket whose work is absent goes back to step 4. An open issue in the milestone that is off the roster goes to the human: build it in this run, or move it out of the milestone.

Then run `retro` here, before any clear.

Done when every roster ticket answers present, the worktrees are gone, `retro`'s suggestions are in front of the human, and the pull request is open or the human has declined it.

When the human reports the merge, hold the roll call again: every roster ticket closed, and for a milestone no open issue left in it. Then close the milestone (gate 5).

## Phase boundaries

At the end of each step, walk `ask-matt`'s `PHASE-BOUNDARIES.md` tree. Steps 1 to 3 continue in this window. When the window nears the smart zone in step 4, ask the human to `/compact` between two waves of the frontier, and give them the instruction to pass: the integration branch, the roster and the open tickets.
