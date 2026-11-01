# agent-skills

[![skills.sh](https://skills.sh/b/jeoste/agent-skills)](https://skills.sh/jeoste/agent-skills)

Skills for coding agents (Claude Code, Codex, Cursor and the other agents the [`skills`](https://github.com/vercel-labs/skills) CLI supports), written while running multi-agent builds on a production codebase.

## Skills

| Skill | What it does |
|---|---|
| [`split-dont-scale`](skills/split-dont-scale/SKILL.md) | Routes every implementation ticket to Sonnet and keeps the larger models for planning, splitting, review and orchestration. A ticket that seems to need Opus or `xhigh` is treated as too big: it goes back to be split, never up to a bigger model. |
| [`foreman`](skills/foreman/SKILL.md) | Takes one feature or one milestone to a pull request: `/ask-matt` picks the route, Matt Pocock's skills run in its order, `split-dont-scale` splits every ticket until Sonnet could build it, and implementers build it on the model the human picks (Opus 5.5 at `high` by default). The foreman asks the human at five gates, and a roll call at the end proves every ticket closed. |

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

Type `/foreman <feature>`, or `/foreman milestone <name>` for tickets already published, in a session on Opus at `high` effort. The skill reads `/ask-matt` fresh on every run and follows its flow; it only fixes where the split, the human's questions and the roll call sit in it:

1. **Route**: the ordered list of skills for this feature, or the milestone's open tickets read from the tracker, and the implementers' model (Opus 5.5 `high` by default, Sonnet on request).
2. **Plan**: `/grill-with-docs`, `/to-spec`, then `/to-tickets` up to its draft, in one context window. A milestone, already ticketed, skips this step.
3. **Split**: the Sonnet test on every ticket, then one table (ticket, blocked by, model and effort, files) before publishing new tickets or editing a milestone's in place.
4. **Build**: `/implement-spec`, one implementer per ticket at the chosen model.
5. **Close**: one review, its fixes, the pull request, then a roll call on the tracker: every ticket closed or closed by the pull request, with a comment naming its commit and its command, and no open issue left in the milestone. Then `/retro`. After the merge, a second roll call, then the milestone is closed.

It needs `split-dont-scale` and Matt Pocock's skills installed.

## License

[MIT](LICENSE)
