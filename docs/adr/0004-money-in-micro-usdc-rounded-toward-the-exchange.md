# Money is whole micro-USDC, and every rounding favours the exchange

Ticks and lots (ADR-0002) are per market, but cross margin adds amounts across markets, so every amount of money — paper balance, margin, profit and loss, funding payments — is a `bigint` count of micro-USDC (10⁻⁶ paper USDC, USDC's own precision). Each market's tick × lot must be a whole number of micro-USDC, which the Markets domain checks when a market is configured; a position's notional value is then exact, and rounding is needed only where a division or a rate appears, such as initial margin and funding. Every rounding goes against the account: amounts an account owes or must reserve round up, amounts it receives round down. Rounding can therefore never create paper balance, and the leftover from each rounding is credited to the Exchange account, so the conservation invariant stays exact: the paper balance of all accounts, the Exchange account included, equals Deposits minus Withdrawals.

Rates and ratios that multiply money (funding rates, margin ratios, leverage) are fixed-point `bigint` too, so no `number` appears in a domain type.

## Considered Options

- **Money in each market's tick × lot**: rejected; one account holds positions in several markets, and their units differ.
- **Round to nearest**: rejected; it can round in the account's favour, creating paper balance out of nothing and breaking conservation.
- **Let rounding leftovers vanish**: rejected for the same reason in reverse; conservation would hold only approximately, and a property test could not check it exactly.
