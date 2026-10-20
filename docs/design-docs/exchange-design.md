# Exchange design decisions

The product and architecture decisions for the paper-money perpetual-futures exchange, with the reasons behind them and the options rejected. Decisions that are hard to reverse have their own ADRs in [`docs/adr/`](../adr/) and are only linked here. How the mechanics work (margin, mark price, funding, liquidation) is explained in the [perpetual-futures primer](../domain/perpetual-futures-primer.md). Vocabulary follows [`GLOSSARY.md`](../../GLOSSARY.md).

## Product

- **What:** a perpetual-futures exchange modelled on the reference exchange, Hyperliquid. Building all of Hyperliquid (its own L1 and consensus, on-chain order book, spot, vaults, bridge, HyperEVM, token launches) would take years. The scope below keeps the parts that make it interesting: the matching engine and the margin and liquidation maths.
- **Off-chain, paper money now; testnet later.** See [ADR-0001](../adr/0001-off-chain-paper-exchange-with-custody-seam.md). Real money was rejected outright.
- **Users:** the human, plus bot traders that trade continuously. Friends may be invited to paper-trade after week 2. The bots supply the "real users" pressure that surfaces bugs, including while the human sleeps.
- **Prices and liquidity:** oracle prices and market parameters come from the reference exchange's public API. The order book is filled by our own market-maker bots quoting around the oracle price. Rejected: fully synthetic prices (unrealistic for liquidations and funding), and filling orders against snapshots of Hyperliquid's own order book (that would make our matching engine a wrapper around theirs).
- **v1 markets:** BTC and ETH perps only. Two markets are the minimum that proves cross margin. Their lot size and max leverage are mirrored from the reference exchange (BTC 40×, sizes to 5 decimals; ETH 25×, sizes to 4 decimals). Each market's tick is fixed ([ADR-0002](../adr/0002-integer-ticks-and-lots.md)). More markets are a configuration change later.

## Trading rules

- **Order types in v1:** GTC, IOC, ALO (post-only), and market orders, which are IOC orders with a slippage cap. **Reduce-only** arrives with Liquidation in week 2, since liquidation needs reduce-style closing anyway. **TP/SL** trigger orders are deferred: they need a separate trigger engine watching mark price.
- **Margin:** cross margin only. Copy the reference exchange's rules with **one tier per market**: initial margin is notional divided by the account's chosen leverage for that market, and maintenance margin is half the initial margin at the market's maximum leverage (1.25% for BTC, 2% for ETH). Tiered margin tables only matter for positions far larger than any paper account will hold. Isolated margin is deferred.
- **Mark price is computed locally**, roughly the way Hyperliquid does it (a median of: the oracle price plus a moving average of the book's premium, the median of best bid, best ask and last trade, and an external reference, for which we use Hyperliquid's own mid price). Rejected: copying Hyperliquid's `markPx`, because our order book could trade far from it and liquidations would fire on prices our users never saw. Also rejected: mark price equal to oracle price, because then our order book never affects profit, loss or liquidations.
- **Conservation:** the paper balance of all accounts, the Exchange account included, always equals Deposits − Withdrawals. Money is whole micro-USDC and every rounding favours the exchange, with the leftover credited to the Exchange account, so this holds exactly ([ADR-0004](../adr/0004-money-in-micro-usdc-rounded-toward-the-exchange.md)).
- **Fees:** none in v1. In week 3, fees (base tier ≈ 0.045% taker, 0.015% maker) are paid into the Exchange account, which keeps conservation intact. Fees that simply vanish were rejected because they would break conservation.
- **Accounts:** no login in v1; the human picks an account ID locally. Every new account gets a 100,000 paper USDC Deposit. Bots are accounts too. Simple login (e.g. GitHub sign-in or magic links) arrives in week 3, before friends join. "Paper balance" stays the term; paper USDC is its unit.

## Architecture

