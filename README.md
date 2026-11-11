# Agent skills

My global skills for coding agents. They take a piece of work from "I'm not sure what I want" to a merged pull request. I can walk away while the agent builds, proves, and reviews the change.

They are plain Markdown (`SKILL.md` files) and work in Claude Code, Codex, and OpenCode. They work best in T3 Code, which lets one thread hand work to models from different providers. Outside T3 Code, the skills fall back to the harness's own sub-agents.

## What to type, and when

| Situation | Type | What happens |
|---|---|---|
| I don't know what to build yet, or how it should be shaped | `$architect` | The agent reads the code the change touches, asks you questions, sketches two or more designs, and stops with one agreed design. It writes no code. |
| I know what I want and will merge it myself | `$full-stop` | The agent builds, proves, and reviews the change, then opens a PR and gets it ready. You merge. |
| I know what I want and I'm not watching | `$full-send` | Same as `$full-stop`, then the agent merges the PR once every check passes. |
| There are issues or outside PRs to sort | `$triage` | The agent labels them, reproduces bug reports, and writes briefs an agent can pick up. |
| A session went badly | `$retro` | The agent suggests changes to checks, docs, and steering files that would have prevented the mistakes. |
| The agent's last message made no sense | `$wait-what` | The agent explains it again in simple words, using the project's glossary. |

Claude Code also accepts `/full-send` and so on.

A common path: `$architect` to agree on the shape, then `$full-stop` to build it.

## What a `$full-send` run does

1. **Frame.** It writes the goal, what "done" means, what is out of scope, and how it will prove the change works.
2. **Size.** It picks a route:
   - **Direct:** one small concern. It builds the change straight away.
   - **Design:** it fits one session, but the shape is open. It runs `architect` first and makes the design decisions itself.
   - **Split:** too big for one session. It designs, cuts the work into tickets, and builds them in parallel worktrees.
3. **Impact check.** It asks what else the change could break, and proves the one fact the change depends on.
4. **Build and prove.** It tests first where a test is cheap. Proof means something it saw in the session: a reproduction that now passes, the repo's CI command, or the running app for UI changes.
5. **Review.** Up to three passes, and it fixes or declines each finding before the next:
   - `code-review` checks the repo's standards and the original spec.
   - `interrogate` has several models try to break risky changes (auth, payments, data loss, concurrency). Only findings a run proves count.
   - A review loop with one outside model, three rounds at most.
6. **Ship.** It opens the PR with the repo's own PR rules, waits for CI, and answers review comments.
7. **Merge or stop.** `$full-send` merges only when the proof covers the whole diff, the required check is green on the latest commit, and no review asks for changes. `$full-stop` stops here.
8. **Report.** It lists the route, which stages ran or were skipped and why, the proof, and every decision it made for you.

It asks you only when it can't prove the change, can't get the checks to pass, or the next step is risky (deleting data, touching production, closing a PR). Everything else it decides and lists in the report.

## Skills the agent loads on its own

You can type these too, but usually another skill or the agent pulls them in.

- **[grilling](./grilling/SKILL.md)**: Asks you questions in numbered rounds, each with a recommended answer. Records agreed terms in `GLOSSARY.md` and hard-to-reverse decisions as ADRs.
- **[code-review](./code-review/SKILL.md)**: Reviews a diff on two separate axes at once: does it follow the repo's rules, and does it do what the issue asked.
- **[interrogate](./interrogate/SKILL.md)**: Has 3 or 4 models from different families try to break a change. A finding counts only if a command shows the failure.
- **[unslop](./unslop/SKILL.md)**: Rewrites PR bodies and docs so they don't read as machine-written.
- **[t3-tools](./t3-tools/SKILL.md)**: The exact T3 Code tool names, and the model tiers that decide which model does which job.

## Setup

You need `git`, and the GitHub CLI (`gh`) logged in, for the PR steps.

The skills call each other, so install the whole set, not single skills:

```bash
git clone https://github.com/RIJUDEVx/agent-skills ~/.agents/skills
ln -s ~/.agents/skills ~/.claude/skills   # only if you use Claude Code
```

Then change two things that are specific to my machine:

1. **Paths.** `full-send` and `full-stop` load their guides from `/Users/home/.agents/skills/...`. Replace `/Users/home` with your home directory.
2. **Models.** The Tiers table in [`t3-tools`](./t3-tools/SKILL.md) lists the models I use (Fast, Balanced, Strong, Cheap strong). Replace them with yours.

The skills read per-repo rules from the repo itself, so put those in the repo, not here. A repo's `AGENTS.md`, `CONTRIBUTING.md`, and `.agents/skills/pr/SKILL.md` set its base branch, CI command, and PR format. `.agents/docs/issue-tracker.md` tells `triage` and `code-review` where the issues live.

## Layout

Each folder is one skill: `SKILL.md` is what the agent reads, and `references/` holds longer material it reads when needed. `agents/openai.yaml` is Codex metadata. `full-send/routes/` holds two guides that only `full-send` uses: `to-tickets` (spec to tickets) and `implement` (TDD loop and parallel workers).

## Credits

Most of these started as other people's skills. I cut them down, joined them into one pipeline, and rewrote the parts that didn't fit how I work. The original MIT notices are in [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md).

- [mattpocock/skills](https://github.com/mattpocock/skills) by Matt Pocock: `grilling`, `code-review`, `triage`, `retro`, `wait-what`, the codebase survey in `architect`, and the `to-tickets` and `implement` guides.
- [pstack](https://github.com/cursor/plugins) by Lauren Tan: `architect`, `interrogate`, `unslop`.
- [tstack](https://github.com/t3dotgg/tstack) by Theo Browne: `full-send`, `full-stop`.

## License

[MIT](./LICENSE)
