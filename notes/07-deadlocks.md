[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Deadlocks
* **Deadlock** — condition where a process cannot proceed because it needs to obtain a resource held by another process and it itself is holding a resource the other process needs
* **Communication Deadlock** — occurs when process A is trying to send a message to process B, which is trying to send a message to process C, which is trying to send a message to A
* **Resource Deadlock** — occurs when processes are trying to get exclusive access to devices, files, locks, servers or other resources

## Necessary Conditions for Deadlock
* **Mutual exclusion** — a resource can be held at most by one process
* **Hold & wait** — processes that already hold resources can wait for another resource
* **Non-preemption** — a resource, once granted, cannot be taken away from a process
* **Circular wait** — two or more processes are waiting for resources held by one of the other processes

## Deadlock Modelling
* **Resource allocation graph** — a directed graph in which both the nodes and edges are partitioned into two types:
	* **Process Nodes**
	* **Resource Nodes**
	* **Assignment Edges**
	* **Request Edges**
* In a resource allocation graph, a _cycle_ is a necessary condition for a deadlock to exist
* If there is only one copy of all resource: can result in a cycle
* If there are multiple copies of some resources: can result in a know
* A **knot** in a directed graph is a collection of nodes and edges with the property that _every vertex in the knot has outgoing edges, and all outgoing edges from vertices in the knot terminate at other vertices in the knot_ (i.e. when all the vertices are connected by outgoing edges)

### Wait for Graph
* **Wait-for-graph** — a simplified form of the resource allocation graph
* Used when all resource types have only a single unit of resource each
* Constructed by removing resource nodes and collapsing appropriate edges

## Handling Deadlocks
### Ostrich Algorithm
* Basically, ignore the deadlock problem
* Avoiding deadlocks is difficult — but detecting them is easier
* When a deadlock is detected:
	* Kill of one or more processes
	* If a system is based on atomic transactions, abort one or more transactions
		* Transactions have been designed to withstand being aborted
		* System restored to state before transaction began
		* Transaction can start a second time
		* Resource allocation in system may be different so the transaction may succeed

### Centralised Deadlock Detection
* Imitate the non-distributed algorithm through a coordinator
* Each machine maintains a resource graph for its processes and resources
* A _central coordinator_ maintains a graph for the entire system
	* Message can be sent to coordinator each time an arc is added or deleted
	* List of arc adds/deletes can be sent periodically
* A “false deadlock” can occur if messages sent to the coordinator arrive in the wrong order
	* Global time ordering must be imposed on all machines
	* Or coordinator can reliably ask each machine whether it has any release messages

### A Distributed Approach (Chandy-Misra-Haas Algorithm)
* Processes can request multiple resources at once
* Some processes wait for local resources
* Some processes wait for resources on other machines
* Algorithm invoked when a process has to wait for a resource
* A _probe_ message is generated, containing:
	* Process that just blocked
	* Process sending the message
	* Process to whom it is being sent
* When a probe message arrives, recipient checks to see if it is waiting for any processes
	* If no, ignore the message
	* If so, update the message
		* Replace second field by its own process number
		* Replace third field by the number of the process it is waiting for
		* Send messages to each process on which it is blocked
	* If message goes all the way around and comes back to the original sender, a cycle exists
		* The first part of the message is what determines the cycle
	* That means we have a deadlock!

## Recovery after Detection
* One of the following methods may be used
	* Asking for operator intervention
	* Terminating of process(es)
	* Rollback of process(es)
* Issues in recovery from deadlock
	* Minimisation of recovery cost
	* Prevention of starvation
