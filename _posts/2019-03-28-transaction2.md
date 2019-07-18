---
layout: post
title: "Transactions & Raplication Basics"
subtitle: "Learning Notes for CMU Course 15640"
date: 2019-03-28 23:45:13 -0400
background: '/img/posts/04.jpg'
---

## Distributed Transactions

Two-phase commit  - indefinite blocking window
- All sites who voted positive are blocked until outcome is known
- Unanimous agreement on all participants

Three-phase commit or 3PC
- Only helpful when there's not network partition
- There's no protocol resilient to network partitioning when messages are lost.
- "Price of certainty" - if want to make sure all votes - then have to block
    - Safety - correct - more important
    - Liveness - no blocking - have to sacrifice safety

High Availability
- Taxonomy 
    - Part 1 - Data - persistent storage - replication - read-only replication
        - If Turing machine has a non-volatile tape
        - "ACID" Durability
    - Part 2 -  computation

## Replication basics 
- High availability technique for data
- Assume site failures are uncorrelated - only works when this is satisfied
- Multiple copies of data at different state
- One-copy semantics for transparency requires
- "Replica control algorithm"
- Fundamental tradeoffs
    - Overhead for one copy semantics - cost overhead! When mach replicas
    - When failure occur, the application might not work, because of the inconsistency of one-copy semantics
    - Availability and consistency tradeoff!!
        - E.g. failure - network partitions
        - Can't have both when there's failure
- Replica control algorithm
    - Optimistic - violates the one copy semantics, allow updates in all partitions
    - Pessimistic - updates in at most one partition - one copy semantics would work


