# practice-harness

A four-week practice run of harness engineering. The product is a paper-money perpetual-futures exchange modelled on Hyperliquid. Humans steer through prompts and reviews; Claude Code writes everything, and nothing hand-written enters the repo.

The real output is the harness: what the agent needs to work reliably on its own. Two measures track it:

- the `harness-gap` issue log: each time the agent got stuck, and the capability added to fix it
- unattended runtime per task

## The exchange

- Off-chain and paper money only: every new account gets a 100,000 paper USDC deposit. No real funds are involved.
- v1 markets are BTC and ETH perpetuals, with oracle prices taken from Hyperliquid.
- Cross margin, Hyperliquid's margin rules (one tier), and GTC, IOC, ALO and market orders.
- A deterministic engine fed by an append-only event log, with read models in SQLite.

## Status

No code yet. Work follows the [roadmap issue (#3)](https://github.com/maybe1046/practice-harness/issues/3): each ticket is one of its sub-issues, and "blocked by" links set the order.

## Where to read

| File | What it holds |
|---|---|
| [`GLOSSARY.md`](GLOSSARY.md) | The project's vocabulary: one definition per term |
| [`docs/domain/`](docs/domain/) | How the exchange's mechanics work, with worked examples; start with the [perpetual-futures primer](docs/domain/perpetual-futures-primer.md) |
| [`docs/adr/`](docs/adr/) | Decisions that are hard to reverse, and why they were made |
| [`docs/design-docs/harness-experiment.md`](docs/design-docs/harness-experiment.md) | The experiment's rules, measures, gap routine and how work flows |
| [`docs/design-docs/exchange-design.md`](docs/design-docs/exchange-design.md) | Product and architecture decisions, with reasons and rejected options |
| [`docs/references/harness-engineering-article.md`](docs/references/harness-engineering-article.md) | Summary of the OpenAI article the project is based on |
| [`AGENTS.md`](AGENTS.md) | Instructions for the agent: issue tracker, triage labels, domain docs |
