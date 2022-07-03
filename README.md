# CRDT

## Conflict-free replicated data type

In distributed computing, a conflict-free replicated data type (CRDT) is a data structure that is replicated across multiple computers in a network, with the following features:

- The application can update any replica independently, concurrently and without coordinating with other replicas.
- An algorithm (itself part of the data type) automatically resolves any inconsistencies that might occur.
- Although replicas may have different state at any particular point in time, they are guaranteed to eventually converge.

## Background

Concurrent updates to multiple replicas of the same data, without coordination between the computers hosting the replicas, can result in inconsistencies between the replicas, which in the general case may not be resolvable. Restoring consistency and data integrity when there are conflicts between updates may require some or all of the updates to be entirely or partially dropped.

Accordingly, much of distributed computing focuses on the problem of how to prevent concurrent updates to replicated data. But another possible approach is optimistic replication, where all concurrent updates are allowed to go through, with inconsistencies possibly created, and the results are merged or "resolved" later. In this approach, consistency between the replicas is eventually re-established via "merges" of differing replicas. While optimistic replication might not work in the general case, there is a significant and practically useful class of data structures, CRDTs, where it does work — where it is always possible to merge or resolve concurrent updates on different replicas of the data structure without conflicts. This makes CRDTs ideal for optimistic replication.

As an example, a one-way Boolean event flag is a trivial CRDT: one bit, with a value of true or false. True means some particular event has occurred at least once. False means the event has not occurred. Once set to true, the flag cannot be set back to false. (An event, having occurred, cannot un-occur.) The resolution method is "true wins": when merging a replica where the flag is true (that replica has observed the event), and another one where the flag is false (that replica hasn't observed the event), the resolved result is true — the event has been observed.

## Types of CRDTs

There are two approaches to CRDTs, both of which can provide strong eventual consistency: operation-based CRDTs and state-based CRDTs.

The two alternatives are theoretically equivalent, as each can emulate the other.[1] However, there are practical differences. State-based CRDTs are often simpler to design and to implement; their only requirement from the communication substrate is some kind of gossip protocol. Their drawback is that the entire state of every CRDT must be transmitted eventually to every other replica, which may be costly. In contrast, operation-based CRDTs transmit only the update operations, which are typically small. However, operation-based CRDTs require guarantees from the communication middleware; that the operations are not dropped or duplicated when transmitted to the other replicas, and that they are delivered in causal order.

**Operation-based CRDTs**
Operation-based CRDTs are also called commutative replicated data types, or CmRDTs. CmRDT replicas propagate state by transmitting only the update operation. For example, a CmRDT of a single integer might broadcast the operations (+10) or (−20). Replicas receive the updates and apply them locally. The operations are commutative. However, they are not necessarily idempotent. The communications infrastructure must therefore ensure that all operations on a replica are delivered to the other replicas, without duplication, but in any order.

Pure operation-based CRDTs are a variant of operation-based CRDTs that reduces the metadata size.

**State-based CRDTs**
State-based CRDTs are called convergent replicated data types, or CvRDTs. In contrast to CmRDTs, CvRDTs send their full local state to other replicas, where the states are merged by a function which must be commutative, associative, and idempotent. The merge function provides a join for any pair of replica states, so the set of all states forms a semilattice. The update function must monotonically increase the internal state, according to the same partial order rules as the semilattice.

Delta state CRDTs (or simply Delta CRDTs) are optimized state-based CRDTs where only recently applied changes to a state are disseminated instead of the entire state.

**Comparison**
While CmRDTs place more requirements on the protocol for transmitting operations between replicas, they use less bandwidth than CvRDTs when the number of transactions is small in comparison to the size of internal state. However, since the CvRDT merge function is associative, merging with the state of some replica yields all previous updates to that replica. Gossip protocols work well for propagating CvRDT state to other replicas while reducing network use and handling topology changes.

Some lower bounds on the storage complexity of state-based CRDTs are known.
