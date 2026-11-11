---
name: t3-tools
description: Use when the work involves T3 Code itself: linking or watching a pull request, delegating to another model or provider, waiting on a background command or test run, showing a chart, diagram or HTML visual in the thread, previewing a page or simulator, scheduling, or requesting a secret. Gives the exact T3 tool names and how to load or call them in this harness, so nothing is searched or guessed. Also use to explain visually, for an inline diagram, or when the user says show me.
---

# T3 tools

Reach for the tool named here; do not search or guess.

## Route per provider

- **Claude Code:** T3 tools are deferred. Load a family with one call:
  `ToolSearch` with `select:mcp__t3-code__<name>,...`. Then call
  `mcp__t3-code__<name>`.
- **OpenCode:** T3 tools are not top-level. Find one with `search({query: "<name>"})`.
  Then call the returned path inside `execute`:
  `await tools["t3-code-<threadId>"].<name>({...})`. Do not use `tools.shell`,
  `read`, `webfetch`, `mcp__t3_code__...`, or the dot form.
- **Codex:** T3 tools are direct. Call `t3-code.<name>`. No load step.

## Families

| Family | Claude `select:` value | Tools |
|---|---|---|
| Delegate | `mcp__t3-code__orchestrator_capabilities,mcp__t3-code__delegate_task,mcp__t3-code__task_status,mcp__t3-code__task_cancel` | `orchestrator_capabilities`, `delegate_task`, `task_status`, `task_cancel`, `create_threads` |
| Threads | `mcp__t3-code__t3_thread_search,mcp__t3-code__t3_thread_read,mcp__t3-code__t3_thread_list,mcp__t3-code__t3_thread_send,mcp__t3-code__t3_thread_wait,mcp__t3-code__t3_thread_launch` | `t3_thread_search`, `t3_thread_read`, `t3_thread_list`, `t3_thread_send`, `t3_thread_wait`, `t3_thread_launch`, `t3_thread_configure`, `t3_thread_update`, `t3_thread_organize`, `t3_thread_transfer`, `t3_thread_interrupt` |
| PRs | `mcp__t3-code__link_pull_request,mcp__t3-code__watch_pull_request,mcp__t3-code__unwatch_pull_request,mcp__t3-code__list_thread_pull_requests` | `link_pull_request(s)`, `unlink_pull_request(s)`, `list_thread_pull_requests`, `watch_pull_request`, `unwatch_pull_request` |
| Background | `mcp__t3-code__t3_background_wait` | `t3_background_wait` |
| Visuals | `mcp__t3-code__html_preview,mcp__t3-code__html_render` | `html_preview`, `html_render` |
| Browser | `mcp__t3-code__preview_open,mcp__t3-code__preview_navigate,mcp__t3-code__preview_snapshot,mcp__t3-code__preview_act,mcp__t3-code__preview_wait_for,mcp__t3-code__preview_status,mcp__t3-code__preview_evaluate` | `preview_*`, `t3_preview_list`, `t3_preview_close` (browser work only) |
| Device | `mcp__t3-code__device_list,mcp__t3-code__device_open,mcp__t3-code__device_screenshot,mcp__t3-code__device_close` | `device_*` (taps and text go through the `agent-device` command that `device_open` returns) |
| Schedule | `mcp__t3-code__schedule_task,mcp__t3-code__list_scheduled_tasks,mcp__t3-code__delete_scheduled_task,mcp__t3-code__run_scheduled_task_now` | `schedule_task` and the matching list, delete, and run-now tools |
| Secrets | `mcp__t3-code__request_secret` | `request_secret` |

For cloud routines or one-time scheduled agent runs, use the `schedule` skill
instead. Those are not T3 tasks.

## Choosing a model

Every sub-agent model comes from the user's T3 favourites, unless a skill names
one that is on that list. This is the user's standing request to set a model on
sub-agents, so it counts as asked. Do not fall back to the parent's model because
a harness rule says to set a model only on request.

1. Pick the tier the skill asks for from the table below. When a skill says
   "cheapest" use Fast, and when it says "strongest" use Strong.
2. Check the model with `orchestrator_capabilities`, passing its name as `search`.
   The default result is a short list, so a missing name there proves nothing.
3. Set it with `delegate_task` `target`: `providerInstanceId`, `model`, and
   `options` with the option shown in the table. Native sub-agent tools run on
   the parent's model, so use them only when that is the right pick.
4. If a model fails or is deprecated, take the next model in the same tier and
   say so. If the tier is used up, read `favorites` in
   `~/.t3/userdata/settings.json` and pick the closest match. Never go outside
   that list. Ignore `client-settings.json`, which is older.

### Tiers

| Tier | Use for | Models, in order (`providerInstanceId` / `model` / option) |
|---|---|---|
| Fast | Cheapest and fastest. Scouts, builders, mergers, `code-review` | `codex` / `gpt-6-luna` / `reasoningEffort: high`; `claudeAgent` / `claude-haiku-5-5` / `effort: medium` |
| Balanced | Medium reasoning, still fast. A job a Fast model got wrong, or that needs more thought than Fast gives | `opencode` / `opencode-go/mimo-v2.6-flash` / no option |
| Strong | Design and adversarial review | `claudeAgent` / `claude-opus-5-5` / `effort: medium`; `opencode` / `opencode-go/muse-spark-1.3-contributor` / `variant: high`; `opencode` / `opencode-go/deepseek-v4.1-flash` / `variant: high` |
| Cheap strong | Strong work that runs many times, such as review rounds | `opencode` / `opencode-go/deepseek-v4.1-flash` / `variant: high`; `opencode` / `opencode-go/muse-spark-1.3-contributor` / `variant: high` |

Opus, Muse Spark, and DeepSeek are three different model families. When a skill
asks for models from different families, take them from Strong.

## Parameters that fail when wrong

| Tool | Rule |
|---|---|
| `html_preview` | `html` is the page text, not a `path`. |
| `html_render` | `title` and `height` are required. |
| `delegate_task` | `mode` is `"async"` or `"wait"`. Use a distinct `clientRequestId`. |
| `preview_resize` | `preset` is a device name such as `iphone-se`. |
| `run_scheduled_task_now` | Pass `taskId`, not `scheduledTaskId`. |
| `t3_thread_read` | `afterPosition` is 0 or greater. |
| `t3_queue` | Use `action` (`list`, `read`, `edit`, `cancel`, `reorder`, `promote_to_steer`). |

## Visuals

Show a chart, diagram, or page in the thread: call `html_preview` with `html`,
then `html_render` with `title` and `height`, before the final reply. Keep the
visual inline in the reply. Never `open` an HTML file, and never start a server.
Write a PNG file only when the user asks for a file to post elsewhere.

## Background waits

Wait on a background command or test run with `t3_background_wait`, passing the
background task ids. On OpenCode, wait with `t3_thread_wait` (or
`t3_background_wait` if the search returns it). Never wait by polling with `Monitor`, `sleep`, or
`setTimeout`. `execute` has no `setTimeout` or `sleep`.

## PRs

Call `link_pull_request` with the PR URL right after `gh pr create`, before
anything else. Call `watch_pull_request` only when this thread owns the PR and is
babysitting it. Call `unwatch_pull_request` before any handoff or question.
