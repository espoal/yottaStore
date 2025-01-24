# Introduction

This repository contains the architectural documentation of the yottaStore project, which is made up of two main 
components:

- yottaFS: A key value store backed by NVMe storage and CRDTs
- yottaStore: A large scale distributed datastore built on top of yottaFS

Three main ideas are at the core of the yottaStore project:

- NVMe storage is fast enough to be a replacement for RAM [NVMe as RAM]()
- The dynamo paper is an excellent starting point to build a distributed datastore [Dynamo as a datastore]()
- io_uring is the best interface for async IO [Why io_uring]()

## yottaFS

yottaFS is a single node key value store, designed to exploit the performance characteristics of NVMe storage.
One could think of it as a cross between a disk backed Redis and BTRFs. 
While an interesting project in itself, yottaFS is also the foundation of yottaStore.

Read more documentation [here](docs/yottaFS.md) or visit the [repository]()

## yottaStore

yottaStore is a [next generation]() storage system aiming to **scale out to the yotta byte range** 
and **scale up to millions of concurrent read and writes per record**. 
The goal is to have **two orders of magnitude more throughput than DynamoDB**, 
dollar per dollar, while maintaining a **sub-ms latency**. 

// TODO: move to a separate sections

Yotta Store is built on top of a 512 bit distributed machine, with a 4kib word size.
We try to design a system which can exploit the capabilities of
modern hardware and software, like  NVMe disks or `io_uring`. 
Read more in the [docs](docs/README.md)

- [Rust implementation](https://github.com/yottaStore/rust) (WIP) Backed by io_uring NVMe API
- 
## Main features

- Linear scalability, up to 10^9 nodes and 10 yotta bytes of addressable space.
- Anti fragility, the multi tenant setup increase reliability and availability with load.
- Self configuring and optimizing, thanks to machine learning
- Strong consistency guarantees, aiming at sub ms latency.
- Cheap transactions and indexes, at around `o(n)`.
- Storage decoupled from compute with a serverless architecture.
- Two orders of magnitude faster than DynamoDB, dollar per dollar.


## Techniques used

Yotta Store does not create any new technique, but uses existing ones in a novel combination:

| Problem                          | Solution                        | Description                                                                                                               |
|----------------------------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| Partitioning and replication     | ReBaR: Rendezvous Based Routing | A generalization of consistent hashing, for agreeing k <br/>choices in a stable way and with minimal information exchange |
| Highly available read and writes | YottaFS                         | A wait free userspace filesystem, designed around NVMe devices and CRDTs.                                                 |
| Hot records                      | YottaSelf                       | Hot records can be sandboxed in dedicated partitions.                                                                     |
| Transactions                     | Modified Warp                   | Thanks to the modified warp algorithm no distributed consensus is needed.                                                 |
| Indexes, queries across keys     | Partitioned indexes             | Indexes are sharded and lazily built on the same partition seeing the changes.                                            |
| Membership and failure detection | Gossip agreement protocol       | Thanks to GAP and ReBaR the expected cost is `o(log(n))` deterministically.                                               |


## Inspirations

- [Designing Data-Intensive Applications, Martin Kleppmann](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/)
- [The Dynamo paper, many authors](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)
- [A pragmatic implementation of non-blocking linked-lists, Timothy L. Harris](https://timharris.uk/papers/2001-disc.pdf)
- [Warp: Lightweight Multi-Key Transactions for Key-Value Stores, many authors](https://arxiv.org/pdf/1509.07815.pdf)
- [Thread per core](https://helda.helsinki.fi/bitstream/handle/10138/313642/tpc_ancs19.pdf)
- [Kafka](https://docs.confluent.io/platform/current/kafka/design.html)


