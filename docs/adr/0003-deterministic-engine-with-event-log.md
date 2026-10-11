# A deterministic in-memory engine fed by an append-only event log

The exchange has no database as its source of truth. Every input — order placed, order cancelled, oracle update, funding tick, deposit, withdrawal — is appended to an event log, and the engine is a pure function of that log: the same events always produce the same state. Read models such as trade history are projected from the log into SQLite and can be rebuilt at any time. We chose this so any bug can be reproduced exactly by replaying the log, which is what lets an agent reproduce a reported failure before fixing it, and because the log maps naturally onto on-chain settlement in the later testnet phase (ADR-0001).

## Consequences

- Service-layer code may never read the wall clock, generate randomness or touch the network; time and randomness arrive as events through Providers. A lint enforces this.
- Restarts rebuild state by replaying the log, so the log's event formats need versioning, and long logs will eventually need snapshots.

## Considered Options

- **Database as source of truth**: rejected; logic spreads across queries and failures cannot be replayed deterministically.
- **In-memory only**: rejected; a restart loses the bots' overnight history.
