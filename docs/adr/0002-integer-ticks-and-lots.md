# Prices and sizes are whole numbers of ticks and lots, never floating point

Every price is stored as a `bigint` count of its market's tick and every size as a `bigint` count of its lot; conversion to and from decimals happens only at the API and UI boundary. Floating point makes margin and liquidation arithmetic drift by rounding noise, so whether an account gets liquidated would depend on accumulated error, and such bugs cannot be reproduced. Integer ticks and lots are exact by construction, mirror the reference exchange's own size and price rounding rules, and are cheap to enforce with a lint that bans `number` in domain money types.

## Considered Options

- **JavaScript `number`**: rejected for rounding drift.
- **A decimal library (`decimal.js`, `big.js`)**: exact, but slower and easily mixed with plain `number` by accident; integers make the boundary explicit.
