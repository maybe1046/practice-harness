# practice-harness

A practice run of harness engineering: building a real product where humans steer and Claude Code writes everything, to learn which harness capabilities make agent work reliable. The product is a paper-money perpetual-futures exchange modelled on Hyperliquid.

## Language

### Harness

**Harness**:
Everything around the agent that lets it do reliable work in this repo — the knowledge store, tooling, mechanical constraints, and feedback loops — as opposed to the product itself.
_Avoid_: Scaffolding, framework, setup

**Harness gap**:
A missing capability that caused the agent to get stuck or need a human, recorded together with the capability added to close it.
_Avoid_: Blocker, agent failure

**Unattended runtime**:
How long the agent works on a single task before a human has to intervene.
_Avoid_: Autonomy, session length

**Hand-written**:
Any repository content typed by a human rather than produced by the agent; nothing hand-written enters the repo.
_Avoid_: Manual code

### Exchange

**Reference exchange**:
Hyperliquid, whose public market data supplies this exchange's oracle prices and market parameters. It is never traded against.
_Avoid_: Upstream, source, Hyperliquard

**Market**:
A single perpetual-futures contract on one asset (e.g. BTC), with its own order book and parameters.
_Avoid_: Pair, symbol, instrument

**Order book**:
The resting limit orders for one market, matched by price-time priority.
_Avoid_: Book, depth, CLOB

**Clearinghouse**:
The part of the exchange that owns accounts, positions, margin and profit and loss.
_Avoid_: Risk engine, ledger

**Paper balance**:
An account's simulated collateral; it has no real-world value.
_Avoid_: Funds, deposit, wallet

**Oracle price**:
A market's external reference price, taken from the reference exchange.
_Avoid_: Index price, spot price

**Funding**:
A periodic payment between longs and shorts that pulls a market's price toward its oracle price.
_Avoid_: Funding fee, interest

**Liquidation**:
The forced closing of an account's positions when its margin falls below the maintenance requirement.
_Avoid_: Margin call, stop-out

**Bot trader**:
An automated account run by the exchange's operators that trades to keep markets liquid and active; a market-maker bot is one kind.
_Avoid_: Simulated user, fake trader
