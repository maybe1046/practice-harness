# Off-chain paper exchange first, with a custody seam for a later testnet

The exchange starts fully off-chain with paper balances, and a later testnet phase moves only custody and settlement on-chain (the hybrid shape dYdX v3 used), never matching or margin. To keep that move cheap, paper balance enters and leaves the clearinghouse only through Deposit and Withdrawal, so a contract can later replace that one boundary without touching the order book or margin code.

## Considered Options

- **Real money**: rejected outright. Custody of others' funds and derivatives carry regulatory, legal and loss risk no practice project should hold.
- **Everything on-chain, as Hyperliquid does**: rejected. It turns the testnet phase into a rewrite and spends the time on consensus plumbing rather than harness engineering.
- **Decide later**: rejected. Balance mutations would spread through the codebase and make any later move expensive.
