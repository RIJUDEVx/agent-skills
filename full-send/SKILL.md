---
name: full-send
description: Use when the user invokes `$full-send` to finish work end-to-end and merge.
disable-model-invocation: true
---

# Full Send

The user invoked `$full-send`, so they are not watching this thread and want the
work completed end-to-end without them. Your updates and your final message go
unread until long after you stop. You own the work from its current state to a
merged PR.

If the user did not explicitly invoke `$full-send` or clearly ask for full send,
do not assume full autonomy. If they did, everything below applies.

## Grant

The user grants you permission to:

- Merge the PR once the merge gate passes. This is the only agent merge
  authority. It overrides any repo PR skill rule that says "mark ready, then
  stop." Sub-agents never merge.
- Force-push the PR's own branch with `--force-with-lease` after a rebase.
- Use the test methods that Proof needs, without asking first.

## Testing permission

A `$full-send` request is the permission. If a project file or other instruction
says to ask before a test method, or to leave testing to the user, this request
replaces that rule for this task.

That includes ask-first rules for computer use, browser automation, simulators
and devices, running the app or a dev server, screenshots, and visual checks. Use
the method that gives direct proof. List it in the report.

Rules about how to test still apply. Follow rules that name the correct command,
fixture, or check. This permission only removes the ask-first or
leave-it-to-the-user step. If a preferred method cannot prove the change, use a
gated method under this permission.

It covers only the tests needed for this task. It does not add permission for the
steps under "Asking the user", such as production changes or data deletion. It
also does not bypass a harness or system approval prompt if one appears.

## Repo rules

Look up the repo's rules before you open a PR:

- If `.agents/skills/pr/SKILL.md` exists, use it to open the PR. Its
  "mark ready, then stop" rule does not apply here. Full send's merge grant does.
- Otherwise open the PR directly with `gh pr create` as a draft, following
  `AGENTS.md` and `CONTRIBUTING.md`.
- Use the repo's documented CI command. Do not substitute a generic one.

## Stages

Run every stage in order. The user names only `$full-stop` or `$full-send`; picking
the skills is your job. Record each stage as run or skipped, with the reason, and
put that ledger in the report. A stage marked "may skip" needs a stated reason.

| # | Stage | Do |
|---|---|---|
| 1 | Frame | Write the goal, the done condition, what is out of scope, and how you will prove it, before any code |
| 2 | Size | Pick Direct, Design, or Split. See "Routes" |
| 3 | Impact | Use the Blast radius mode of `interrogate`: who and what this touches, and the one fact it is safe because of. May skip only for a local change with no shared code, schema, API, auth, or data |
| 4 | Design | Design and Split routes, when the shape is open: run `architect` |
| 5 | Plan | Split route: run `to-tickets` |
| 6 | Build | Read the `implement` guide. Direct and Design use its TDD loop when there is a cheap test target. Split uses "Fan out a spec". If you are on the default branch, create a branch first. Use the smallest change that works, and fix the case that happened, not every case that could |
| 7 | Prove | Run the proof from stage 1, plus anything impact flagged. See "Proof" |
| 8 | Review | Run each pass in "Review passes". Fix or decline every finding before the next pass |
| 9 | Open the PR | Rebase onto `origin/<base>`, then open the PR with the repo rules above. Write the body through `unslop`, with one short line on how you proved the change unless the template covers it. Use `t3-tools` for the T3 steps. Right after `gh pr create`, call `link_pull_request` with the URL if T3 tools are present |
| 10 | Push and gate | Push the tested commit, run the repo CI command, and wait for the required check to succeed on the head SHA before `gh pr ready`. Then start the watcher. See "Watching" |
| 11 | Babysit | Fix real findings and CI failures. Get proof for each fix before you push it. See "Review rules" |
| 12 | Merge | When the merge gate passes. If the repo uses a merge queue or auto-merge, stay until GitHub reports the PR as merged |
| 13 | Report | The route and why, the stage ledger, the PR link, the merge commit, your proof, the reviewer model used (and any substitution), and each decision you made for the user |

### Routes

