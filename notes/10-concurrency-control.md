[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Concurrency Control
* **Concurrency Control** is a mechanism to guarantee isolation (or serialisability) in transactions
* Three typical concurrency algorithms:
	* Using locks
	* Using timestamps
	* Optimistic

## Using Locks
* This is the oldest and most widely used concurrency control algorithm
* When a process needs to access a shared resource as a part of a transaction, it first locks the resource
* If the resource is locked by another process, the requesting process waits until the lock is released
* The basic scheme is overly restrictive — the algorithm can be improved by distinguishing read locks from write locks
* Data only to be read (referenced) is locked in *shared* mode
* Data needs to be updated (modified) is locked in *exclusive* mode
* Lock modes have the following compatibility
	* If a resource is not locked or locked in shared mode, a transaction can lock the resource in shared mode
	* A transaction can lock a resource in exclusive mode only if the resource is not locked
* If two transactions are trying to lock the same resource in incompatible mode, the two transactions are in *conflict*
* A conflict relation can be:
	* **shared-exclusive** (read-write conflict)
	* **exclusive-exclusive** (write-write conflict)

### Two Phase Locking
* Using locks does not guarantee serialisability
* Only transactions executed concurrently as follows will be serialisable
	1. A transaction locks shared resource before accessing the resource and releasing the lock after using it
	2. Locking is granted according to compatibility constraint
	3. A transaction does not require any locks after releasing a lock
* The usage of locks satisfies above conditions are 2 phase lock protocol
* A transaction acquires all necessary lock in the first phase (“growing phase”)
* The transaction releases all locks in the second phase (“shrinking phase”)
* Two phase locking is the sufficient condition to guarantee serialisability
* Suppose a transaction aborts after releasing some of the locks — if the other transactions have used resources protected by these released locks, the transactions must also be aborted — **cascading abort**

### Strict Two Phase Locking
* To avoid the cascading abort, many systems use **strict two phase locking** — in which transactions do not release any lock until commit is completed
* Strict two phase locking is done as follows
	1. Begin transaction
	2. Acquire locks before read or write resources
	3. Commit
	4. Release all locks
	5. End transaction

#### Advantages
* A transaction always reads a value written by a committed transaction
* A transaction never has to be aborted — cascading never happens
* All lock acquisitions and releases can be handled by the system without the transaction being aware of these
* Locks are acquired whenever data are accessed and released when the transaction has finished

#### Disadvantages
* Two phase locking can cause deadlocks to occur
* Two processes try to acquire the same pair of locks but in the reverse order

### Lock Granularity
* The size of resources that can be locked by a single lock is called **lock granularity**
* The smaller the granule, the more concurrent processing is possible (though requires more locks)
* e.g. compare locking by file and by record contained in a file:
	* Suppose two transactions access different records in the same file — if the record is the unit of lock, two transactions can access the file simultaneously
	* On the other hand, if the file is the unit of lock, the two transactions cannot access the data simultaneously

## Using Timestamps
* Assign each transaction a timestamp once they begin
* Timestamps must be unique (ensuring uniqueness using Lamport’s algorithm i.e. “Happens-Before relation”)
* Each resource has a read timestamp and a write timestamp
* A **read timestamp** is the timestamp of the transaction which read the resource most recently — let TRD(x) be the read timestamp of resource X
* A **write timestamp** is the timestamp of the transaction which updated the resource most recently — let TWR(x) be the write timestamp of resource X
* Note that read and write timestamps are not the values of actual time when the data items are actually being read or written

### Process
* When a transaction with timestamp T tries to read resource X
	* If T < TWR(x), that means the transaction needs to read a value of X that is already overwritten, hence the request is rejected and the transaction is aborted
	* If T ≥ TWR(x), the transaction is allowed to read the resource
* When a transaction with timestamp T tries to update resource X
	* If T < TWR(x), the transaction is attempting to write an obsolete value of X — the request is rejected and the transaction is aborted
	* If T < TRD(x), then the value of X that the transaction is producing was needed previously, and the system assumed the value would never be produced — the request is rejected and the transaction is aborted
	* Otherwise, the transaction is allowed to update the resource
* The timestamp-ordering protocol guarantees serialisability since all arcs in the precedence graph are of the form:
	* Transaction with smaller timestamp → Transaction with larger timestamp
* Thus, there will be no cycles in the precedence graph
* Timestamp protocol ensure freedom from deadlock as no transaction ever waits
* But the schedule may not be recoverable

## Optimistic
* Run the transactions, be optimistic about transaction conflicts
* At the end of a transaction (i.e. when it is committing) existence of conflict is examined — if no conflict is detected, the local copies are written on the real resource — otherwise, aborted
* Progression is done in phases
	1. Read & private write
	2. Validate
	3. Write
* Conflict check is done by examining *whether the resource used by this transaction is updated by other transactions while this transaction was executing*
* To make this check possible, the system must keep track of the resources used by the transactions

### Time-stamping
* Each transaction is assigned a timestamp TS(Ti) at the beginning of the validation phase

### Validation Conditions
* For every pair of transactions Ti, Tj, such that TS(Ti) < TS(Tj), one of the following conditions must hold:
	* Ti completes (all three phases) before Tj begins
	* Ti completes before Tj starts its write phase, and Ti does not write any object read by Tj
	* Ti completes the read phase before Tj completes its Read phase and Ti does not write any object that is either read or written by Tj

### Observations
* Optimistic concurrency control is designed with the assumption that conflicts are rare and we do not need to abort transactions often
* Update is performed on local copies — this method fits well with the private workspace
* The advantage is that it allows maximum parallel execution since transactions never wait and deadlocks never occur
* The disadvantage is that a transaction may be aborted at the very end since the conflict check is done at the commit point — all operations of the aborted transaction must be done again from the start