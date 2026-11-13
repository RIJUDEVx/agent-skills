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

## Skills for the work itself

These are not part of the pipeline above. They are here because every repo needs them.

- **[pr](./pr/SKILL.md)**: Opens, updates and dresses a pull request. Reads the repo's base branch, PR template, title format, labels and checks instead of inventing one. A repo's own `.agents/skills/pr/SKILL.md` overrides it.
- **[diagnosing-bugs](./diagnosing-bugs/SKILL.md)**: Builds a tight failing signal first, then explains the cause. Useful when a failure, flake or slowdown has no obvious reason.
- **[writing-for-agents](./writing-for-agents/SKILL.md)**: How to write a document an agent reads well: skills, `AGENTS.md`, or a doc reached by a pointer.
- **[gh](./gh/SKILL.md)**: GitHub CLI patterns: structured output, pagination, repo targeting, search vs list, the `gh api` fallback.
- **[no-use-effect](./no-use-effect/SKILL.md)**: The React rule that components never call `useEffect` directly: derive state, handle events, use a data library, or use `useMountEffect`.

## Skills for the interface

- **[design-engineering](./design-engineering/SKILL.md)**: The craft rules that only break after the build: a hover that moves the layout, a weight that is wider than the one it replaced, a transition that drops frames. Start with the nav-hover rule, then the variable-font axes.
- **[impeccable](./impeccable/SKILL.md)**: A design director for interface work: shape, critique, audit, animate, typeset, colour, motion, accessibility, and design-system capture.
- **[improve-animations](./improve-animations/SKILL.md)**: Surveys a codebase's motion code and writes prioritized, self-contained plans for other agents to execute. Read-only on source.
- **[vitest](./vitest/SKILL.md)**: Vitest's API, mocking, configuration, and test filtering, in one reference.
- **[cloudflare](./cloudflare/SKILL.md)**: Picks the Cloudflare product for a job, then loads the reference for it.
- **[analyze-logs](./analyze-logs/SKILL.md)** and **[build-audit-logs](./build-audit-logs/SKILL.md)**: Read structured wide-event logs, and build or review an audit trail on top of them.


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

The skills read per-repo rules from the repo itself, so put those in the repo, not here. A repo's `AGENTS.md`, `CONTRIBUTING.md`, and `.agents/skills/pr/SKILL.md` set its base branch, CI command, and PR format. The global [`pr`](./pr/SKILL.md) skill reads those same places and fills in the gaps when a repo documents none. `.agents/docs/issue-tracker.md` tells `triage` and `code-review` where the issues live.

## Layout

Each folder is one skill: `SKILL.md` is what the agent reads, and `references/` holds longer material it reads when needed. `agents/openai.yaml` is Codex metadata. `full-send/routes/` holds two guides that only `full-send` uses: `to-tickets` (spec to tickets) and `implement` (TDD loop and parallel workers).

## Credits

Most of these started as other people's skills. I cut them down, joined them into one pipeline, and rewrote the parts that didn't fit how I work. The original notices, including the Apache 2.0 ones, are in [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md).

- [mattpocock/skills](https://github.com/mattpocock/skills) by Matt Pocock: `grilling`, `code-review`, `triage`, `retro`, `wait-what`, the codebase survey in `architect`, the `to-tickets` and `implement` guides, `writing-for-agents`, `diagnosing-bugs`.
- [pstack](https://github.com/cursor/plugins) by Lauren Tan: `architect`, `interrogate`, `unslop`.
- [tstack](https://github.com/t3dotgg/tstack) by Theo Browne: `full-send`, `full-stop`.
- [cli/cli](https://github.com/cli/cli): `gh`.
- [antfu/skills](https://github.com/antfu/skills): `vitest`.
- [emilkowalski/skills](https://github.com/emilkowalski/skills): `improve-animations`.
- [evloghq/evlog](https://github.com/evloghq/evlog): `analyze-logs`, `build-audit-logs`.
- [pbakaus/impeccable](https://github.com/pbakaus/impeccable) and [cloudflare/skills](https://github.com/cloudflare/skills), both Apache 2.0: `impeccable`, `cloudflare`.
- [alejandrobailo/no-use-effect](https://github.com/alejandrobailo/no-use-effect): `no-use-effect`.

`pr` and `design-engineering` are written here. `pr` started from the selegic-core PR skill, with the repo-specific parts replaced by rules for reading a repo's own conventions.

## License

[MIT](./LICENSE)
