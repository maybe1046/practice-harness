# Prices and sizes are whole numbers of ticks and lots, never floating point

Every price is stored as a `bigint` count of its market's tick and every size as a `bigint` count of its lot; conversion to and from decimals happens only at the API and UI boundary. Floating point makes margin and liquidation arithmetic drift by rounding noise, so whether an account gets liquidated would depend on accumulated error, and such bugs cannot be reproduced. Integer ticks and lots are exact by construction, mirror the reference exchange's own size and price rounding rules, and are cheap to enforce with a lint that bans `number` in domain money types. Each market's tick is fixed, set from its price when the market is configured; we deliberately do not copy the reference exchange's rule of at most five significant figures, because that makes the tick change with the price, so stored tick counts and resting orders would need rescaling whenever the price crosses a power of ten.

## Considered Options

- **JavaScript `number`**: rejected for rounding drift.
- **A decimal library (`decimal.js`, `big.js`)**: exact, but slower and easily mixed with plain `number` by accident; integers make the boundary explicit.
- **A price-dependent tick, as the reference exchange uses**: more faithful, but rejected because a tick that changes with the price breaks the meaning of stored tick counts.