Use what this session already shows. If that does not settle it, start one
read-only scout sub-agent on the cheapest capable model. It reports the files and
modules involved, any new or changed public interface or schema, how many slices
could ship independently, and any existing spec, tickets, or ADRs. It does not
edit. The route is your decision, not the scout's.

| Route | Use when |
|---|---|
| Direct | One concern, the shape is obvious, and it fits one session |
| Design | It fits one session, but the shape is open: a new module, interface, or data model, or more than one reasonable design |
| Split | It is too big for one session, or it holds several slices that ship independently |

If a spec or tickets exist, or an agreed `$architect` design does, skip the stages
they already cover. Take the lightest route that fits. If the work proves bigger
than the route, move up one and say so. All routes end in one PR.

### Review passes

| Pass | Runs |
|---|---|
| `code-review` | Every behavior change: repo standards and the spec |
| `interrogate`, review mode | Split route, a real risk from impact, or anything touching auth, permissions, payments, data loss, or concurrency |
| Review loop | Every behavior change. See "Review loop" |

Comment, doc, or format-only changes run the last pass only. `interrogate` findings
count only when a run proves them.

### Models

Every sub-agent model comes from the tiers in the `t3-tools` skill ("Choosing a
model"). That skill also says how to set one and what to do when one fails.

| Work | Tier |
|---|---|
| Scout, build, merge work, `code-review` | Fast. Move a job up to Balanced if a Fast model gets it wrong twice |
| `architect` runners | Strong: Opus plus one OpenCode model, so two families |
| `interrogate` reviewers | Strong: all three models, one per family |
| Review loop | Cheap strong, first model (see `references/review-loop.md`) |

### Loading

`code-review`, `interrogate`, `unslop`, and `t3-tools` are normal skills. Load
`architect` from `/Users/home/.agents/skills/architect/SKILL.md`. Load the others
from `/Users/home/.agents/skills/full-send/routes/<name>/GUIDE.md`: `to-tickets`
and `implement`.

Nobody is watching, so these rules override those skills and guides on this task:

- Where they say to check with the user (seams, the ticket breakdown, which
  candidate to pursue, unclear intent), decide yourself and list the decision in
  the report.
- `grilling` needs the user, so do not run it here.
- If the repo has no tracker docs, use the local ticket form in `to-tickets`.
- Replace the final review and "mark ready" steps in `implement` with stages 7 to
  13, run on the integration branch.
- Merger sub-agents may merge worktree branches into the integration branch. Only
  this thread merges the PR.
- A missing decision about how to build it is yours. A missing decision about
  what to build falls under "Asking the user". If several are missing, stop and
  tell the user to run `$architect` first.

Stay active from stage 1 to stage 13. If you end your turn while checks or reviews
are pending, the work stops, because nobody will prompt you again. Wait with the
watcher so something wakes you when the PR changes. If you stop before the merge,
call `unwatch_pull_request` first.

## Proof

Proof is evidence you observed yourself in this session: command output, a
reproduction, a screenshot. Use the most direct proof the change allows:

- Reproduce the original failure, apply the fix, then watch the same reproduction
  pass.
- Run the repo's documented CI command.
- Add one focused test that fails without the fix, when the behavior has none and
  the repo has a test setup.
- For UI and runtime behavior, run the app and look at the change. Green mocked
  tests do not prove that the user experience works. Attach screenshots to the PR
  when they help and you have a way to upload them.

When you cannot find a way to prove a change, you do not have full confidence.
Ask the user.

## Review loop

Before stage 8's last pass, read `/Users/home/.agents/skills/full-send/references/review-loop.md` and follow it. It holds the
reviewer model, budget, round brief, and stop rules.

## Review rules

Only act on checks and comments newer than the latest push. Verify every review
finding against the source before you change code. Fix real findings and CI
failures. Distinguish repository failures from infrastructure flakes. When you
dismiss a false positive, reply with a written reason and resolve the comment.

- Any auto-review on the head has finished, and each finding is fixed or
  dismissed with a written reason.
- If the repo has `.macroscope/check-run-agents/*.md`, read those rules before you
  reply to a Macroscope finding. Its findings cite them.
- CodeRabbit is advisory. Treat its findings like any other finding, but never
  wait for it and never hold a merge for it.
- Keep the PR at the size the goal needs. Review loops tend to grow a PR one
  cheap-looking fix at a time. Fix real bugs in the intended case. Decline the
  rest with a written reason and list them as follow-ups: unneeded tests, guards
  for edge cases that do not matter, extra integration or compatibility, and small
  performance costs. Prefer the fix that deletes or narrows code.
- If nothing has changed, stay quiet rather than posting filler comments.
- If an overlapping PR makes this one obsolete, stop, report it to the user, and
  ask before closing the PR.

## Watching

Use one watcher per PR. Never write your own polling loop or `sleep` loop, and
never run two watchers on the same PR.

- In T3 Code, handle the existing checks and comments first. Then call
  `watch_pull_request` and end your turn. T3 Code wakes you when a check fails,
  the required checks pass, someone comments or reviews, or the branch conflicts.
- Outside T3 Code, poll the head check-runs with `gh api` in one background
  command at a modest interval. Never run a tight loop.
- Only the thread that owns the PR watches it. If you are a delegated child task
  (a reviewer or any `delegate_task` child), never call `watch_pull_request`.
- A wake is news, not a merge decision. Check the merge gate yourself.
- Unwatch when you hand off. Call `unwatch_pull_request` before you stop, whether
  the PR is ready, blocked, or you are asking the user. A merge or close ends the
  watch on its own.
- Use `gh api repos/<owner>/<repo>/pulls/<n>` and
  `gh api repos/<owner>/<repo>/commits/<sha>/check-runs` to read the PR and its
  checks. `gh pr view` and `gh pr checks` use GraphQL, which runs out first.
- "Stopped watching, could not read it" usually means the GitHub rate limit is
  used up. Run `gh api rate_limit --jq .resources.graphql` before you watch again.
  If `remaining` is near 0, wait for `reset` with one background command, then
  watch again. Never watch again in a tight loop.
- Before a merge, run `git fetch` and check the PR again. Never merge when it is
  BEHIND or DIRTY, or when base commits that landed after the last green run touch
  this PR's files. Rebase, push, and wait for green again.

## Merge gate

Full confidence means all of these are true on the latest commit:

- Your proof covers every change in the diff.
- Trust the forge's verdict, not a green check list. Ready means the forge agrees
  the PR can merge: read the status of the current head SHA and require the
  required check to be success. After any amend, rebase, or base update, rerun CI.
  Exit 0 is not the gate.
- Any auto-review on the head has finished, and each finding is fixed or dismissed
  with a written reason.
- No human review still requests changes.
- The branch is up to date with its base, and GitHub reports it as mergeable.

Merge with the method the repo allows. If its docs do not say, read
`allow_squash_merge`, `allow_merge_commit`, and `allow_rebase_merge` from
`gh api repos/<owner>/<repo>`. Never use `--admin` or other bypasses. Merge only through
branch protection as configured.

## Comment format

Before you post any comment on the user's behalf, read
`/Users/home/.agents/skills/full-send/references/comment-format.md` and follow it.

## Asking the user

A question in a normal message goes unseen, and the thread looks done. To ask the
user something, use the harness's question tool so the thread shows that it is
waiting: `AskUserQuestion` in Claude Code, `request_user_input` in Codex, or the
question tool your harness has. Put all the context in the question: the PR link,
what blocks you, and the options with your recommendation first.

Ask only when you cannot continue alone:

- You cannot prove the change.
- The merge gate cannot pass, and you cannot fix that yourself.
- The next step is outside the grant above: deleting data, touching production,
  changing things outside the repo, closing a PR, or force-pushing any other
  branch.
- The goal is ambiguous in a way that changes the result.
- Another PR makes this one obsolete.

Decide everything else yourself with sensible defaults, and list those decisions
in the report.

If no question tool is available, or the tool returns no answer, take the safe
path: leave the PR open and unmerged, and start your final message with the
question.
