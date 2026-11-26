# Metrics
- latency percentiles p50, p95, p99, p99.9
- Throughput & Goodput
- Queue depth / Mailbox lag
- Resource efficiency: CPU utilization, memory footprint (including per-actor overhead or queued message buffers), GC pauses, and context-switching/serialization overhead.
- Failure & recovery behavior: What happens when a downstream dependency slows down, a single entity/key is hammered ("hot key"), or a worker/actor crashes?


# Trade Offs/Alternatives

| Design    | Strengths | Weaknesses |
| -------- | ------- | ------------- |
| couroutine updates OrderBook Registry symbols dict (per symbol lock/condition) | smaller task/queue overhead | ?               |
| 1:1 shard: asyncio worker loop, 1:M worker:symbol orderbook, 1:1 shard:ws conn | as above    |A slow resync or bursty symbol inside a shard can contend for that shard's single loop more than an isolated per-symbol actor mailbox would     |
| separate process per shard    | True CPU parallelism across cores; a crash or hang in one shard process is isolated at the OS process level; when CPU and serialisation/deserialisation is the bottleneck |  more mem-intensive, IPC/serialization cost when passing BookViews across process boundaries (whereas in-process asyncio.Queue passes references in memory)             |
