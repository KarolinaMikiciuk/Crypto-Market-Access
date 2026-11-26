Date: 26-11-2026
Author: Karolina

### Status
Accepted

### Context
- We need to ...

### Decision
- CAP
    - CP-leaning stance (Safety / Correctness first)
    - You mark the book STALE / SYNCING, halt downstream pricing/execution signals for that symbol, and re-stitch a fresh REST snapshot before returning to LIVE. (incl. crossed book, best_bid >= best_ask)

- streams: (spot only, L2 live order book)
- cadence: Binance buffers book deltas over a 100ms window and emits one aggregated diff frame every 100ms (if changes occurred), no 100ms network guaranntee
- freshness requirements: 
- exchange boundaries:
    - conn limits:
        - connection attempts per IP (300 every 5 minutes)
    - message limits:
    - max. session lifetime:
    - ping-pong rules: 
        - stream: 5 msgs incoming to Binance / second
        - REST:
    - shared multiplexing: 10 symbols per conn across 4 sockets (measure)
        - how did I arrive at this number? Too few shards (1 connection × 30 symbols): Big blast radius on disconnects, big resync bursts, single-stream bottleneck. Too many connections (30 connections × 1 symbol): High socket/task overhead, more handshake/ping traffic, and higher risk of hitting connection-creation rate limits if many reconnect at once.

- observability + staleness SLAs
    - expose exchange_event_time vs. ingest_recv_time vs. publish_time on every book update,
    - and guarantee correctness & staleness gating (e.g., "if feed lag exceeds X ms or a sequence gap occurs, the book immediately transitions to STALE within Y ms so downstream consumers never trade on silent bad data").

- deserialisation: start with JSON, alternative SBE parser class (SBE data available for some symbols only)
### Trade-offs
