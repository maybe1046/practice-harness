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

**ExecPlan**:
A living, self-contained document in the repo describing how a multi-PR piece of work will be done, with its progress, decisions and discoveries; the issue it serves says what and why.
_Avoid_: Plan, design doc, spec

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
An account's simulated collateral, denominated in paper USDC; it has no real-world value.
_Avoid_: Funds, deposit, wallet, paper USDC (that is its unit, not the term)

**Account value**:
An account's paper balance plus the unrealized profit and loss of all its open positions, valued at mark price.
_Avoid_: Equity, net worth, margin balance

**Leverage**:
The multiple an account chooses per market, up to that market's maximum, that sets the initial margin a position requires.
_Avoid_: Multiplier, gearing

**Initial margin**:
The margin a position requires to open: its notional value divided by the account's leverage for that market.
_Avoid_: Required margin, collateral requirement

**Maintenance margin**:
The minimum margin a position needs to stay open: half the initial margin at the market's maximum leverage.
_Avoid_: Minimum margin, liquidation margin

**Oracle price**:
A market's external reference price, taken from the reference exchange.
_Avoid_: Index price, spot price

**Mark price**:
A market's fair price, computed by this exchange from the oracle price and its own order book; it drives unrealized profit and loss, margin checks and liquidations.
_Avoid_: Fair price, last price, markPx

**Tick**:
A market's fixed smallest price increment; every price is a whole number of ticks. Unlike the reference exchange's significant-figure rule, it does not change with the price.
_Avoid_: Price step, pip

**Lot**:
A market's smallest size increment; every size is a whole number of lots.
_Avoid_: Size step, contract, unit

**Cross margin**:
A margin mode in which one account's paper balance backs all its positions across markets at once.
_Avoid_: Portfolio margin, shared margin

**Deposit**:
The only way paper balance enters the clearinghouse.
_Avoid_: Top-up, mint, credit

**Withdrawal**:
The only way paper balance leaves the clearinghouse.
_Avoid_: Cash-out, burn, debit

**Exchange account**:
The single account owned by the exchange itself; it collects every rounding leftover and, once fees exist, every fee, and it counts toward conservation like any other account.
_Avoid_: House account, fee account, treasury

**Funding**:
A periodic payment between longs and shorts that pulls a market's price toward its oracle price.
_Avoid_: Funding fee, interest

**Liquidation**:
The forced closing of an account's positions when its account value falls below its total maintenance margin.
_Avoid_: Margin call, stop-out

**Bot trader**:
An automated account run by the exchange's operators that trades to keep markets liquid and active; a market-maker bot is one kind.
_Avoid_: Simulated user, fake trader
