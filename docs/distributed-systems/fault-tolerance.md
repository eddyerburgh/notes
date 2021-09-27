---
layout: default
title: Fault tolerance
description: Notes on fault tolerance in distributed systems.
has_toc: false
nav_order: 5
parent: Distributed systems
permalink: /distributed-systems/fault-tolerance
---

<!-- prettier-ignore-start -->

# Fault tolerance
{:.no_toc}

In large distributed systems failures are a common occurrence, so fault tolerance must be built in by design.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

**Fault tolerance** is the ability of a system to continue operating despite partial failures. Achieving fault tolerance is one of the benefits of creating a distributed system {% cite distributed-systems -l 423 %}.

Availability, reliability, and recoverability are all important concepts in fault tolerance.

**Availability** is "the property that a system is ready to be used immediately". It's usually measured as a percentage of uptime (e.g. 99.99%) {% cite distributed-systems -l 424 %}.

**Reliability** is the property that a system can run continuously without failure.

The difference between reliability and availability can be thought of like this: "if a system goes down on average for one, seemingly random, millisecond every hour, it has an availability of more than 99.999 percent, but is still unreliable. Similarly, a system that never crashes but is shut down for two weeks every August has high reliability but only 96 percent availability" {% cite distributed-systems -l 424 %}.

**Recoverability** is the ability of a system to recover after becoming unavailable due to failure.

A **fault** is a deviation from expected behavior that can affect the function or performance of a system.

There are different types of fault:

- **Transient faults** occur once and then disappear.
- **Intermittent faults** occur periodically.
- **Permanent faults** exist until the cause of the fault is fixed.

{% cite distributed-systems -l 427 %}

A fault can result in a failure.

**Fail-stop failures** are failures that can be reliably detected. They are often caused by a system crash or network partition. Fail-stop failures can be tolerated by utilizing methods like replication and redundancy.

**Byzantine failures** are failures where nodes send conflicting information to different parts of the system, either accidentally or intentionally. Many protocols that can tolerate fail-stop failures cannot tolerate byzantine failures {% cite LamportSP82 -l 382 %}.

### Failure detection

The main approach to failure detection is to continuously send heartbeat messages between nodes. Timeouts are then used to detect failures—if heartbeat messages from process $$P$$ to process $$Q$$ stop for a certain period of time, then process $$Q$$ will suspect that $$P$$ is down {% cite distributed-systems -l 462 %}.

In an asynchronous distributed system, it's not possible for a node to ascertain whether another node has failed, or whether the node is simply unreachable {% cite distributed-systems -l 430 %}.

Failure detection can be done at the same time as exchanging information, such as during gossip dissemination {% cite distributed-systems -l 463 %}.

### Failure metrics

Useful metrics when dealing with failures include:

- MTTF (Mean Time To Failure)
- MTTR (Mean Time To Repair)

The MTTF of a hardware component is the average amount of time the component can be expected to run for before it fails.

MTTR is the amount of time required to fix (or more commonly, to replace) a failed hardware component. A low MTTR can be achieved by running standby components that can replace failed live components automatically {% cite patterson1988 -l 111 %}.

### Split brain

**Split brain** is a situation that can occur during network partitions where multiple nodes believe they are the primary. Split brain can quickly lead to data inconsistency between nodes.

Using write quorums can help avoid split brain.

## Redundancy

**Redundancy** is where components are duplicated in order to improve the reliability of a system. If one of the components fails, the others can continue operating.

A system is k-fault-tolerant if it can handle the failure of k components {% cite distributed-systems -l 435 %}.

There are several types of redundancy:

- Information redundancy
- Time redundancy
- Physical redundancy

With **information redundancy** extra bits are used to provide the ability to recover from flipped/incorrectly read bits (e.g. checksums) {% cite distributed-systems -l 431 %}.

With **time redundancy** operations are retried in the case of timeouts {% cite distributed-systems -l 431 %}.

