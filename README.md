# Agent skills

The skills I use every day to plan, build, review, and ship code with coding agents.

They are small Markdown files. Any agent that reads `SKILL.md` files can use them: Claude Code, Codex, OpenCode, and others. A few parts are tuned for T3 Code, which lets one thread hand work to models from several providers.

Most of these started as other people's skills. I trimmed them, joined them into one pipeline, and changed the parts that did not fit how I work. See [Credits](#credits).

## Install

Clone the repo where your agents look for global skills:

```bash
git clone https://github.com/RIJUDEVx/agent-skills ~/.agents/skills
ln -s ~/.agents/skills ~/.claude/skills   # Claude Code reads ~/.claude/skills
```

Or copy single skills into a project with the [skills CLI](https://skills.sh):

```bash
npx skills@latest add RIJUDEVx/agent-skills
```

Two things to change before you use `full-send` or `full-stop`:

- They load their guides by absolute path (`/Users/home/.agents/skills/...`). Replace `/Users/home` with your home directory.
- The model tiers in [`t3-tools`](./t3-tools/SKILL.md) list the models I pay for. Put your own models in the Tiers table.

## How they fit together

```
 unclear what to build?          clear enough?
          │                            │
     $architect ── grilling            │
          │      (asks you)            │
          ▼                            ▼
   agreed design ─────────────► $full-stop  ── PR ready, you merge
                                $full-send  ── PR merged for you
```

`$full-send` runs 13 stages: frame, size, impact, design, plan, build, prove, review, open PR, push and gate, babysit, merge, report. Stage 2 picks a route:

| Route | When | Extra stages |
|---|---|---|
| Direct | One concern, obvious shape | none |
| Design | Fits one session, shape is open | `architect` |
| Split | Too big for one session | `architect`, then tickets and parallel workers |

Every change goes through up to three review passes before the PR opens: `code-review` (repo standards and the spec), `interrogate` for risky changes (several models try to break it, and only findings a run proves count), and a review loop with one outside model.

`$full-stop` does the same work but never merges.

## Reference

**User-invoked** skills run only when you type them. They orchestrate. **Model-invoked** skills can also be loaded by the agent when the task fits. They hold the reusable discipline.

### User-invoked

- **[architect](./architect/SKILL.md)**: Work out what to build, trace the code it touches, sketch at least two designs on different models, and stop with one agreed design. Also surveys a codebase for deepening opportunities.
- **[full-send](./full-send/SKILL.md)**: Finish the work end to end: build, prove, review, open the PR, wait for CI, fix review comments, and merge.
- **[full-stop](./full-stop/SKILL.md)**: Everything `full-send` does, but stop with the PR ready for you to merge.
- **[triage](./triage/SKILL.md)**: Move issues and outside PRs through triage labels, verify claims, and write briefs an agent can pick up.
- **[retro](./retro/SKILL.md)**: After a session, suggest changes to the agent's environment (checks, docs, steering files, tools) that would have prevented its mistakes.
- **[wait-what](./wait-what/SKILL.md)**: Fire this when a message doesn't land. The agent explains it again, simply, with your glossary terms.

### Model-invoked

- **[grilling](./grilling/SKILL.md)**: Ask you about a plan in rounds of numbered questions, each with a recommended answer, and write `GLOSSARY.md` and ADRs as decisions land.
- **[code-review](./code-review/SKILL.md)**: Review a diff on two separate axes in parallel: Standards (repo rules plus a Fowler smell baseline) and Spec (did it do what the issue asked).
- **[interrogate](./interrogate/SKILL.md)**: Have 3 or 4 models from different families try to break a change. A finding survives only if a run proves it. Also has a blast-radius mode for "what does this break beyond the diff?"
- **[unslop](./unslop/SKILL.md)**: Rewrite PR bodies, docs, and other prose to remove the patterns that make text read as machine-written.
- **[t3-tools](./t3-tools/SKILL.md)**: The exact T3 Code tool names for delegating to other models, linking and watching PRs, and showing visuals, plus the model tiers every other skill uses.

### Guides inside full-send

These are not skills on their own. `full-send` loads them when a route needs them.

- [`to-tickets`](./full-send/routes/to-tickets/GUIDE.md): Write a spec if there is none, then cut it into tracer-bullet tickets with blocking edges.
- [`implement`](./full-send/routes/implement/GUIDE.md): Build from a spec or tickets with a TDD loop, fanning tickets out to parallel workers in their own worktrees.
- [`review-loop`](./full-send/references/review-loop.md): Rounds with one outside reviewer, a severity ledger, and rules for when to stop.

## Credits

These skills are adapted from MIT-licensed work. The original copyright notices are in [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md).

- [mattpocock/skills](https://github.com/mattpocock/skills) by Matt Pocock: `grilling`, `code-review`, `triage`, `retro`, `wait-what`, the codebase survey in `architect`, and the `to-tickets` and `implement` guides.
- [pstack](https://github.com/cursor/plugins) by Lauren Tan: `architect`, `interrogate`, `unslop`.
- [tstack](https://github.com/t3dotgg/tstack) by Theo Browne: `full-send`, `full-stop`.

## License

[MIT](./LICENSE)