- **Prices and sizes:** whole numbers of ticks and lots as `bigint`, never floating point. See [ADR-0002](../adr/0002-integer-ticks-and-lots.md).
- **Money:** every amount of money is a whole number of micro-USDC as `bigint`, each market's tick × lot is a whole number of micro-USDC, and rates and ratios are fixed-point `bigint`. See [ADR-0004](../adr/0004-money-in-micro-usdc-rounded-toward-the-exchange.md).
- **State:** a deterministic in-memory engine fed by an append-only event log, with read models projected into SQLite. See [ADR-0003](../adr/0003-deterministic-engine-with-event-log.md).
- **Stack:** TypeScript everywhere, in strict mode, with Vitest. One language means every harness capability (lints, structural tests, replay tools) is built once and applies everywhere. Rejected: a Rust matching engine (two toolchains, and performance is irrelevant at paper scale) and a Python backend (two languages, weaker types for layering rules). The testnet contracts will be Solidity whatever the stack.
- **Repo layout:** a pnpm workspace with three packages. The exchange app holds the engine, Clearinghouse, API, Oracle feed and Bots. The web app is the trading UI (React + Vite). The protocol package holds the API's request and response shapes as Zod schemas, shared by server and UI and parsed at the boundary. Rejected: one package per domain, because dependency rules then become cross-package rules, which are harder to enforce. Three packages is not a real monorepo, so the glossary stays single-context.
- **Domains** inside the exchange app: Markets, Oracle, Order book, Pricing, Clearinghouse, Funding, Liquidation and Bots.
- **Bots are API clients.** They talk to the exchange only through the protocol package's API, exactly as the trading UI does, so their orders enter the event log as ordinary events and their randomness and timers never reach the deterministic engine. They import no other domain. Rejected: Bots calling Services directly, which would bring randomness into the engine and leave the API untested by the bots' constant traffic.
- **Layers:** every domain uses the fixed layers Types → Config → Repo → Service → Runtime → UI, and may only depend forward along that chain. With the event-log design, "Repo" means reading and writing the event log and read models.
- **Providers:** cross-cutting concerns (logging, the clock, randomness, the reference-exchange connection) enter only through the Providers interface.
- **Determinism rule:** Service-layer code never reads the wall clock, generates randomness, uses timers or touches the network or files. Time and randomness arrive as events. A lint enforces this, because replay depends on it.
- **Custody seam:** paper balance enters and leaves the Clearinghouse only through Deposit and Withdrawal (ADR-0001). A structural test enforces this.

## Seeing the running system (application legibility)

These are needed from the first runnable server. Everything heavier waits for a harness gap.

- Structured JSON logs, written to a file per run.
- An `exchange` command-line tool to replay the event log to any point and inspect state, e.g. an account's margin at event N, or why it was liquidated (issue #9).
- Playwright for driving and testing the UI.
- Ports taken from environment variables, so several worktrees can run the app at once.

Deferred until a gap asks: the article's per-worktree Loki/Prometheus stack and Chrome DevTools wiring.

## Testing strategy

In order of weight:

1. **Unit tests** for each Service.
2. **Property-based tests** (`fast-check`) for rules that must always hold: the order book never crosses, paper balance is conserved, and funding sums to zero across accounts, the Exchange account included.
3. **Replay tests:** a saved event log goes in and the expected state comes out. A failing property's input becomes a replay test.
4. **End-to-end tests** (Playwright) for only 2–3 key UI flows.

The harness itself is tested through one seam: the repo-wide check command. Fixtures that break each architecture rule must fail the check with a remediation message, and clean fixtures must pass (see issue #1).

## Deferred decisions

- Where the exchange is hosted when friends join (decide in week 3).
- Which testnet chain and contract design (decide in week 4).
- Bot strategies: spreads, sizes, taker frequency (decide in the bot tickets' specs, #12 and #15).
- Who covers a liquidation that leaves negative account value (bad debt): for example the Exchange account acting as a backstop, as Hyperliquid's liquidator vault does, or auto-deleveraging profitable positions. Decide in the Liquidation spec (#14); it affects the conservation invariant.
- Isolated margin, TP/SL orders and more markets (not scheduled).