With **physical redundancy** extra physical components are added to a system so that the system can tolerate the loss of some components {% cite distributed-systems -l 431 %}.

### Process groups

One way to improve resiliency is to group processes together into **process groups** that are treated as a whole. The loss of one or more processes in the group shouldn't then break the system.

In some groups, all processes are equal, in other groups (like a database cluster) some processes are designated special roles {% cite distributed-systems -l 433 %}.

In a process group where nodes contain shared state, each node should execute the same commands in the same order as every other non-faulty processes. This means that nodes need to reach consensus on which commands to execute {% cite distributed-systems -l 433, 436 %}.

## Consensus

A consensus algorithm is an algorithm that's used to get distributed processes to agree on a value.

One use of consensus algorithms is to keep a sequence of replicated logs consistent between replicated state machines. A server can receive commands from clients and add them to its log, the server would then communicate with the consensus modules on other servers to ensure that the replicated log contains the same commands in the same order on a majority of participating nodes {% cite 10.5555/2643634.2643666 -l 2 %}.

A practical consensus algorithm will:

- Ensure safety: never return an incorrect value despite non-Byzantine failures.
- Be available: as long as the majority of machines are running and can communicate with each other.
- Not depend on clocks to ensure the consistency of logs.
- Allow a command to complete once a majority of the cluster has agreed to it.

{% cite 10.5555/2643634.2643666 -l 2 %}

### Paxos

Paxos is a distributed consensus protocol that can handle failures of some participating processes.

The aim of Paxos is to agree upon a single proposed value {% cite Lamport2001PaxosMS -l 1 %}.

There are five roles in Paxos:

1. Client
2. Proposer
3. Acceptor
4. Learner
5. Leader

_Note: in an implementation, a single process can perform multiple roles {% cite Lamport2001PaxosMS -l 1 %}._

**Clients** send requests to the system and wait for a response.

**Proposers** advocate a client request and send the proposed value to a set of **acceptors** which may accept the value. A value is chosen when a majority of acceptors have accepted it {% cite Lamport2001PaxosMS -l 2 %}.

**Learners** learn the chosen value and can take action on it.

The **leader** (a distinguished proposer) can be used as the sole proposer to avoid stalemate situations. A new leader should be elected in the case of a leader failing {% cite Lamport2001PaxosMS -l 7 %}.

In order to determine which proposal should win in the case of multiple proposals, each proposal is assigned an increasing number $$n$$.

There are two phases to the Paxos algorithm: promise and accept.

In the promise phase:

- A proposer selects a proposal number $$n$$ and sends a PREPARE request (with $$n$$ attached) to the acceptors.
- If an acceptor receives a PREPARE request with number $$n$$ greater than the number of any PREPARE request that it has already responded to, then the acceptor responds to the request with a promise not to accept any more proposals numbered less than $$n$$, along with the highest-numbered proposal (if any) that it has accepted.

{% cite Lamport2001PaxosMS -l 5-6 %}

In the accept phase:

- If the proposer receives a response to its PREPARE requests (numbered $$n$$) from a majority of acceptors, then it sends an ACCEPT request to each of those acceptors for a proposal numbered $$n$$ with a value $$v$$, where $$v$$ is the value of the highest-numbered proposal among the responses, or is any value if the responses reported no proposals.
- If an acceptor receives an ACCEPT request for a proposal numbered $$n$$, it accepts the proposal unless it has already responded to a PREPARE request having a number greater than $$n$$.

{% cite Lamport2001PaxosMS -l 5-6 %}

Once a value has been accepted, each learner needs to learn the chosen value. There are several ways this could be done. One way is to have each acceptor send a message to each learner whenever an acceptor accepts a proposal and have the learner handle duplicate messages (although this is inefficient) {% cite Lamport2001PaxosMS -l 8 %}.

One of the issues with Paxos is that it focusses on agreeing on a single value, rather than focussing on the common case of agreeing on multiple values over time. Raft is an alternative protocol that was designed with multiple decisions in mind {% cite 10.5555/2643634.2643666 -l 2 %}.

