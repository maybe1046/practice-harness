# The harness experiment

How this project runs: its purpose, its rules, how progress is measured, and how an agent continues the work. Vocabulary follows [`GLOSSARY.md`](../../GLOSSARY.md). The idea comes from OpenAI's harness-engineering article, summarised in [`docs/references/harness-engineering-article.md`](../references/harness-engineering-article.md).

## Purpose

Learn harness engineering by building a real product, then extract a reusable harness kit from what worked.

- **Weeks 1–3:** build the product (a paper-money perpetual-futures exchange, see [`exchange-design.md`](exchange-design.md)). The harness grows from the friction the agent hits.
- **Week 4:** retrospective on the harness gap log, then extract the kit. A kit designed before hitting real friction would be guesswork, so extraction waits.

The product is a means to an end, but it must be used for real, by the human and by bot traders. Real use is what surfaces real gaps.

## Rules

1. **Nothing hand-written enters the repo.** The human writes prompts and review comments only. Code, tests, CI, docs, `AGENTS.md`, commit messages and harness fixes all come from the agent. Anything typed into the repo by a human counts as hand-written, even a one-line fix. The only exceptions are the first `README.md` commit and the GitHub setup, both done before this rule existed.
2. **When the agent is stuck, the fix is a capability, never "try harder".** Ask: what capability is missing, and how do we make it legible and enforceable for the agent?
3. **The repository is the system of record.** Anything the agent cannot reach while running doesn't exist for it. Decisions made in chat must land in the repo or the issue tracker. (This document exists because of that rule.)
4. **Enforce invariants, not implementations.** Mechanical rules (lints, structural tests) guard boundaries. Inside them, the agent chooses how to express solutions.
5. **Grow the harness only on demand.** A small core is built upfront (issue #1). Everything else is added only when a harness gap asks for it. Building the article's full apparatus upfront would hide the friction the experiment exists to find.

## Agent

- **Claude Code** is the only agent for now. A second agent (e.g. Codex) could later test whether the harness carries over. `AGENTS.md` is the real instruction file; `CLAUDE.md` contains only `@AGENTS.md` so Claude Code loads it.
- **Interactive first, unattended later.** Sessions start interactive. Once the harness is reliable (target: week 2, issue #17), runs move to unattended headless mode (`claude -p`, `/loop`, several worktrees in parallel). Running unattended before the harness exists just produces silent overnight failures.
- **CI-driven agents** (e.g. responding to `@claude` on GitHub) are deliberately not set up. Add them only when a harness gap asks.

## Measures

The experiment is time-boxed to **4 weeks**. It tracks two things, and deliberately not PR throughput, which in a solo repo mostly measures how often the human prompted.

- **The harness gap log:** GitHub issues labelled `harness-gap`. Each records what the human had to do and the capability that would have prevented it. Each one is closed by the PR that adds that capability. The log is `gh issue list --label harness-gap --state all`.
- **Unattended runtime:** how long the agent works on one task before a human has to step in. It is calculated from Claude Code's own session transcripts, which are timestamped, by a script the agent writes in week 3. Until then, the first time the number is needed is itself a harness gap.

## The harness gap routine

When the human has to step in:

1. The human says **`gap`** plus one line about what was missing.
2. The agent opens a `harness-gap` issue: what the human had to do, and which capability would have prevented it.
3. The agent resumes the original task.
4. The fix comes later as a separate PR that references the gap issue.

Issue #1 writes this routine into `AGENTS.md`.

## How work flows

- **Issues and ExecPlans.** A GitHub issue says *what* and *why*, with acceptance criteria. An ExecPlan says *how*, and is used only for work spanning several PRs. Issue #1 defines the ExecPlan format in `docs/PLANS.md`, adopted from OpenAI's ExecPlan guide. ExecPlans live in `docs/exec-plans/active/` and move to `completed/` when their issue closes.
- **Roadmap.** [Issue #3](https://github.com/maybe1046/practice-harness/issues/3) is the roadmap, labelled `wayfinder:map`. Its sub-issues are the tickets (`wayfinder:task`), and GitHub "blocked by" links set the order. **Next ticket** = the first open, unassigned sub-issue of #3 with no open blockers. Week 3–4 tickets are created at the end of week 2.
- **Ticket lifecycle.** Only issue #1 has a full spec so far. Every other ticket needs `/to-spec` first, which publishes a spec and labels it `ready-for-agent`. Then `/implement #N` builds it. Specs name test seams; the default is the highest single seam available.
- **Every change goes through a PR.** Issue #1 adds a pre-commit hook kept in the repo (`.githooks/`, switched on by a pnpm `prepare` script) that blocks direct commits to `main`, and branch protection on GitHub. A change reaches `main` only through a PR with a passing check.
- **Who merges.** In weeks 1–2 the human reviews and merges. From week 3, the agent squash-merges its own PRs once checks are green, and the human spot-checks afterwards. The agent reviews its own PR with the `code-review` skill before asking for a merge.
- **Merge style.** PRs are squash-merged. Merged branches can be deleted on GitHub (`git push origin --delete <branch>`). Locally, `git branch -d` reports squash-merged branches as "not fully merged" even when their content is identical to `main`.
- **Commit attribution.** Agent commits end with a `Co-Authored-By: Claude …` line, and agent PR descriptions end with the Claude Code attribution line.

## The GitHub wiki mirror

The [wiki](https://github.com/maybe1046/practice-harness/wiki) mirrors `GLOSSARY.md` and the ADRs for human reading. Each mirrored page starts with a note saying the repository copy is the source of truth.

- It is synced **by hand**: clone `https://github.com/maybe1046/practice-harness.wiki.git` (branch `master`), copy the changed files with the mirror note on top, then commit and push.
- The pages are `Home` (an index), `Glossary` and `ADR-<number>-<slug>` for each ADR. A new ADR needs a new page and a Home link.
- Re-syncing has been needed once already (after PR #2). If it keeps costing effort, file it as a `harness-gap`; the fix is a CI job that syncs the wiki on every merge to `main`.

## Environment notes

- The human develops on **Linux, WSL or macOS**; CI runs on Linux. Anything in the check path (scripts, hooks) is written as Node scripts, never shell-specific or GNU-only commands, so it behaves the same on all of them.
- The repo is **public**. No secrets go into the repo or into workflow files.
- `gh` is logged in as the repo owner (`maybe1046`). On the WSL machine it is installed from the official release binary into `~/.local/bin`, because `sudo` needs a password there. Pushing changes to `.github/workflows/` needs the `workflow` scope (`gh auth refresh -s workflow`). Labels in use: the five triage labels, `harness-gap`, `wayfinder:map` and `wayfinder:task`.
- Hyperliquid's public `info` endpoint needs no authentication. A POST with `{"type":"metaAndAssetCtxs"}` returns every perp market's `szDecimals`, `maxLeverage`, `oraclePx`, `markPx` and `funding`.

## Suggested skills for a new session

- `/implement #N` to build a ticket that has a spec (uses `tdd` at the spec's seams).
- `/to-spec` to turn a roadmap ticket into a `ready-for-agent` spec.
- `/grill-with-docs` to stress-test a design and record the outcome in the glossary and ADRs.
- `/code-review` to review the agent's own PR before merging.
- `/triage` for incoming issues; `/wayfinder` when a large foggy piece of work needs mapping.
