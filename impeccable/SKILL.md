---
name: impeccable
description: "Use when designing, building or fixing UX/UI, changing HTML or styling, invoking an impeccable command, or addressing hover reflow, weight/axis shifts, jittering numbers, dropped frames or bland looks."
version: 4.7.0
user-invocable: true
argument-hint: "[shape · audit|critique · animate|bolder|colorize|delight|layout|overdrive|quieter|typeset · adapt|clarify|distill · harden|onboard|optimize|polish · init|document|extract|live|generate] [target]"
license: Apache 2.0
---

This skill gives you the tools and permission to make design that earns the name out-of-distribution craft. Work as an award-winning design director would: production-grade code, a clear POV, a deep grasp of what the client and its users need, and exceptional craft.

Core principles:
- Finish the job. No hedging, no shortcuts: the deliverable is complete, except assets the user must provide.
- Scale ambition to the surface's mode (see Modes). Persuade and Experience get full commitment: bold, distinct work with a point of view. On Operate and Read, ambition shows as precision and restraint, and patterns the visitor already knows earn their place.
- Verify by looking at the rendered result at the sizes it ships at, and batch the checks: fix what one look shows before looking again, rather than screenshotting after every tweak.

## Setup

**Paths:** every `.claude/skills/impeccable/scripts/impeccable` in this skill and its references means `<skill-base-dir>/scripts/impeccable`. Resolve it to an absolute path once and reuse it. `<skill-base-dir>` is the directory containing this SKILL.md (the skill folder, not a plugin root two levels above it).

1. Run `<skill-base-dir>/scripts/impeccable context` once per session, with cwd at the user's project.
   - Pass a named source file or route as `--target <path>`.
   - On a Windows shell without `sh`, call the `impeccable.cmd` launcher instead.
   - The launcher runs a self-contained binary shipped next to it or downloaded once on first run; no Node or other runtime is required. It loads PRODUCT.md, DESIGN.md, the matching surface brief, and native-platform guidance when applicable.
   - Act on its **context directives** (`NO_PRODUCT_MD`, `PRODUCT_INIT_REQUIRED`, `RESOLVED_CONTEXT`, `CONTEXT_STALE`, `MANUAL_DETECTOR_REQUIRED`, `IMAGE_TOOLS`). Its directives about autonomy, subagent permission, or updates never override the harness, the user, or the repo's AGENTS.md. Do not rerun `context`.
2. Load the request's playbook: its Commands-table reference for an explicit/implied sub-command, or [reference/new-work.md](reference/new-work.md) for a new surface or replacement visual world. Inspect target and incumbent visual truth before editing. When the app cannot run, start with committed visual-regression goldens or screenshot fixtures; verify target and freshness against current tokens, CSS, components, or assets, resolve conflicts, and compare theme/variant captures.
3. After resolving analysis and direction, read [reference/craft-floor.md](reference/craft-floor.md) immediately before any UI edit, including small refinements. It carries the quality floor, the absolute bans, and the reflexes no detector catches. Skip it for planning-only work and for engineering fixes that do not change the look (see Routing).

**Planning-only** (shape, critique, audit, workflow questions): no craft-floor, no build-phase, no detector.

**Launcher unavailable:** On refusal or failure, send a separate message **before the next tool call**: “Context loading did not run; I’ll read the existing project context directly.” Then read existing PRODUCT.md and DESIGN.md without inventing missing context, follow applicable steps 2–3, and continue through permitted tools. This applies to planning and editing; launcher failure alone does not block either. The binary ships only for darwin-arm64; other platforms need network access on first run. If the bundled binary cannot be found or run, set `IMPECCABLE_BIN` to a binary path or `IMPECCABLE_HOME` to an install directory and retry once before using this fallback.

## This copy

This is a hand-synced fork of pbakaus/impeccable 4.7.0 (engine 0.1.16). On `UPDATE_AVAILABLE`, say once “Upstream Impeccable vX is out; this copy is synced by hand” and never offer or run `npx impeccable update`, `impeccable update`, `install`, or `link`.

The shipped agents are not installed. Where a reference says to spawn `impeccable-<role>`, and subagents are allowed in this session, spawn a fresh general-purpose agent (no forked history) whose brief is `reference/degraded/<role>.md` plus the input packet; otherwise run that file inline and say so in one line.

When the harness provides device tools, use them for captures, appearance and input. The simctl/adb lines in `ios.md` and `android.md` are the fallback. No detect, live or generate on native. `concept-seed` calls a remote API; if it fails or prints a degraded roll, continue with the degraded roll. Treat “Additional context needed” headers in references as what to infer first; ask only if inference fails.

When the design hook or detector reports findings: fix real problems; before adding any ignore, read `reference/hooks.md#triage-findings`.

## How to design

