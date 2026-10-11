# Perpetual-futures primer

How the exchange's mechanics work, with worked examples. This file explains; it does not define. Every term in **bold** is defined in [`GLOSSARY.md`](../../GLOSSARY.md), and the glossary wins if the two disagree.

All examples use one fictional setup so the arithmetic stays easy:

- The account's **paper balance** is 300.
- The COIN **market** has a maximum **leverage** of 10×.
- The account selects 5× leverage and opens a long position of 10 COIN at a **mark price** of 100.

The examples ignore fees. Only the funding section includes **funding**.

## Margin

Margin is reserved, not spent. Opening the position does not reduce the paper balance; it reserves part of the **account value**.

Two lines matter:

- **Initial margin** (line A) gates opening or increasing a position, and **withdrawals**.
- **Maintenance margin** (line B) gates staying open. Below it, **liquidation** starts.

Both lines use the notional value (size × mark price) at the current mark price, so both move when the price moves. A loss reduces the account value; it does not come out of the reserved initial margin.

| Mark price | Account value | Initial margin (÷ 5) | Maintenance margin (½ × ÷ 10) | Result |
|---|---|---|---|---|
| 100 | 300 | 200 | 50 | Above line A: 100 is free for new positions or a withdrawal. |
| 90 | 200 | 180 | 45 | Above line A: 20 is free. |
| 80 | 100 | 160 | 40 | Between the lines: stays open; no new positions, no withdrawal. |
| 75 | 50 | 150 | 37.50 | Between the lines: stays open. |
| 73 | 30 | 146 | 36.50 | Below line B: liquidation starts. |

Exact liquidation price:

```
300 + 10 × (P − 100) < 0.5 × P   →   9.5 × P < 700   →   P < 73.68
```

Maintenance margin uses the market's **maximum** leverage, so the leverage the account selects does not move the liquidation price under **cross margin**. It only changes how much account value line A reserves.

Under cross margin, one account value backs every position in every market: a profit in one market supports a loss in another, and a large loss in one market can liquidate all of them.

## Mark price

Margin and liquidation use the mark price, not the last trade price, so one unusual trade cannot easily force a liquidation.

Example: the **oracle price** is 100. A large sell order into a thin **order book** prints a last trade at 70.

| If the exchange used... | Price | Result for the 10 COIN position |
|---|---|---|
| Last trade price | 70 | Liquidation. |
| Mark price | about 99 | Stays open. |

The mark price still takes this exchange's order book into account, so a lasting difference between traders here and the **reference exchange** does move it.

## Funding

A dated future converges to the asset's price at expiry. A perpetual future never expires, so funding does that job.

| This exchange's price vs the oracle price | Who pays |
|---|---|
| Above | Longs pay shorts |
| Below | Shorts pay longs |

```
funding payment = notional value × funding rate
```

Example: long 10 COIN at a mark price of 80, price below the oracle price, funding rate 0.02%. The long receives 800 × 0.02% = 0.16.

Funding changes the account value just like profit and loss, so paying it for long enough moves an account toward line B.

## Order book

The order book matches by price-time priority: best price first, then the oldest order at that price.

| Side | Price | Size | Trader |
|---|---|---|---|
| Sell | 100.2 | 5 | |
| Sell | 100.1 | 3 | |
| Buy | 99.9 | 2 | Alice (first) |
| Buy | 99.9 | 2 | Bob (second) |
| Buy | 99.8 | 6 | |

- An immediate sell of 3 matches 2 with Alice, then 1 with Bob, all at 99.9.
- An immediate buy of 6 matches 3 at 100.1, then 3 at 100.2, for an average of 100.15.

Prices must be whole **ticks** and sizes whole **lots**; with a tick of 0.1, 99.95 is rejected. **Bot traders** keep both sides of the book filled.

## Liquidation

Liquidation closes the account's positions by trading in the order book. Maintenance margin is the buffer that should leave some account value after that close.

Liquidation of the example position starts at about 73.68, with about 36.84 of account value left.

| Close price | Account value after | Result |
|---|---|---|
| 73.5 | 35 | Buffer was enough. |
| 70 | 0 | Buffer used up. |
| 68 | −20 | Negative: someone else must cover 20. |