### Raft

Raft is "an algorithm for managing a replicated log" {% cite 10.5555/2643634.2643666 -l 3 %}. It was designed as an alternative to Paxos, which the Raft authors consider difficult to understand and hard to implement {% cite 10.5555/2643634.2643666 -l 1 %}.

Whereas Paxos focusses on gaining consensus for a single value, Raft focuses on replicating a log across servers. The intention is that the log is then used by servers following the replicated state machine model to execute operations contained in the log entries locally so as to keep their state up-to-date with the leader.

The three subproblems of raft are:

1. Log replication: a leader accepts logs from clients and replicates them across the cluster.
2. Leader election: a new leader must be elected if the current leader fails.
3. Safety: if a server has applied a log entry, then no other server can apply a log entry at the same index.

{% cite 10.5555/2643634.2643666 -l 3-5 %}

#### Log replication

During normal operation, a node is either a leader of a follower. There can only be a single leader.

Initially all nodes are followers and a leader must be elected. The leader accepts requests from clients, replicates the entry to a majority of followers, and then notifies the client that the request has been successful {% cite 10.5555/2643634.2643666 -l 3 %}.

The replication process follows these steps:

1. The leader appends a command to its own log.
2. The leader sends an `AppendEntries()` RPC to each of the followers to replicate the log entry.
3. When the log has been replicated by a follower, it replies with an acknowledgement.
4. When the entry has been replicated successfully to a majority of the machines, the leader applies the entry to its state machine.
5. The leader commits the value and returns the result to the client.

{% cite 10.5555/2643634.2643666 -l 6 %}

#### Leader election

Raft spits time into terms, where each term begins with an election. In some cases, an election results in a split vote. In this case, the term is ended and a new term will begin {% cite 10.5555/2643634.2643666 -l 5 %}.

Terms are used as a logical clock in raft that can detect obsolete information (like stale leaders) {% cite 10.5555/2643634.2643666 -l 5 %}.

A leader maintains its position by sending heartbeats (in the form of an `AppendEntries()` RPC). If a follower does not receive a heartbeats for a period of time, the follower begins an election by increasing its current term value and entering the candidate state. The candidate votes for itself and issues a `RequestVote()` RPCs to the other servers in the cluster {% cite 10.5555/2643634.2643666 -l 6 %}.

A candidate continues in the candidate state until:

1. It wins an election by receiving votes from a majority of the servers for the same term that it's a candidate for.
2. Another server becomes leader.
3. A period of time goes by without a winner.

If votes are split and no candidate becomes a majority, then the candidates will time out and start a new election. Randomized election timeouts are used to ensure that split votes are rare {% cite 10.5555/2643634.2643666 -l 6 %}.

#### Safety

Raft ensures that the following two properties are met for log entries to be replicated safely:

1. If two entries in different logs have the same index and term then they store the same command.
2. If two entries in different logs have the same index and term, then the preceding entries are the logs are identical.

{% cite 10.5555/2643634.2643666 -l 6 %}

The first property is maintained by the fact that there is only one leader that creates one log entry per index {% cite 10.5555/2643634.2643666 -l 7 %}.

The second property is maintained by each follower checking each `AppendEntries()` RPC. Each `AppendEntries()` RPC includes the preceding entry index and term. If the follower does not find an entry matching the term and index, then the follower refuses new entries {% cite 10.5555/2643634.2643666 -l 7 %}.

Leader crashes can cause logs to become inconsistent, where the new leader's logs don't match the logs of the followers. This is handled by the leader forcing followers to copy its own log, meaning conflicting entries in the follower's logs will be overwritten {% cite 10.5555/2643634.2643666 -l 7 %}.

There is also a restriction on which Raft server can be elected leader to ensure that once a server has applied an entry then no other server can apply an entry at that index. The restriction ensures that the leader for a term includes all entries committed in previous terms {% cite 10.5555/2643634.2643666 -l 8 %}.

