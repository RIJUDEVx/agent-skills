## Review loop

This is a workflow for making the change more correct before the PR, and
sometimes after. Run one outside reviewer, in a loop.
Make the smallest possible changes that answer the reviewer. Watch for scope
creep, and push back when a finding is wrong.

### Reviewer model

- In T3 Code, send each round to `opencode-go/deepseek-v4.1-flash` at `variant`
  `high`, through `delegate_task` with `target` set to provider instance
  `opencode`, that model, and that option. Record the model and variant.
- If that model fails, use the next model in the Cheap strong tier of the
  `t3-tools` skill (Muse Spark 1.3 Contributor at `variant` `high`). State the
  substitution in the final report. Do not block on it.
- Outside T3 Code, use the harness's native reviewer route.

### Budget

- Default: 3 rounds total. Round 1 is full. Later rounds check only changed code
  and open findings.
- Extend to 5 only for security, permissions, auth, data loss, or
  state/concurrency, while a blocker or should stays open with evidence, or when
  the user asks.
- Stop early if a round returns only nits. Declined nits and comment, doc, or
  format-only fixes need no new round.
- One reviewer only. The reviewer must not start subagents or reviewers.
- Keep a ledger: id, severity, outcome, usage if returned. Never guess it.

### Round brief

Run `git status --short`. Do not stash, discard, or include unrelated changes.
Find the base after `git fetch origin`: `git merge-base HEAD origin/<base>`. Use
one diff command for all rounds. State intent and out-of-scope in 1-2 sentences.
Run covering checks. Do not send code that the change breaks. Note missing or
pre-failing checks in the brief.

In T3 Code, send each round with `delegate_task` (`role: "review"`,
`mode: "async"`) and a new `clientRequestId`. Save the `taskId`. Do not use
`t3_thread_send` on the child thread for a review round. Do not edit scoped files
while review runs. Wait for the notice. Do not poll or start a second reviewer.
Timeout, error, or no reply is not approval. On failure, cancel, verify it has
stopped, and restart with the ledger.

Round 1 brief (reviewer children get no history):

```
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Out of scope: <scope> Checks: <results>
See: git diff <base> [-- <paths>]; git status --short (read untracked).
Read surrounding code. Find bugs, broken callers, mismatch with intent.
Skip style unless it hides a bug. Hold to the smallest change. Flag code,
tests, or guards the intent does not need. Other paths, rare cases, small
perf, extra compat: nits.
blocker = intended case wrong. should = real bug callers will hit, in intent. nit = rest.
Reply:
VERDICT: APPROVE or CHANGES
S1 [blocker|should|nit] <file>:<line> <claim>. Fix: <suggestion> (stable ids)
```

### Answer findings

Check each finding and its severity yourself. Another path, rare case, or small
cost is a nit, whatever label it came with.

- Agree: make the smallest fix. Prefer delete or narrow over a new lock, cache,
  helper, or option. Fix the case that happened. Add one focused test that fails
  without the fix, if the intent needs it.
- Disagree: say why, with evidence (file:line, test, output).
- Out of scope or nit: decline in one line. List it as a follow-up.

Update the ledger. Re-run checks and `git diff --shortstat <base>`. If review
grew the diff, cut first. Do not agree just to end the loop.

### Next round

Start one only if budget remains and a blocker or should is open, behavior code
changed, or approval does not cover the current behavior diff. Send the round 1
brief, the ledger (`fixed:` / `disagree:` / `declined:` with evidence), the same
base diff, and the change since the last round. Tell the reviewer: open ids and
changed code only. Mark each old id `resolved` or `still open: <reason>`.

### Stop and ship

Agreed: APPROVE on the current behavior diff, no blocker or should open, and no
concern of your own. Stuck: an item stays disputed after two replies, or the
budget ends. Show both sides in 2-3 lines each. The user decides. Budget end is
not approval.

Do not push during the loop. Push after agreement. Report: rounds, model, what
the reviewer changed, diff size before and after, fixed and declined findings
with reasons, usage if exposed (else say not), and whether it was pushed.
