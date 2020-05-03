---
layout: default
title: Fault tolerance
description: Notes on fault tolerance in distributed systems.
has_children: true
has_toc: false
nav_order: 5
parent: Distributed systems
permalink: /distributed-systems/fault-tolerance
---

<!-- prettier-ignore-start -->

# Fault tolerance
{:.no_toc}

Fault tolerance is the ability to handle failures. In large distributed systems failures are a common occurrence and so fault tolerance should be built in by design.

<!-- <-- TODO: Add details -->

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

**Fault tolerance** is the ability of a system to continue operating despite partial failures. Achieving fault tolerance is one of the benefits of creating a distributed system {% cite distributed-systems -l 423 %}.

Three important concepts in fault tolerance are availability, reliability, and recoverability.

**Availability** is "the property that a system is ready to be used immediately". It's usually measured as a percentage of uptime (e.g., 99.99%) {% cite distributed-systems -l 424 %}4.

**Reliability** is the property that a system can run continuously without failure.

The difference between reliability and availability can be thought of like this: "if a system goes down on average for one, seemingly random, millisecond every hour, it has an availability of more than 99.999 percent, but is still unreliable. Similarly, a system that never crashes but is shut down for two weeks every August has high reliability but only 96 percent availability" {% cite distributed-systems -l 424 %}.

**Recoverability** is the ability of a system to recover after becoming unavailable due to failure. This could be achieved by saving the system state to non-volatile storage (in the form of logs or checkpoints) or by replicating data across nodes.

A system is said to **fail** when it's unable to provide its service to users {% cite distributed-systems -l 426 %}.

An **error** is a "part of a system's state that may lead to a failure". The cause of an error is a **fault** {% cite distributed-systems -l 426 %}.

There are different types of fault:

- **Transient faults** that occur once and then disappear.
- **Intermittent faults** that occur periodically 427.
- **Permanent faults** that exists until the cause of the fault is fixed 427.

{% cite distributed-systems -l 427 %}