Since a candidate must gain a majority of servers to become elected, every committed entry must exist in at least one of those servers. A node sends its log information along with the `RequestVote()` RPC. If the recipient has a more up-to-date log, then the recipient denies giving the server its vote {% cite 10.5555/2643634.2643666 -l 8 %}.

## Failover patterns

**Failover** is the process of switching from a failed component to a redundant or standby component.

Two common failover patterns are active-passive failover and active-active failover.

### Active-passive failover

In the **active-passive** failover pattern (also known as master-slave) replicas run alongside the component they will be replacing. If the active server fails, the passive replica becomes active and begins receiving traffic.

Active-passive can be implemented using a virtual IP address that can be assigned dynamically. If the active server goes down, the IP address is reassigned to the passive server.

### Active-active failover

In the **active-active** failover pattern (also known as master-master) multiple replicas serve traffic. In the case of failure, traffic stops being sent to the failed component.

## Message delivery

> There are only two hard problems in distributed systems: 2. Exactly-once delivery 1. Guaranteed order of messages 2. Exactly-once delivery.

**Message delivery** is the process of delivering a message from one node to another in a distributed system.

Message delivery can fail, so systems offer different failure semantics depending on how they handle failure.

With **At-least-once delivery** a sender will keep retrying until it receives a reply. This can result in the same message being delivered multiple times {% cite distributed-systems -l 466 %}.

With **At-most-once delivery** a sender will not retry in the case of failure. This means a message will be delivered once at most and may not be delivered at all {% cite distributed-systems -l 467 %}.

With **Exactly-once delivery** a message is delivered exactly once every time. Exactly-one delivery is obviously ideal, but it can be expensive to achieve {% cite distributed-systems -l 467 %}.

Messaging semantics are commonly encountered when working with message queuing systems like Apache Kafka.

An operation is **idempotent** if it can be performed many times without any damage being done. Making an operation idempotent helps a system deal with at-least-once message delivery {% cite distributed-systems -l 468 %}.

One way to avoid non-idempotent requests being executed multiple times is to add a sequence number to each request. A server would then keep a record of the sequence numbers of executed requests and would not execute any request multiple times (this then introduces the problem of keeping track of many requests) {% cite distributed-systems -l 469 %}.

## Atomic commit protocols

An **atomic commit** is a commit that applies a set of changes as a single operation (such as an SQL transaction). The operation either completes successfully, or it aborts in a way that makes it appear as though the action never took place.

**Atomic commit protocols** are used for performing atomic commits in a distributed system.

### 2PC

2PC (Two-Phase Commit) is an atomic commit protocol that uses a coordinator to tell other nodes whether to commit or not.

As the name suggests, 2PC consists of two phases (provided no failures occur). The two phases are the voting phase and the decision phase {% cite distributed-systems -l 484 %}.

The order of operations are:

The voting phase:

1. The coordinator sends a VOTE-REQUEST message to participants.
2. When a participant receives a VOTE-REQUEST message, it either returns a VOTE-COMMIT message to the coordinator telling it that it's prepared to execute the transaction, or it sends a VOTE-ABORT message.

The decision phases:

1. The coordinator collects votes from participants. If all the participants have voted to commit the transaction, then the coordinator will too and it will send a GLOBAL-COMMIT message to all participants. If one or more participants voted to abort the transaction, then the coordinator multicasts a GLOBAL-ABORT message instead.
2. Participants that voted for a commit will wait for the GLOBAL message from the coordinator. If they receive a GLOBAL-COMMIT message then the participant will commit the transaction, if they receive a GLOBAL-ABORT message then the transaction is aborted.

{% cite distributed-systems -l 484 %}

Timeout mechanisms and logging can be used to ensure the two-phase-commit can handle node failures {% cite distributed-systems -l 485 %}.

Many relational databases use 2PC for distributed atomic commits, for example MySQL, PostgreSQL, and Oracle Database.

