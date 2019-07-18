---
layout: post
title: "Atomic Transactions"
subtitle: "Learning Notes for CMU Course 15640 Distributed Systems"
date: 2019-03-26 23:45:13 -0400
background: '/img/posts/04.jpg'
---
Recap: Atomic Transactions
- No return calls
- A transaction is a group of actions
    - All succeed and the effects are permanent
    - None succeed, then no visible effects
    - Properties (ACID)
        - Atomicity - all or nothing
        - Consistency - execution on a consistent state leaves data consistent regardless of commit/abort
            - Application level, in a correct state
        - Isolation - as If only one transaction running
        - Durability - permanence
- Shadowing - Oldest technique for atomicity
    - Garbage collection is side effect
    - Garbage doesn't have to be collected right away
    - Based on atomic pointer swing - pointer assignment
        - Carefully construct shadow copy
        - Pointer have to be in storage
        - Contains all modifications
        - Original unchanged
        - Swing one pointer: commit point
    - Typical for hierarchical data, e.g. hierarchical file system
         Change on the least common ancestor
    - Weak: high LCA (COW can help)
    - Good: recover fast. Easy understand
- Intentions lists
    - Commit: write out completion record
    - Apply changes to stable storage
    - Keep a list of proposed changes in "stable storage", not only in changing transient state
    - On abort: delete intention lists
    - Write-Ahead Logging - journalling
        - Append only logging to record intentions
        - The log preserves of all the order (in between the concurrent transactions!)
        - For Debugging
    - Log-Based Recovery
        - Idempotent - with redos or undos
    - Caveats
        - System kills transactions if too long
        - Lock until end of transaction
        - Log head cannot push when there's pink(uncommitted)
        - The log (in stable storage) is for crash recovery, red -> commit and push, pink -> abort
        - If the cache memory is infinite, no need for pushing the log to committed state
        - Al long as the log has length, the log has some truth, the master copy is not 100% truth.
        - So each time a var is visited, go through the stable storage
        
            

