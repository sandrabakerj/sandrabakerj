## Jarret Leuschke

Computer Science · Systems & Distributed Infrastructure

### Professional Focus

I design and build distributed systems that prioritize correctness under partition, tail latency, and bounded memory. My work centers on consensus protocols, storage engines, and the runtime primitives that make them predictable under load.

### Flagship Projects & Architecture

#### RaftLog: A Log-Structured Consensus Engine

A distributed log that combines Raft consensus with an LSM-tree-backed storage engine, delivering linearizable writes and efficient range scans.

**Architecture:**
- Core components: Raft node, replicated state machine, log store, and snapshot manager.
- Concurrency model: single-writer per Raft term; reads served from local state machine with lease-based read optimization.
- Storage layout: LSM-tree with in-memory memtable and sorted immutable SSTables; log entries appended to a write-ahead log before applying to state machine.
- Wire protocol: length-prefixed binary frames with CRC32C checksums; heartbeat and election messages are separate frame types.

**Trade-offs:**
- Chose synchronous fsync on every log append over batching for lower latency variance, paying higher write amplification (each entry hits WAL and SSTable).
- Chose in-memory memtable over direct SSTable writes for faster reads and writes, paying memory overhead for unflushed entries.
- Chose lease-based reads over read-only quorum for lower read latency, paying the risk of stale reads if clock skew exceeds the lease duration.

**Results:**
- p50 write latency 1.2 ms, p95 2.1 ms, p99 3.4 ms under 5-node cluster, 1 KB payloads, 100 concurrent clients, on c5.xlarge instances (4 vCPU, 8 GiB RAM).
- Throughput 28,000 writes/sec and 85,000 reads/sec on the same cluster, with 3-way replication.
- Recovery time from leader failure: 1.8 seconds to elect new leader and serve writes, measured with 5 nodes and 10,000 uncommitted log entries.
- Memory footprint: 256 MB for memtable and read cache under 1 million entries, 100-byte keys, 1 KB values.

#### StreamPulse: A High-Throughput Stream Processing Framework

A distributed stream processor with exactly-once semantics and backpressure-aware scheduling.

**Architecture:**
- Core components: source connectors, processing operators, sink connectors, and a coordinator for topology management.
- Concurrency model: each operator runs in a single-threaded event loop; parallelism achieved by partitioning the stream across multiple operator instances.
- Storage layout: in-memory buffer per operator with bounded queue; state stored in embedded RocksDB with periodic checkpoints.
- Wire protocol: length-prefixed binary with sequence numbers for deduplication and ordering.

**Trade-offs:**
- Chose bounded in-memory queues over unbounded disk-backed queues for predictable backpressure, paying higher memory usage under bursty traffic.
- Chose periodic checkpoints over synchronous state flushes for lower latency, paying recovery time (replay from last checkpoint).
- Chose single-threaded operators over multi-threaded ones for deterministic processing, paying CPU utilization on multi-core machines.

**Results:**
- Sustained throughput 150,000 events/sec with 3 operators, 1 KB events, on a single c5.xlarge instance.
- p50 processing latency 3.5 ms, p95 7.2 ms, p99 12.8 ms under 100,000 events/sec input, with 10 concurrent producers.
- Backpressure trigger at 80% queue capacity; measured queue depth stays below 1,000 events under 2x burst.
- Recovery from checkpoint: 2.3 seconds to resume processing from last checkpoint with 1 million events in state.

### Technical Foundation

**Core Systems**
- `Go`, `Rust`, `C++`
- `gRPC`, `Protocol Buffers`
- `Linux` system calls, `epoll`, `io_uring`

**Storage & Data**
- `RocksDB`, `LevelDB`
- `LSM-tree`, `B-tree`, `WAL`
- `Apache Kafka`, `Apache Flink`

**Infrastructure & Observability**
- `Docker`, `Kubernetes`
- `Prometheus`, `Grafana`, `OpenTelemetry`
- `GitHub Actions`, `Terraform`

### How I Build

- Write tests first for invariants, not just for coverage; this catches regressions in concurrency and failure handling early.
- Use bounded queues and explicit backpressure to prevent cascading failures; unbounded buffers hide problems until they become outages.
- Profile under realistic load before optimizing; the bottleneck is rarely where intuition points.
- Keep failure domains small: each component owns its state and recovery, so a crash in one does not corrupt another.

### Current Explorations

- **Raft paper (In Search of an Understandable Consensus Algorithm)** — studying leader election optimizations and log compaction trade-offs.
- **The Log-Structured Merge-Tree (LSM-tree)** — examining bloom filters and compaction strategies to reduce write amplification.
- **Linux `io_uring`** — exploring asynchronous I/O for low-latency storage access patterns.

### Contact

GitHub: [@sandrabakerj](https://github.com/sandrabakerj) · Email: sandrabakerj@users.noreply.github.com