# Reference: "Harness engineering: leveraging Codex in an agent-first world"

A summary of the article this project is based on: Ryan Lopopolo, OpenAI, 11 February 2026, <https://openai.com/index/harness-engineering/>. Quotes are verbatim. How this project applies the ideas is in [`docs/design-docs/harness-experiment.md`](../design-docs/harness-experiment.md).

## The experiment

Over about five months, an OpenAI team built and shipped an internal beta product with **0 lines of manually-written code**. The repo reached roughly a million lines and about 1,500 merged PRs. Three engineers (later seven) averaged 3.5 PRs per engineer per day, and throughput *rose* as the team grew. The team estimates it took about a tenth of the time hand-coding would have. Even the initial scaffold and the first `AGENTS.md` were written by Codex.

> Humans steer. Agents execute.

## Lessons

**The engineer's job changes.** Engineers design environments, specify intent and build feedback loops. Early progress was slow "because the environment was underspecified", not because the model was incapable. The team worked depth-first, building small blocks (design, code, review, test) and using them to unlock larger tasks. When something failed, the question was always:

> what capability is missing, and how do we make it both legible and enforceable for the agent?

Agents drive PRs through a review loop until all agent reviewers are satisfied, a "Ralph Wiggum Loop" (<https://ghuntley.com/loop/>). Human review is optional.

**Make the application legible.** Human QA became the bottleneck, so the running app was made readable to the agent:

- the app can boot once per git worktree;
- the Chrome DevTools Protocol is wired in, with skills for DOM snapshots, screenshots and navigation;
- each worktree gets an ephemeral logs/metrics/traces stack, queried with LogQL and PromQL.

That makes prompts like "ensure service startup completes in under 800ms" tractable. Single runs regularly last more than six hours.

**The repository is the system of record.**

> give Codex a map, not a 1,000-page instruction manual.

One big `AGENTS.md` failed for four reasons: it crowds out the task in context; when everything is "important", nothing is; it rots instantly; and it can't be checked mechanically. Instead, a short `AGENTS.md` (about 100 lines) is a table of contents, and a structured `docs/` holds the detail: `ARCHITECTURE.md`; design docs with an index, verification status and `core-beliefs.md`; exec plans (active, completed, plus a tech-debt tracker); generated docs; product specs; `references/*-llms.txt`; and top-level `DESIGN`, `FRONTEND`, `PLANS`, `PRODUCT_SENSE`, `QUALITY_SCORE`, `RELIABILITY` and `SECURITY` files. This gives **progressive disclosure**. Linters and CI check that docs are fresh, cross-linked and correctly structured, and a recurring "doc-gardening" agent opens fix-up PRs. Execution plans follow OpenAI's ExecPlan guide: <https://developers.openai.com/cookbook/articles/codex_exec_plans>.

**Agent legibility is the goal.**

> anything it can't access in-context while running effectively doesn't exist.

Slack threads, Google Docs and knowledge in people's heads are invisible to the agent. The team prefers "boring" technology the agent can model, and sometimes reimplements small dependencies instead of working around opaque upstream behaviour.

**Enforce architecture and taste mechanically.** Enforce invariants, not implementations. One example is "parse, don't validate" at boundaries (<https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/>), without mandating a library. Each business domain has fixed layers, **Types → Config → Repo → Service → Runtime → UI**, with forward-only dependencies. Cross-cutting concerns (auth, connectors, telemetry, feature flags) enter only through **Providers**. Custom linters and structural tests enforce this, plus "taste invariants": structured logging, naming conventions, file size limits and reliability rules.

> Because the lints are custom, we write the error messages to inject remediation instructions into agent context.

> enforce boundaries centrally, allow autonomy locally.

Human taste is fed back as doc updates or tooling: "When documentation falls short, we promote the rule into code".

**Throughput changes the merge philosophy.** The repo has minimal blocking merge gates and short-lived PRs; flaky tests get re-run rather than blocking. "Corrections are cheap, and waiting is expensive." The article adds that this "would be irresponsible in a low-throughput environment".

**"Agent-generated" means everything:** product code and tests, CI, release tooling, internal tools, docs, evaluation harnesses, review comments, repo scripts, and dashboard definitions. Humans prioritise work, turn feedback into acceptance criteria, and validate outcomes.

**Autonomy grows as the loop is encoded.** From one prompt, Codex can now validate the codebase, reproduce a bug, record a video of it, fix it, validate the fix by driving the app, record a second video, open a PR, respond to feedback, fix build failures, escalate only when judgment is needed, and merge. This "should not be assumed to generalize without similar investment".

**Entropy and garbage collection.** Agents copy whatever patterns already exist, including bad ones. The team used to spend every Friday (20% of the week) cleaning up "AI slop". They replaced that with **golden principles**: opinionated mechanical rules such as preferring shared utilities over hand-rolled helpers, and never probing data "YOLO-style". Recurring background agent tasks scan for deviations, update quality grades and open small refactor PRs, most reviewed in under a minute and automerged. Technical debt is treated "like a high-interest loan", paid down continuously.

## Open questions in the article

How architectural coherence evolves over years; where human judgment adds the most leverage; how the system changes as models improve.

> building software still demands discipline, but the discipline shows up more in the scaffolding rather than the code.