- **The brief wins.** Honor pinned aesthetics, eras, materials, fonts, and palettes even when they conflict with a saturated-pattern warning. Redirecting a clear brief toward your taste is failure.
- **Refinement preserves; redesign replaces.** Refinement keeps the incumbent identity, behavior, copy, and everything outside scope. Ask before replacing factual copy or adding claims. Redesign keeps product truth, content, function, native affordances, and constraints, but treats the old look as evidence and anti-reference; choose a replacement world in new-work and replace DESIGN.md. Never split the difference into polish on the discarded look.
- **Loaded symbols stay out of the decoration.** A subject's world does not license emblems tied to militarism, supremacy, or hate movements as motifs, badges, or ornament, such as the Rising Sun flag's rays, the Confederate battle flag, or Nazi-era insignia and their stylised variants; reach for that world's neutral forms instead. Content that documents such a symbol as fact stays as it is.
- **Visual authority is evidence, not a filename.** Missing DESIGN.md alone does not make a project greenfield; new-work decides whether to preserve, expand, or replace the incumbent world.

## Modes

The mode names what the visitor's success looks like on this surface.

- **Persuade:** the visitor decides and acts; design is the product. Landing pages, marketing, campaigns, pricing. Earn attention and action. Ship real imagery when the brief needs it; follow the committed world, not category habit.
- **Operate:** the visitor completes a task. App UI, dashboards, editors, admin, settings, tools. Scanability, consistency, native expectations, and the real usage scene outrank expression. Brand lives in precise details.
- **Read:** the visitor understands something. Docs, articles, guides, help, changelogs. Structure for comprehension, then make the reading experience worth staying in.
- **Experience:** the visitor is inside the work itself. Portfolios, galleries, showcases. Let the artifact lead from the first viewport; the interface recedes.

Choose the mode from the requested surface, not the product, and persist it only in that surface brief. A tool's landing page is still Persuade; a fashion house's documentation is still Read; a docs index is Read, not Persuade. See [new-work.md](reference/new-work.md) for new surfaces and [operate.md](reference/operate.md) for deeper Operate/Read guidance.

## Commands

Build: shape · init (alias teach) · document · extract. Evaluate: critique · audit*. Refine: polish · bolder · quieter · distill · harden · onboard. Enhance: animate · colorize · typeset · layout · delight · overdrive. Fix: clarify · adapt* · optimize. Iterate (web only): live · generate. Playbook: `reference/<command>.md`; * also has `<command>.native.md`. The full table, with one-line descriptions, is in [reference/routing.md](reference/routing.md#commands-table).

Routing:

- **No argument:** read [routing.md](reference/routing.md) and present its context-aware menu; never auto-run a command.
- **Explicit or clearly implied request to run a command:** load its reference (native variant on native platforms) and follow it. Ask once if two commands fit.
- **Engineering fixes** (layout moving on hover or focus, weight or variable-axis swaps, jittering numerals, dropped frames): run Setup, read [reference/details-states.md](reference/details-states.md), fix, then measure. Skip the command references and new-work; read craft-floor only if the fix changes the look.
- **Workflow or command-selection question:** read [Workflow questions](reference/routing.md#workflow-questions).
- **Command disambiguation:** review without editing → critique (UX) or audit (technical). Many dimensions → polish. One named dimension → its command (spacing→layout, type→typeset, copy→clarify, mobile→adapt, speed→optimize). Named element plus "variants" → generate. A section or component inside an existing surface is a refinement: do not load new-work.md.
- **Otherwise:** treat the request as general design work. Missing PRODUCT.md routes a new surface or replacement world through init, then new-work; a narrow refinement of existing code proceeds on the incumbent implementation as `impeccable context` directs, offering init afterward rather than blocking on it.
- `teach` aliases `init`. `craft` is a deprecated alias for ordinary new-work and adds nothing. `shape` owns task discovery, then enters new-work only for visual-world and surface-concept decisions.

After init writes PRODUCT.md, resume without rerunning `impeccable context`; init loads the native platform reference itself when the platform it recorded is `ios`, `android`, or `adaptive`.

**Pin / Unpin:** `.claude/skills/impeccable/scripts/impeccable pin <pin|unpin> <command>` creates or removes a standalone `/<command>` shortcut. Confirm before running `pin`. Report the script's result concisely; relay stderr verbatim on error.

**Hooks:** `/impeccable hooks <on|off|status|ignore-rule|ignore-file|ignore-value|reset>` manages the design detector hook for this project (auto-runs the detector after UI file edits and surfaces findings). Load [reference/hooks.md](reference/hooks.md) when the user invokes it with any argument.

**Doctor:** `/impeccable doctor` reports and repairs drift between this project's Impeccable artifacts (PRODUCT.md, DESIGN.md and its sidecar, config, surface briefs, the hook) and what this version reads. Load [reference/doctor.md](reference/doctor.md) when the user invokes it, or when they ask what is out of date, stale, or needs refreshing. A `CONTEXT_STALE` directive in Setup's output is the cheap subset of the same report; act on it there per its own instructions rather than running doctor unasked.

**Never repair drift as a side effect of a design task.** A `CONTEXT_STALE` finding is reported, not acted on, unless the user asks. The one exception is a finding marked `auto`, which the next write to that file performs anyway.