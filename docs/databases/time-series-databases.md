---
layout: default
title: Time-series databases
description: Notes on time-series databases.
has_children: true
has_toc: false
parent: Databases
nav_order: 7
permalink: /databases/time-series-databases
---

<!-- prettier-ignore-start -->

# Time series databases

{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Time-series Databases

A time-series database stores time-series data.

Time series data is a sequence of data points collected over time intervals. Time series data is often alerted on and dashboarded for monitoring purposes.

An example time series for the number of server requests over 10s intervals:

```js
{
  "datapoints": [
    // [server_req_count, unix_timestamp_ms]
    [41, 1646680930000],
    [40, 1646680940000],
    [39, 1646680950000],
    // ..
    [15, 1646681050000]
  ]
}
```

Popular open-source time series DBs include Whisper (Graphite), Prometheus, InfluxDB, and Prometheus.

To improve retention, older time series data is often rolled up into increasingly low-resolution data points.

## Gorilla

**Gorilla** is an in-memory time-series database created by Meta.

Gorilla was designed to support the following requirements:

- Low latency queries (10s of ms)
- Fine-grained aggregations over short time windows
- High availability
- Fault tolerance

Meta observed that the workload for their time series system was write-heavy. They also observed that the majority of reads were for recent data (~85% of requests could be satisfied with data <26 hours old). Based on these observations, Gorilla was built as a write-through cache storing recent data, with the rest of the data stored in HBase.

The data is a triple: \<key: string, timestamp: uint64_t, value: double\>. Data is sharded based on the key.

{% cite pelkonen2015gorilla -l 1816-9 %}

### Compression

Gorilla uses two novel encodings: delta-of-delta encoding for integer timestamps and XOR encoding for floating point values.

### Delta-of-delta encoding

In delta-of-delta encoding the delta between previous values is calculated, and then the delta between the previous delta is calculated and stored.

**Timestamp** | 1645503469 | 1645503569 | 1645503669 | 1645503679 |
**Delta** | 1645503469 | 100 | 100 | 10 |
**Delta of delta** | 1645503469 | 100 | 0 | 10 |

Gorilla compresses timestamps using delta-of-delta encoding, based on the observation that timestamp data is often at fixed intervals (leading to small delta-of-deltas).

~96% of Meta's timestamps could be encoded in a single bit using the following variable-bit scheme:

- 0 - store `0b0`
- [-63, 64] store `0b10` followed by value (7 bits)
- [-255, 256] store `0b110` followed by value (9 bits)
- [-2047, 2048] store `0b1110` followed by value (12 bits)
- Else store `0b1111` followed by value (32 bits)

A block header stores the timestamp $$t-1$$, which is aligned to a two hour window. The first timestamp is then stored as the delta from $$t-1$$ in 14 bits.

{% cite pelkonen2015gorilla -l 1820 %}

### XOR encoding

Data point values (doubles) are stored using XOR encoding, where data is based on an XOR of previous values.

The first value is stored without compression.

For the rest of the data, the following scheme is used:

- If XOR with the previous value is 0, then store `0b0`
- If XOR is nonzero, calculate the number of leading and trailing zeros in the XOR. Store `0b1` followed by:
  - Control bit `0b0`—if the block of meaningful bits is within the same block as the previous meaningful bits (there are at least as many leading zeros and trailing zeros as the previous one), then use that information for the block position and just store the meaningful XOR value.
  - Control bit `0b1`—store the length of the number of leading 0s in the next 5 bits, then store the length of the meaningful XORed value in the next 6 bits, then store the meaningful bits of the XORed value.

Meta's found ~51% of values are stored within a single bit, ~30% are stored with control bits `0b10`, and the remainder are stored with control bits `0b11`.

{% cite pelkonen2015gorilla -l 1820-1 %}

### Data structures

Tha main in-memory data structure for Gorilla is a `TimeSeriesMap` consisting of a vector of stdlib shared pointers to `TimeSeries` objects and a case-insensitive case-preserving map from time series names to a `TimeSeries` entry:

```cpp
struct TimeSeriesMap {
  ReadWriteLock *lock;
  vector<sharded_ptr<TimeSeries>> *ts_vec;
  unorderd_map<string, shared_ptr<TS>> *ts_map;
}

struct TimeSeries {
  SpinLock *lock;
  string open_block;
  vector<string> closed_blocks;
}

struct ShardMap {
  vector<unique_ptr<TSmap>>;
  ReadWriteLock *lock;
}
```

A `ShardMap` maps shard IDs to TSMaps. Null pointers are stored in the shard map if the shard isn't held by the node {% cite pelkonen2015gorilla -l 1821 %}.

A `TimeSeries` contains a sequence of closed blocks for data older than two hours and a single open block which is an append-only string where new values are added (it's often reallocated as its size changes). When a block is closed, it's moved to slab-allocated memory where it is left untouched until it's deleted from memory {% cite pelkonen2015gorilla -l 1822 %}.

Data is read by copying all data blocks that could contain data for a query's key and time range directly into the output RPC structure. Decompression is done outside of Gorilla {% cite pelkonen2015gorilla -l 1822 %}.

### Persistence

Gorilla achieves persistence by storing data in GlusterFS with 3X replication.

A Gorilla host owns multiple shards of data. It maintains a single directory per shard. A directory contains four types of files:

- Key lists (map of key string to integer identifying index in in-memory vector)
- Append-only logs
- Complete block files
- Checkpoint files

{% cite pelkonen2015gorilla -l 1822 %}

Each shard represents about 16GB on-disk storage {% cite pelkonen2015gorilla -l 1823 %}.

New keys are appended to the key list. Gorilla scans all keys for each shard in order to re-write the file.

When data is streamed to Gorilla, it is stored in a log file in compressed format. Keys are interleaved, and so a timestamp-value pair is stored along with its 32-bit integer ID {% cite pelkonen2015gorilla -l 1822 %}.

Gorilla doesn't offer ACID guarantees (it's not a WAL). Gorilla buffers ~64KB of data before writing it to the log file. The buffer is flushed on a clean shutdown but a crash can cause a small amount of data loss {% cite pelkonen2015gorilla -l 1822 %}.

Every two hours, Gorilla copies the compressed block data to disk. The block has two sections: a set of consecutive 64KB slabs of compressed data blocks, and a list of \<time_series_ID, data_block_pointer\> pairs. When a block file is complete, Gorilla creates a checkpoint file (marking when a complete block file is flushed to disk) and deletes the corresponding logs {% cite pelkonen2015gorilla -l 1822 %}.

If a block file isn't flushed to disk on a crash then the new Gorilla process will find that the checkpoint file doesn't exist. In this case it will read from the log file only {% cite pelkonen2015gorilla -l 1822-3 %}.

### High availability

Region failures are handled by having two Gorilla instances in separate DCs. Data is streamed to both instances and there is no attempt to guarantee consistency. In the case one instance fails, traffic is routed to the redundant instance.

Single node failure is handled using ShardManager—a Paxos-based system. When a node fails, ShardManager distributes its shards among the remaining nodes in the cluster. During shard movement, write clients buffer their incoming data (the buffer holds 1 minute of data and older data is dropped), this works for routine shard reassignment. If a Gorilla host crashes in a region, writes are buffered by the client and the Gorilla cluster attempts to resurrect the host. If the shard movement takes too long, reads can be pointed to the corresponding Gorilla host in the other region.

When a shard is added to a host, the host reads all the data from GlusterFS. A host can read all the data it needs to be fully functional in about 5 minutes. While the host is reading data, it accepts incoming data points and puts them in a queue to be processed. When shards are reassigned, clients drain their buffers by writing to the new node. In the case of a crash, as soon as a new host is assigned a shard it begins accepting streaming writes, so no in-flight data is lost. If a host shuts down gracefully then it flushes data to disk before exiting meaning that no data is lost (software upgrades can be handled via rolling upgrades using this mechanism).

If a host crashes before flushing the data to disk, the data is lost. In practice this is rare and only a few seconds of data will be lost, and so the increased write throughput is considered worth the tradeoff.

After a node failure, queries return _partial data_. When a client library receives a partial result, it will try the redundant region. In the case that both results are partial, the client returns the partial data with some flags so that users can be alerted to the status of the data.

{% cite pelkonen2015gorilla -l 1823 %}

## References

{% bibliography --cited_in_order %}
