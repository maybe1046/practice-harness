# practice-harness

## Start here

- Nothing hand-written enters this repo: you write everything, and the human only prompts and reviews. Rules, measures and the `gap` routine: `docs/design-docs/harness-experiment.md`.
- Product and architecture decisions: `docs/design-docs/exchange-design.md`, the ADRs in `docs/adr/`, and the vocabulary in `GLOSSARY.md`.
- Work comes from the roadmap, issue #3: take the first open, unassigned sub-issue with no open blockers. Every change goes through a PR.

## Agent skills

### Issue tracker

Issues live in GitHub Issues on `maybe1046/practice-harness`, managed via the `gh` CLI. See `docs/agents/issue-tracker.md`.

### Triage labels

Default vocabulary: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context: one `GLOSSARY.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.
