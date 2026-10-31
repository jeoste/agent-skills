# agent-skills

[![skills.sh](https://skills.sh/b/jeoste/agent-skills)](https://skills.sh/jeoste/agent-skills)

Skills for coding agents (Claude Code, Codex, Cursor and the other agents the [`skills`](https://github.com/vercel-labs/skills) CLI supports), written while running multi-agent builds on a production codebase.

## Skills

| Skill | What it does |
|---|---|
| [`split-dont-scale`](skills/split-dont-scale/SKILL.md) | Routes every implementation ticket to Sonnet and keeps the larger models for planning, splitting, review and orchestration. A ticket that seems to need Opus or `xhigh` is treated as too big: it goes back to be split, never up to a bigger model. |
| [`foreman`](skills/foreman/SKILL.md) | Takes one feature from idea to pull request: `/ask-matt` picks the route, Matt Pocock's skills run in its order, `split-dont-scale` splits every ticket until Sonnet can build it, and Sonnet implementers build it. The foreman asks the human at five gates. |

## Install

```sh
npx skills add jeoste/agent-skills --skill split-dont-scale
npx skills add jeoste/agent-skills --skill foreman
```

Add `-g` to install it for every project.

## split-dont-scale

Implementation sessions are where the tokens go: long contexts read again on every tool call. Moving implementation from Opus to Sonnet halves output and cache-write cost, while cache reads cost the same per token on both, so the skill pairs the model rule with short worker sessions and small contexts.

The skill gives:

- a routing table (role, model, effort) for implementers, chores, planners, reviewers and orchestrators;
- the **Sonnet test**, five checks a ticket passes before dispatch (one seam, decided, observable, one session, edges written);
- a split route by what is missing, and what to do when Sonnet fails a ticket.

It works best alongside [Matt Pocock's skills](https://github.com/mattpocock/skills) (`/grill-with-docs`, `/to-spec`, `/to-tickets`, `/wayfinder`, `/ask-matt`), which do the splitting. Without them, the agent writes the slices itself in the same shape.

## foreman

Type `/foreman <feature>` in a session on Opus at `high` effort. The skill reads `/ask-matt` fresh on every run and follows its flow; it only fixes where the split and the human's questions sit in it:

1. **Route**: the ordered list of skills for this feature.
2. **Plan**: `/grill-with-docs`, `/to-spec`, then `/to-tickets` up to its draft, in one context window.
3. **Split**: the Sonnet test on every drafted ticket, then one table (ticket, blocked by, model and effort, files) before publishing.
4. **Build**: `/implement-spec`, one Sonnet implementer per ticket.
5. **Close**: one review, its fixes, the pull request, `/retro`.

It needs `split-dont-scale` and Matt Pocock's skills installed.

## License

[MIT](LICENSE)
