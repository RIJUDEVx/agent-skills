---
name: pr
description: Opens, updates, dresses, and summarises pull requests. Use whenever a branch is pushed for review, a PR is created or edited, or someone asks to open, update, dress, review, or summarise a PR. Reads the repo's own conventions (base branch, PR template, title format, labels, checks) and follows them instead of inventing one. Make sure to use this skill even when the user says only "open a PR" or "update the PR"; never freehand a title or body when the repo documents a format.
---

# Pull request

The repo decides the conventions. This skill finds them and follows them. A repo may
override any of this with its own `.agents/skills/pr/SKILL.md`; that file wins.

## 1. Preflight read

Before writing anything:

- **Repo rules.** `AGENTS.md`, `CONTRIBUTING.md`, `.github/pull_request_template.md`,
  and the repo's own `.agents/skills/pr/SKILL.md` if it exists. Read them; a source you
  have not read does not count.
- **Base branch.** The repo says it (`Dev`, `main`, `develop`), otherwise the remote
  default: `git symbolic-ref --short refs/remotes/origin/HEAD`. Never guess.
- **Existing PR.** `gh pr view --json title,body,labels,assignees,projects,isDraft`.
  Update it rather than opening a second one.
- **The change.** `git diff <base>...HEAD` and `git log <base>..HEAD --oneline`.
- **The issue behind the branch,** when one exists. Fetch it; never invent an issue.
- **Reverse state.** Archive ↔ restore, open ↔ close. When the change has one, both
  directions are implemented and verified, and the body says so.

## 2. Conventions

Read them, do not assume them:

| What | Where it lives |
|---|---|
| Title format | `CONTRIBUTING.md`, the PR template, `git log --oneline -20` |
| Body sections | `.github/pull_request_template.md` |
| Labels | `gh label list` is the source of truth for the vocabulary |
| Boards / projects | `gh project list --owner <owner>` |
| Checks | the repo's CI workflow, `AGENTS.md`, or `CONTRIBUTING.md` |

Where the repo documents a format, follow it exactly, including its prefixes and type
words. Where it documents nothing, use the defaults below.

## 3. Title

Default, when the repo documents nothing: `<type>: <description>`.

- Type: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `perf`.
- Description: the change in one clause, imperative mood, no trailing period.

The title reaches the changelog, so name the product change, not the files. "Reports now
filter by owner" lands; "refactored report filter" does not.

*Done when:* the title matches the format the repo documents.

## 4. Body

Fill the repo's template. Keep every heading it defines; drop only the sections that
genuinely do not apply. Write for a reviewer who knows the product but not the codebase.

- **Summary** — the problem or need, then the solution. Two to four plain sentences.
- **Changes** — one bullet per meaningful change. Skip for a small PR.
- **How can I test it?** — the steps a reviewer follows by hand and what they should see.
  Manual verification only; check results already live in the CI UI.
- **Screenshots or demo** — required for any UI change. See step 6.
- **Notes for reviewers** — root cause for a fix, why this approach over the alternatives,
  reverse state. One short paragraph each.
- **Anything to know before release?** — migrations, flags, and config or data updates.
  Drop it when there are none.

Name things in the product's own words — `Record inspector`, `Invoice template` — not the
code's. "The record page now groups fields in a right-hand inspector" lands; "extracted a
`RecordInspector` component" does not. "Notes for reviewers" is the one section allowed
engineering vocabulary.

### Traceability

Every linked issue gets exactly one keyword. The PR solves the issue or it does not;
extra cleanup in the same PR does not change that.

- **`Closes #N`** (`Fixes` / `Resolves`) — this PR solves the issue and closes it on
  merge. Use it even when the PR goes past the minimal fix.
- **`Refs #N`** (`Related to`) — the PR partially addresses, builds on, or merely
  references an issue that stays open.

Put it in the summary so it is unambiguous; several issues go on one comma-separated
line. A PR does not need an issue invented for it.

Write the prose through `unslop`. No filler, no title-case headings the repo did not ask
for, no restating the diff.

*Done when:* the body keeps the template's headings and the issue keyword is unambiguous.

## 5. Labels, assignee, board

- One label for the kind of change, plus one area label for **each** area the change
  touches. `gh label list` gives the exact vocabulary; a shared package change takes its
  shared label **and** the product labels it affects.
- Set the assignee (`--assignee @me`) when the repo has one person per area.
- Add the board when the repo uses them: `gh project list --owner <owner>` is the source
  of truth for the numbers. A cross-cutting change lands on every board it affects.
- Never invent a label or a board number. If the repo has no label scheme, ship without
  labels rather than guessing.

*Done when:* the labels and boards match what the repo actually defines.

## 6. Screenshots

- Quick before/after: `--attach` the files, alt text after `#`, and put a matching
  `![...](./after.png)` in the body so `gh` rewrites it to the upload URL.
- Formats, limits, and edge cases are in the `gh` skill under "Attaching images and
  videos".
- The screenshot section is mandatory for any UI change, and every changed surface gets
  a capture.

*Done when:* every changed UI surface has a before/after capture.

## 7. Readiness gate

Push the tested commit before checking. Checks attach to the remote SHA.

- Run the repo's documented CI command, not a generic one.
- Mark ready only on green: `gh pr ready`. A pending or failing check stays a draft.
- Trust the forge's verdict, not a green checklist. Ready means GitHub agrees the
  required check succeeded on the pushed SHA.
- The body points at behaviour; it does not restate check results.

*Done when:* the draft is marked ready only after the required check is green on the
head SHA.

## 8. Updating an existing PR

- `gh pr edit` for title, body, labels, boards, and the issue link.
- Preserve existing screenshots; only add new ones.
- Re-check the invariants in steps 3–7 after every edit, including the issue keyword: a
  PR that grew extra cleanup still `Closes` its issue.
- In T3 Code, call `link_pull_request` after `gh pr create`, and `watch_pull_request`
  when you need to wait on checks. The `t3-tools` skill has the exact tool names.

*Done when:* the edit still satisfies steps 3–7 and nothing required was dropped.

## Guards

- The draft is the default state; readiness waits on a green check.
- Merge ownership belongs to the workflow that owns it. Mark ready, then stop, unless a
  skill with explicit merge authority (such as `full-send`) is running.
- The scope of the PR matches the app named in the title and the area labels applied.
- The body's issue keyword and the template headings are part of the definition of done.
