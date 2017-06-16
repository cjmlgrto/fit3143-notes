[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Faults
## Dependability
* **Availability** — system is ready to be used immediately
* **Reliability** — system can run continuously without failure
* **Safety** — when a system (temporarily) fails to operate correctly, nothing catastrophic happens
* **Maintainability** — how easily a failed system can be repaired

## Total vs Partial Failure
* **Total** — all components in a system fail
* **Partial** — one ore more (but not all) components in a distributed system fail

## Failure
* **Failure** — when a system does not meet its promises or cannot provide services in the specified manner
* **Error** — part of the system state that leads to failure (i.e. differs from intended value)
* **Fault** — the cause of an error (results from design errors, manufacturing problems, etc.)
* Recursive:
	* Failure may be initiated by a mechanical fault
	* Manufacturing fault leads to disk failure
	* Disk failure is a fault that leads to database failure
	* Database failure is a fault that leads to email failure
	* etc.

## Classification of Failures
### Crash Failure
* When a process permanently ceases to execute its actions
* Irreversible
* In async, crashes cannot be detected with total certainty (since there is no lower bound of the speed at which a process can execute its actions)
* In sync, crash failures can be detected using timeouts

### Omission Failure
* If the receiver does not receive one or more messages sent by the transmitter

### Transient Failure
* Can disturb the state of processes in an arbitrary way
* The agent inducing this problem may momentarily be active but can make a lasting effect on the global state (e.g. power surge, short circuit, etc.)

### Byzantine Failure
* Weakest of all failures
* Assume process i follows the value of x of a local variable to each of its neighbours — the following inconsistencies may occur:
	* Two distinct neighbours j and k receive values x and y, where x ≠ y
	* One ore more neighbours do not receive any data from i
	* Every neighbour receives a value z where z ≠ x
* Some possible causes of Byzantine failures are:
	* Total or partial breakdown of a link joining y with one of its neighbors
	* Software problems in process i
	* Hardware sync problems
		* Assume that every neighbour is connected to the same bus, and reading the same copy sent out by i, but since the clocks are not perfectly synced, they may not read the value of x at the same time — if x varies with time, then different neighbours of i may read different values of x from process i

### Software Failure
* Coding or human error
* Software design error
* Memory leaks (processes fail to free up the physical memory that has been allocated to them)
* Inadequacy of specification

### Temporal Failure
* Real-time systems require actions to be completed within a specific time frame
* When this is not met, a temporal failure occurs

### Security Failure
* Viruses and other malicious software may lead to unexpected behaviour that manifests itself as a system fault

### Human Errors
* Yup

## Fault-Tolerant Systems
* **Safety** properties specify that “something bad never happens”
* **Liveness** properties assert that “something good will eventually happen”

### Masking Tolerance
* Let P be the set of configurations for the fault-tolerant system
* Given a set of fault actions F, the fault span Q corresponds to the largest set of configurations that the system supports
* In a Masking Tolerance system, when fault F is masked, its occurrence has no impact on the application (that is, P = Q)
* Masking tolerance is important in many safety-critical applications where the failure can endanger human life or cause massive loss (i.e. an aircraft must be able to fly even if one of its engines fail)
* Masking tolerance preserve both _safety_ and _liveness_ properties of the original system
* Implementing failure masking — through information, time and physical redundancy 

### Non-Masking Tolerance
* Fault temporarily affect and violate the *safety* property (i.e P ⊂ Q)
* However, *liveness* is not compromised, and eventually normal behaviour is restored (e.g. while streaming a movie, the server crashed, but the system automatically restores service by switching to a backup proxy server)
* Stabilisation and Checkpointing represent two opposing scenarios in non-masking tolerance
	* **Checkpointing** relies on history and recovery is achieved by retrieving lost computation
	* **Stabilisation** is history-insensitive and does not care about lost computation as long as eventual recovery is guaranteed

### Fail-Safe Tolerance
* Certain faulty configurations do not affect the application in an adverse way and therefore considered harmless
* A fail-safe system relaxes the tolerance requirement by avoiding only those faulty configurations that will have catastrophic consequences (not withstanding the failure)
* e.g. at a 4-way traffic crossing, if the lights are green in both directions, then a collision is possible, however if the lights are red, at best traffic will stall but will not have any catastrophic side effect

### Graceful Degradation
* Systems that neither mask nor fully recover from the effect of failures
* But exhibits a degraded behaviour that is still considered near-normal & near-acceptable
* The notion of acceptability is highly subjective and entirely dependent on the user
* e.g. While routing a message between two points in a network, a program computes the shortest path — in the presence of failure, if this program returns another path but not the shortest, this may be acceptable