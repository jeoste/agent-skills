# agent-skills

[![skills.sh](https://skills.sh/b/jeoste/agent-skills)](https://skills.sh/jeoste/agent-skills)

Skills for coding agents (Claude Code, Codex, Cursor and the other agents the [`skills`](https://github.com/vercel-labs/skills) CLI supports), written while running multi-agent builds on a production codebase.

## Skills

| Skill | What it does |
|---|---|
| [`split-dont-scale`](skills/split-dont-scale/SKILL.md) | Routes every implementation ticket to Sonnet and keeps the larger models for planning, splitting, review and orchestration. A ticket that seems to need Opus or `xhigh` is treated as too big: it goes back to be split, never up to a bigger model. |

## Install

```sh
npx skills add jeoste/agent-skills --skill split-dont-scale
```

Add `-g` to install it for every project.

## split-dont-scale

Implementation sessions are where the tokens go: long contexts read again on every tool call. Moving implementation from Opus to Sonnet halves output and cache-write cost, while cache reads cost the same per token on both, so the skill pairs the model rule with short worker sessions and small contexts.

The skill gives:

- a routing table (role, model, effort) for implementers, chores, planners, reviewers and orchestrators;
- the **Sonnet test**, five checks a ticket passes before dispatch (one seam, decided, observable, one session, edges written);
- a split route by what is missing, and what to do when Sonnet fails a ticket.

It works best alongside [Matt Pocock's skills](https://github.com/mattpocock/skills) (`/grill-with-docs`, `/to-spec`, `/to-tickets`, `/wayfinder`, `/ask-matt`), which do the splitting. Without them, the agent writes the slices itself in the same shape.

## License

[MIT](LICENSE)