**Fail-stop failures** are crash failures that can be reliably detected where the computer stops executing (notably this means the computer won't compute incorrect results) {% cite distributed-systems -l 430 %}. These are the kind of failures that we can handle with approaches such as replication.

There are other modes of failure: bugs in software, and bugs in hardware. These won't be solved by replication.

### Failure detection

The main approach to failure detection is to continuously send heartbeat messages between nodes. A timeout system is used—if messages from process $$P$$ stop for a certain period of time, then process $$Q$$ suspects that $$P$$ is down 462.

Process $$Q$$ can't know for certain that process $$P$$ is down 463. In an asynchronous distributed system, it's not possible for a node to determine for certain whether another node has failed or whether it is simply unreachable {% cite distributed-systems -l 430 %}.

Failure detection can be done at the same time as exchanging information, such as during gossip dissemination (could read: http://www2003.org/cdrom/papers/alternate/P712/p712-vogels.html) 463.

Failure detection systems should be able to differentiate between network failures and node failures 464.

## Redundancy

Redundancy is where duplicate components are provided in order to improve the reliability of a system. If one of the components fails, the others can continue executing.

There are several types of redundancy:

- Information redundancy
- Time redundancy
- Physical redundancy

With **information redundancy**, extra bits are used to provide the ability to recover from flipped/incorrectly read bits (e.g., checksums and hamming codes) {% cite distributed-systems -l 431 %}.

With **time redundancy**, operations are retried in the case of timeouts {% cite distributed-systems -l 431 %}.

With **physical redundancy**, extra physical components are added to a system so that it can tolerate with the loss of some components. Physical redundancy can either be physical or in software {% cite distributed-systems -l 431 %}.

A system is k-fault-tolerant if it can handle the failure of k components {% cite distributed-systems -l 435 %}.

One way to improve resiliency is to group processes together into process groups that are treated as a whole. The loss of one or more processes in the group won't break the system. 433.

In some groups, all processes are equal, in other groups (like a database cluster), some processes will be designated special roles.
In a process group where nodes contain shared state, each node should execute the same commands in the same order as all every other nonfaulty processes. This means nodes need to reach consensus on a command to execute {% cite distributed-systems -l 433, 436 %}.

## Consensus

A consensus algorithm is used to allow a collection of machines ...


One use of consensus algorithm is to keep the sequence of replicated logs consistent between replicated state machines 2. A server would receive commands from clients and add them to its log, it would then communicate with the consensus modules on other servers to ensure that the replicated log contains the same commands in the same order 2.

A practical consensus algorithm will:

- Ensure safety: never return an incorrect value despite failures (except for Byzantine faults).
- Available: as long as the majority of machines are running and can communicate with each other.
- Do not depend on clocks to ensure the consistency of logs.
- A command can complete once the majority of a cluster has agreed to it.

2

a single value to keep in the case that multiple values are proposed 1.

Consider multiple processes that can propose values.

### Paxos

Paxos is a distributed consensus protocol that can handle failures of some participating processes 438.

The aim of Paxos is to agree upon a single proposed value {% cite Lamport2001PaxosMS -l 1 %}.

There are five roles in Paxos:

1. Client
2. Proposer
3. Acceptor
4. Learner
5. Leader

_Note: in an implementation, a single process can perform multiple roles 1._

**Clients** send requests to the system and wait for a response.

**Proposers** advocates a client request and send the proposed value to a set of **acceptors** which may accept the value. A value is chosen when a majority of acceptors have accepted it {% cite Lamport2001PaxosMS -l 2 %}.

**Learners** learn the chosen value and can take action on it.

The **leader** (a distinguished proposer) can be used as the sole proposer to avoid stalemate problems.

In order to determine which proposal should win in the cse of multiple proposals, each proposal is assigned an increasing number $$n$$.

There are two phases to the Paxos algorithm: promise and accept.

- Promise
  - A proposer selects a proposal number $$n$$ and sends a prepare request with $$n$$ to the acceptors.
  - If an acceptor receives a prepare request with number $$n$$ greater than the number of any prepare request that it has already responded to, then the acceptor responds to the request with a promise not to accept any more proposals numbered less than $$n$$ along with the highest-numbered proposal (if any) that it has accepted.
- Accept
  - If the proposer receives a response to its prepare requests (numbered $$n$$) from a majority of acceptors, then it sends an accept request to each of those acceptors for a proposal numbered $$n$$ with a value $$v$$, where $$v$$ is the value of the highest-numbered proposal among the responses, or is any value if the responses reported no proposals.
  - If an acceptor receives an accept request for a proposal numbered $$n$$, it accepts the proposal unless it has already responded to a prepare request having a number greater than $$n$$.

{% cite Lamport2001PaxosMS -l 5-6 %}

Once a value has been accepted, each learner needs to learn of the value. There are several ways this could be done. One way would be to have each acceptor send a message to each learner whenever an acceptor accepts a proposal {% cite Lamport2001PaxosMS -l 8 %}.

A leader can be chosen, which becomes the only node acting as proposer. This is in order to avoid a situation where multiple proposers will continue to issue proposals that overrule each other before a proposal has been accepted {% cite Lamport2001PaxosMS -l 7 %}.

A new leader can be elected in the case of a leader failing {% cite Lamport2001PaxosMS -l 7 %}.

<!-- Paxos focusses on agreeing to a single value. In reality, Paxos often has to be run -->

Paxos defines a protocol that reaches decisions on a single value. It can then be run multiple times to handle a series of decisions, like agreeing on a log 2. One of the issues with Paxos is that it focusses on the single instance, whereas most implementations will run Paxos many times to agree to multiple decisions 2. Raft is an alternative protocol that was designed with multiple decisions in mind.

### Raft

Raft is "an algorithm for managing a replicated log" 3. It was designed as an alternative to Paxos, which the Raft authors consider difficult to understand and hard to implement 1.

Whereas Paxos focusses on agreeing consensus for a single value, Raft focuses on replicating a log across servers. The intention is that this log is then used by servers following the distributed state machine model to execute operations contained in the log entries locally so as to keep their state up-to-date with the leader NEEDS CITATION.

The three subproblems of raft are:

1. Log replication: a leader accepts logs from clients and replicates them across the cluster.
2. Leader election: a new leader must be elected if the current leader fails.
3. Safety: if a server has applied a log entry, then no other server can apply a log entry at the same index.

3-5

#### Log replication

During normal operation, a node is either a leader of a follower. There can only be a single leader.

Initially all nodes are followers and a leader must be elected. The leader accepts requests from clients, replicates the entry to a majority of followers, and then notifies the client that the request has been successful 3.

The replication process follows these steps:

1. The leader appends a commands to its own log.
2. It sends an `AppendEntries()` RPC to each of the followers to replicate the log entry.
3. Wen the log has been replicated by a follower, it replies with an acknowledgement.
4. When the entry has been replicated successfully to a majority of the machines, the leader applies the entry to its state machine.
5. The leader then informs the followers that the value has been commited (check)
6. The leader returns the result to the client.

6

#### Leader election

Raft spits time into terms, where each term begins with an election. In some cases, an election results in a split vote. In this case, the term is ended and a new term will begin 5.

Terms are used as a logical clock in raft used to detect obsolete information (like stale leaders) 5.

A leader maintains its position by sending heartbeats (in the form of an AppendEntries RPC). If a follower does not receive a heartbeats for a period of time (what period of time?) the follower begins an election by increasing its current term value and entering the candidate state 6. The candidate votes for itself and issues `RequestVote()` RPCs to the other servers in the cluster 6.

A candidate continues in the candidate state until:

1. It wins an election by receiving votes from a majority of the servers for the same term that it's a candidate for.
2. Another server becomes leader.
3. A period of time goes by without a winner.

If votes are split and no candidate becomes a majority, then the candidates will time out and start a new election 6. Randomized election timeouts are used to ensure that split votes are rare 6.

#### Safety

Raft ensures that the following two properties are met for log entries to be replicated safely:

1. If two entries in different logs have the same index and term then they store the same command.
2. If two entries in different logs have the same index and term, then the preceding entries are the logs are identical.

6

The first property is maintained by the fact that there is only one leader that creates only one log entry per index 7.

The second property is maintained by each follower checking each `AppendEntries()` RPC. Each AppendEntries RPC includes the preceding entry index and term. If the follower does not find an entry matching the term and index, then the follower refuses new entries 7.

Leader crashes can cause logs to become inconsistent, where the new leader's logs don't match the logs of the followers. This is handled by the leader forcing follower's to copy its own log, meaning conflicting entries in follower's logs will be overwritten 7.

There is ALSO a restriction on which Raft server can be elected leader to ensure that once a server has applied an entry then no other server can apply an entry at that index. The restriction ensures that the leader for a term includes all entries committed in previous terms 8.

Since a candidate must gain a majority of servers to become elected, every committed entry must exist in at least one of those servers. A node sends its log information along with the RequestVote RPC. If the recipient has a more up-to-date log, then the recipient denies giving the server its vote 8.

## Client-server communication

### RPC during failure

## Message delivery

In the case of server crashes, there are several options for how an RPC implementation can react.

In the event of a server crash, the client should react differently depending on whether the server crashed before executing the request, or whether the server crashed after executing the request. The client cannot determine this in the case of a crash, since all it knows is that it's timer has expired 466.

<!-- TODO: this section should be about queues since they often deal with these semantics explicitly -->

**At-least-once** semantics the client will keep retrying until it receives a reply. There are several cases where this will result in multiple executions on the server p466.

An alternative is at-most-once semantics. This is achieved by giving up an reporting failure 467.

**Exactly-once** semantics is the ideal. This means that a request is executed exactly once every time 467.

https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/

<!-- TODO: read https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/-->

### Lost reply messages

One approach to dealing with lost reply messages is to use a timer. After the timer has expire, it's assumed that a reply message has been lost 468.

An **idempotent** operations is one that can be performed many times without any damage being done 468.

An example of a request that is not idempotent would be a request to transfer money from one account to another. If this was performed multiple times, then multiple times more money would be transferred than should have been 469.

One way to solve the problem of non-idempotent requests being executed multiple times is by adding a sequence number to each request. A server would then keep record of the sequence numbers of executed requests, and would not execute any request multiple times (although this introduces the problem of keeping track of many requests) 469.

### Reliable multicast

A reliable multicast protocol is a protocol that provides a reliable sequence of packets to multi receivers.

Atomic multicast is a communication primitive where messages are delivered to multiple groups of processes according to a total order 477.

## Message ordering

"There are only two hard problems in distributed systems: 2. Exactly-once delivery 1. Guaranteed order of messages 2. Exactly-once delivery"

Tennenbaum lists four different orderings of multicasts:

1. Unordered multicasts
2. FIFO-ordered-multicasts
3. Causally-ordered multicasts
4. Totally-ordered multicasts

A reliable unordered multicast is a virtually synchronous multicast where no guarantees around the order of received messages are given 480.

Reliable FIFO-ordered multicasts delivers incoming messages from the same process in the same order that they were sent 480. There are no guarantees around messages sent from different processes.

Reliable causally ordered multicasts will deliver messages in an order so that causality between messages is preserved. Causally ordered multicasts can be implemented using Vector timestamps 481.

Total-ordered delivery guarantees that messages are delivered in the same order to each process (whether the other ordering is FIFO, unordered or causally ordered) 481.

Virtually synchronous reliable multicasting with total-order delivery of messages is called atomic multicasting 481.

## Atomic commit

An atomic commit is an operation that applies a set of changes as a single operation.

Atomic commit protocols usually uses a coordinator to tell other nodes whether or not to locally perform operations {% cite distributed-systems -l 484 %}.

A common atomic commit protocol is 2PC.

### 2PC

2PC (Two-Phase Commit) is an atomic commit protocol.

As the name suggests, 2PC consists of two phases (provided no failures occur). The two phases are the voting phase and the decision phase 484.

The order of operations are:

1. The voting phase
   2. The coordinator sends a VOTE-REQUEST message to participants.
   3. When a participant receives a VOTE-REQUEST message, it either returns a VOTE-COMMIT message to the coordinator telling it that it's prepared to execute an transaction, or it sends a VOTE-ABORT message.
1. The decision phases
   1. The coordinator collects votes from participants. If all the participants have voted to commit the transaction, then the coordinator will too and it will send a GLOBAL-COMMIT message to all participants. If one participant had voted to abort the transaction, then the coordinator multicasts a GLOBAL-ABORT message instead.
   1. Participants that voted for a commit will wait for the GLOBAL message from the coordinator. If they receive a GLOBAL-COMMIT message then the participant will commit the transaction, if they receive a GLOBAL-ABORT message then the transaction is aborted.

{% cite distributed-systems -l 484 %}

Timeout mechanisms are used to ensure the two-phase-commit can handle node failures {% cite distributed-systems -l 485 %}.

Many relational databases use 2PC for distributed atomic commits, for example MySQL, PostgreSQL, and Oracle Database.

MySQL Cluster uses 2PC for replicating data synchronously.

<!-- TODO: read paper on 2PC (https://lamport.azurewebsites.net/video/consensus-on-transaction-commit.pdf) -->

## Recoverability

Recoverability is the ability of a system to recover after failure.

There are two forms of error recovery:

1. Backward recovery.
2. Forward recovery.

In backward recovery, the purpose is to restore the system to a previous and correct state. To achieve backward recovery, the system must save its state regularly {% cite distributed-systems -l 491 %}.

Forward recovery is the act of overcoming an error state and transitioning to a new correct state. In order for forward state recovery to work, the possible error states must be known (verify this) {% cite distributed-systems -l 491 %}.

The benefit of backward recovery is that it can work for a general failure, rather than a known failure {% cite distributed-systems -l 492 %}.

The problem of backward recovery is that checkpointing (saving the state) can be expensive, recovering is often a costly operation. Also, there's no guarantee that a machine that failed won't fail again once recovery has taken place {% cite distributed-systems -l 492 %}.

<!-- To reduce the cost of checkpointing, many distributed systems combine checkpointing with logging. In sender-based logging, a process logs its messages before sending them to a server (add more on this). In receiver-based logging, ?? -->

### Checkpointing

Checkpointing involves regularly saving state. Normally it's performed by each node, which together builds a distributed snapshot of a consistent global state {% cite distributed-systems -l 493 %}.

To recover from a failure, processes can recreate the state either locally or globally by using the checkpoints of each process {% cite distributed-systems -l 493 %}.

Coordinated checkpointing involves processes coordinating with each other to write their state to local storage. With coordinated checkpointing, saved state is automatically globally consistent 4{% cite distributed-systems -l 494 %}94.

One way to implement coordinated checkpointing is to use a two-phase blocking protocol. A coordinator first multicasts a CHECKPOINT-REQUEST message to all processes. When a process receives a message, it takes a local checkpoint,queues messages that it receives, and acknowledges that it has taken a checkpoint. When the coordinator has received an acknowledgement message from all processes, it multicasts CHECKPOINT-DONE to the processes which can then continue executing {% cite distributed-systems -l 494 %}.

Independent checkpointing involves each process independently recording its state periodically {% cite distributed-systems -l 494-5 %}.

<!-- TODO: more research into checkpointing -->

### Message logging

Message logging can be used in place of checkpointing. With message logging, logs are recorded. The transmission of messages can then be replayed to reach a globally consistent state without having to restore the state from local storage 496. A checkpointed state is taken as a starting point, and messages that have been sent since the checkpoint are retransmitted and handled correctly {% cite distributed-systems -l 496 %}.

<!-- TODO: define message logging -->

## References

{% bibliography --cited_in_order %}
