[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Mutual Exclusion
* A **critical section** is a section in a program that accesses shared resources
* A process enters a critical section before accessing the shared resource to ensure that no other process will use the shared resource at the same time
* Critical sections are protected using flags & monitors in single-processor systems
* These mechanisms cannot be used in distributed systems

## Centralised Algorithm
* Simulate mutual exclusion in a single-processor system
* Once process is elected as the “coordinator”
* When a processes wants to enter a critical section, it sends a request stating which critical section it wants to enter
* If no other processes is currently in that critical section, the coordinator grants request
* If a different processes is in the critical section, it is queued
* When that different process finishes, the process sends a message to the next item on the queue saying that it’s ready to use

### Pros
* Fair since first-come-first-serve
* Easy to implement
* Requires only three messages: request, grant and release (per critical section)

### Cons
* If the coordinator crashes, entire system can go down
* Processes cannot distinguish a dead coordinator from “permission denied”
* A single coordinator may become a performance bottleneck

## Distributed Algorithm
* Requires ordering of all events in the system — Lamport’s algorithm is used (“Happens-before” relation)
* When a process wants to enter a critical section, the process sends a request message to all other processes:
	* Name of critical section
	* Process number
	* Current time
* The other processes receive the request message
	* If the process is not in the requested critical section and also has not sent a request message for the same critical section, return OK to the requesting process
	* If the process is in the critical section, it does not return any response and puts the request to the end of a queue
	* If the process has also sent out a request for the same section, it compare the timestamp of the sent request message and the received message
* Time stamps are compared
	* If the time stamp of the received message is smaller than the one of the sent message, return OK
	* If the timestamp of the received message is larger than the one of the sent message, put into queue
* The requesting process waits until all processes return OK messages
	* When the requesting process receives all OK messages, the process enters the critical section
	* When a process exits from a critical section, it returns OK to all requests in its queue corresponding to the critical section and dequeues those requests
* Processes enter a critical section in timestamp order using this algorithm

### Pros
* No deadlock or starvation happens

### Cons
* 2(n-1) messages are required to enter a critical section, where n is the number of processes
* If one of the processes fails, the process does not respond to the request — it means no process can enter the critical section
* The coordinator is the bottleneck in a centralised system — since call processes send requests to all other processes in this distributed algorithm, all processes are bottlenecks

## Token Ring Algorithm
* Arrange the processes in a virtual ring (even if they are not topologically arranged as such)
* Process zero receives a token
* The token is passed to the process with the next sequence number (i.e. process 1 → 2 → 3 → …)
* A process with the token can enter the critical section, then passes it when it is done
* The token is simply passed if the process does not need to enter the critical section

### Pros
* Processes don’t suffer from starvation — before entering a critical section, a process waits at most for the duration that all other processes enter and exit the critical section

### Disadvantage
* If a token is lost for some reason, another token must be generated
* Detecting the lost token is difficult since there is no upper bound in the time a token takes to rotate the ring
* The ring must be reconstructed when a process crashes

## Mutual Exclusion Algorithms, Compared
### Messages Exchanged
* Centralised: 3
* Distributed: 2(n-1)
* Ring: 2

### Reliability (Problems that may occur)
* Centralised: coordinator crashes
* Distributed: any process crashes
* Ring: lost token, process crashes

## Group Mutual Exclusion Problem
* Processes choose “session” when they want to enter the critical section
* Processes are allowed to be in the critical section simultaneously when they request the same session

### Applications of GME
* In some applications such as Computer Supported Cooperative Work
* Wireless application
* An efficient GME solution could help improve the quality of services of an Internet server
	* The GME protocol can be used to group different requests for the same service, and thereby, reduce the memory swapping