## Recoverability

Recoverability is the ability of a system to recover after failure.

There are two forms of error recovery:

1. Backward recovery (rollback)
2. Forward recovery (rollforward)

In **backward recovery**, the purpose is to restore the system to a previous and correct state. To achieve backward recovery, the system must save its state regularly {% cite distributed-systems -l 491 %}.

**Forward recovery** is the act of overcoming an error state and transitioning to a new correct state. In order for forward state recovery to work, the possible error states must be known in advance {% cite distributed-systems -l 491 %}.

The benefit of backward recovery is that it can work for a general failure, rather than a known failure {% cite distributed-systems -l 492 %}.

One downsides to backward recovery is that checkpointing (saving the state) can be expensive {% cite distributed-systems -l 492 %}.

### Checkpointing

**Checkpointing** is the process of regularly saving state (creating a checkpoint). The checkpoint can then be used to restore a system's state in the case of failure {% cite distributed-systems -l 493 %}.

Normally checkpointing is performed by each node. Together, each checkpoint builds a distributed snapshot of a consistent global state {% cite distributed-systems -l 493 %}.

A **distributed snapshot** is a consistent global state, i.e. there will be no misnomers like process $$P$$ receiving message $$M$$, but no recording of message $$M$$ being sent to process $$P$$ {% cite distributed-systems -l 494 %}.

To recover from a failure, processes can recreate the state either locally or globally by using the checkpoints of each process {% cite distributed-systems -l 493 %}.

**Coordinated checkpointing** involves processes coordinating with each other to write their state to local storage. With coordinated checkpointing, saved state can be made globally consistent by the coordinator {% cite distributed-systems -l 494 %}.

**Uncoordinated checkpointing** (also known as independent checkpointing) involves each process independently recording its state periodically. During recovery, checkpoints are rolled back until there is a globally consistent state {% cite distributed-systems -l 494-5 %}.

Uncoordinated checkpointing can lead to **the domino effect** where processes may be forced to rollback to the initial state due to inconsistent global state created by rolling back local states independently {% cite distributed-systems -l 494-5 %}.

An alternative to checkpointing is message logging.

### Message logging

In **message logging**, each process maintains a log of messages that it has received. In the case of failure, the messages can be replayed to rebuild the state.

In order to avoid an unbounded log, checkpoints are periodically created and then used as the starting point for replaying operations {% cite distributed-systems -l 496 %}.

In **sender-based logging**, a process logs its messages before sending them to a server. In **receiver-based logging**, a process first logs a message before passing it to the application {% cite distributed-systems -l 492 %}.

## Zookeeper

Zookeeper is a popular coordination service for distributed systems that uses a consensus algorithm to ensure fault tolerance. It provides fault-tolerant primitives that more complex systems can be built on top of {% cite zookeeper2010 -l 1 %}.

Zookeeper implements an API that manipulates data objects that are organized hierarchically like a file system. The Zookeeper API resembles a file system API (including calls like `create()`, `setData()`, and `getData()`) {% cite zookeeper2010 -l 1,3 %}.

Zookeeper has two ordering guarantees:

1. Linearizable writes. All requests that update Zookeeper's state are serializable and respect precedence.
2. FIFO client order: all requests from a given client are executed in the order that they were sent from the client.

{% cite zookeeper2010 -l 4 %}

To improve the read performance of Zookeeper, reads are weakly consistent, meaning it's possible for reads to receive stale data. Reads can ensure that they get up-to-date data by forcing pending updates to complete with `sync()` {% cite zookeeper2010 -l 4 %}.

Zookeeper can perform a variety of functions in a fault-tolerant manner, for example Zookeeper can:

1. Maintain group membership information.
2. Store system configuration.
3. Be used to elect a master.

Popular projects that use Zookeeper include Hadoop and Kafka (although Kafka voted removed Zookeeper in 2019).

## References

{% bibliography --cited_in_order %}
