---
layout: default
title: Durability
description: Durability in RDBMSes.
has_children: true
has_toc: false
parent: Databases
nav_order: 6
permalink: /databases/durability
---

<!-- prettier-ignore-start -->

# Durability
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

In ACID, **durability** guarantees that committed transactions will survive permanently despite failures (e.g. system failures or hardware failures).

Recovery algorithms are techniques to ensure database consistency, transaction atomicity, and durability.

_Note: Storage media failures, where the storage media is corrupted, cannot be recovered from. In this case the DB would need to be restored from an archived version._

{% cite 15445-notes-20 %}

Most DBMSes support durability using a variation of ARIES—a recovery method that uses write-ahead logging.

## Buffer Management Policies

A **steal policy** defines whether the DBMS allows transactions to write uncommitted changes to disk. STEAL means that blocks modified by active transactions can be flushed to disk, whereas NO-STEAL means blocks modified by active transactions cannot be flushed to disk.

A **force policy** defines whether the DBMS is required to flush to disk all blocks modified by a transaction before the transaction is committed. FORCE means this is required, NO-FORCE means it's not. FORCE makes it easier to recover from but results in poor performance.

{% cite 15445-notes-20 %}

## Write-ahead logging

Write-ahead logging (WAL) is a technique where changes are first written to a log (on stable storage), before they are written to a disk page.

WAL is a common technique used to provide atomicity and durability. Almost every DBMS uses WAL. The log contains sufficient information to perform the necessary actions to restore the DB after a crash.

WAL is a STEAL + NO FORCE system.

{% cite 15445-notes-20 %}

### Logging schemes

There are several logging schemes that specify the format of each log line.

In **physical logging** the log records changes made to a specific location in the database (a diff).

In **logical logging** the log records high-level operations that were executed.

Logical logging requires less data to be written out than physical logging, but it's difficult to recover using logical logging since it might not provide enough information to know which parts of the database may have been modified by a query before a crash (e.g. an UPDATE that runs across the whole database but crashes before completing).

One solution to this problem is physiological logging. In **physiological logging** the log records the changes at page-level but does not specify the data organization of the page. Physiological logging is the most common approach.

## Checkpointing

**Checkpointing** is a technique used to reduce the amount of work required to recover a database. It involves creating a checkpoint at which all changes are guaranteed to have been flushed to disk.

An example implementation of blocking checkpointing would be:

1. Stop accepting transactions.
2. Wait for all active transactions to complete.
3. Flush dirty pages to disk.
4. Add a CHECKPOINT entry to the WAL.

Blocking checkpointing can suffer from performance problems.

**Fuzzy checkpointing** is a technique where two checkpoint log records are made: CHECKPOINT-BEGIN (when the checkpointing process began) and CHECKPOINT-END (where the checkpointing process ends). The CHECKPOINT-END record contains the ATT (Active Transactions Table) and DPT (Dirty Page Table) (see ARIES section).

{% cite 15445-notes-21 %}

## ARIES

ARIES is a transaction recovery method, defined in the [ARIES paper](https://web.stanford.edu/class/cs345d-01/rl/aries.pdf). Most DBs use something similar to ARIES for recovery.

_Note: These notes make several assumptions to simplify discussion: all log records fit within a single page, disk writes are atomic, the buffer manager uses STEAL + NO-FORCE with WAL, the RDMS uses Strict 2PL. For comprehensive information on ARIES, see the ARIES paper._

ARIES has three phases:

1. Analysis
2. Redo
3. Undo

The Analysis phase involves reading the WAL from the last checkpoint to identify dirty pages in the buffer pool, and active transactions at the time of the crash.

The Redo phase involves repeating all actions starting from an appropriate point in the log where there can be potential changes from transactions that weren't flushed to disk (even transactions that will abort).

The Undo phase involves reversing the actions of transactions that did not commit before crash.

Two data structures are used:

- ATT (Active Transactions Table) holds the active transactions.
- DPT (Dirty Page Table) holds all pages that have been modified but not yet flushed to disk.

### LSNs

A log sequence number (LSN) is a unique monotonically increasing number that identifies a log record in the WAL.

The LSNs used in ARIES are:

- `flushedLSN`—the last LSN flushed to disk.
- `pageLSN`—newest update to page P (stored on P).
- `recLSN`—oldest update to page P since it was flushed (stored on P).
- `lastLSN`—the last LSN that a transaction T created in WAL (stored on T).
- `masterLSN`—the LSN of the latest checkpoint taken.
- `prevLSN`—added to log records. This is previous LSN for a given transaction.

{% cite 15445-notes-21 %}

### Normal execution

Before a page $$P$$ is written to disk, the log must be flushed so that $$\text{pageLSN}(\text{P}) \le \text{flushedLSN}$$.

#### Transaction commit

Writing a COMMIT record to the log requires that all log records up to (and including) the transaction's COMMIT record are flushed to disk.

When a COMMIT succeeds, a special TRANSACTION-END record is written to the log. This entry doesn't need to be flushed—it is an internal record that verifies that we can remove bookkeeping data about the transaction.

#### Transaction abort

Aborting a transaction is a special case of the ARIES Undo operation applied to only one transaction.

A CLR (Compensation Log Record) is written to the log during abortion. A CLR describes the actions taken to undo the actions of a previous update record. It has all the fields of an update log record plus an `lastLSN` pointer.

The abort algorithm involves writing an ABORT record to log and then playing back the transaction's updates in reverse order. For each record, a CLR is written to the log, before the old value is restored. When a transaction is fully aborted, a TRANSACTION-END record is written to the log.

### Recovery

#### Analysis

The Analysis phase involves restoring the DPT (Dirty Page Table) and ATT (Active Transaction Table) by scanning forward in the logfile from the start or from the last successful checkpoint.

For all TRANSACTION-BEGIN records, the transaction is added to the ATT with status UNDO. When a TRANSACTION-END record is encountered, the corresponding transaction status is set to COMMIT.

During the same run, the DPT is filled by adding a new entry when a page is modified but is not yet in the DPT, and the page's `recLSN` is set to the log record's LSN.

#### Redo

The Redo phase involves applying all changes in the log starting from a point where there was a modified dirty page that may not have made it to disk.

The point to scan forward from is the log record containing the smallest recLSN in the DPT.

For each update, log record, or CLR with a given LSN, the action is redone unless:

- The affected page is not in the DPT.
- The affected page is not in the DPT and the record's LSN is less than the page's recLSN.

Redoing an action involves reapplying the logged action and then setting the page's `pageLSN` to the log record's LSN.

At the end of Redo Phase, a TRANSACTION-END log record is written for each transaction with status COMMIT, and the transaction is removed from the ATT.

### Undo phase

The Undo phase involves undoing all transactions that were active at the time of the crash and therefore never committed (these are all the transactions with status UNDO in the ATT after the analysis phase).

The steps are:

1. Process the transactions in reverse order (using `lastLSN` and `prevLSN` to speed up traversal).
2. Write a CLR for every modification.

## References

{% bibliography --cited_in_order %}
