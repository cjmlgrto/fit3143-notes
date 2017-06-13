[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Synchronisation
- A distributed system consists of a collection of distinct processes that are spatially separated and run concurrently
- In systems with multiple concurrent processes, it is economical to share system resources
- Sharing may be cooperative or competitive
- Both competitive and cooperative sharing require adherence to certain rules of behaviour that guarantee correct interaction occurs
- The rules of enforcing correct interaction are implemented in the form of sync mechanisms

## Issues Implementing Synchronisation in Distributed Systems
- In single CPU systems, sync problems such as mutual exclusion can be solved using semaphores (flags) and monitors — which rely on shared memory
- Flags and monitors cannot be used in distributed systems since different processors are not expected to share memory
- Even simple matters such as determining one event happening before another requires careful thought
- In distributed systems, it is usually not possible or desirable to collect all information about the system in once place
- And sync among processes is difficult due to the following features of distributed systems:
	- Relevant information is scattered amongst multiple processors
	- Processes make decisions based only on local information
	- A single point of failure in the system should be avoided
	- No common clock or other precise global time source exists

## Time and Distribution: Why?
- **Externally** — needed to measure time accurately
- **Internally** — many distributed algorithms use time

## Clock Synchronisation
- Time is unambiguous in a centralised system
- A process can just make a system call to know the time
- In a distributed system, if processes A and B are on different machine, B’s time may not be the same as A’s time

### Imperfect Clocks
- Human-made clocks are imperfect
	- They run slower or faster than “real” physical time
	- This lag or overtake is known as _drift_
	- A drift of 1% means the lock adds or loses a second every 100 seconds

### Cristian’s Algorithm
- Synchronises all other machines to the clock of one machine, the _time server_
- Every machine requests the current time to the time server
- The time server responds to the request as soon as possible
- The requesting machine sets its clock to: `server_time + (post_request_time - pre-_request_time - server_elapsed)/2`

### The Berkeley Algorithm
- A _master_ processes periodically polls the other _slave_ processes
- Works as follows:
	1. A master is chosen with a ring-based election algorithm
	2. The master polls the slaves who reply with their time (similar to Cristian’s algorithm)
	3. The master observes the round-trip time of the messages and estimates the time of each slave and its own
	4. The master then averages the clock times, ignoring any values it receives far outside the values of the others
	5. Instead of sending the updated current time back to the other processes, the master then sends out the amount (±) that each slave must adjust its clock
	6. This avoid further uncertainty due to round-trip-time at the slave processes
	7. Everyone adjusts their time
- With this method, the average cancels out individual clock’s tendencies to drift
- Computer systems avoid rewinding their clock when they receive a negative clock alteration — doing so would break the property of monotonic time, which is a fundamental assumption
- A simple solution is to half the clock for the duration (but this simplistic solution can cause problems)
- Another solution is to slow the clock

### Averaging Algorithm
- Unlike the algorithms above, the averaging algorithm is **decentralised** (i.e. no master)
- This algorithm divides time into resynchronisation intervals with a fixed length R
- Every machine broadcasts the current time at the beginning of each interval according to its clock
- A machine collects all other broadcasts for a certain interval and sets the local clock by the average of their arrival times

### Lamport’s Synchronisation Algorithm

#### Logical and Physical Clocks
- Clock sync need not be absolute
- If to processes do not interact, their clock need not be synchronised
- What matters is they agree on the order in which events occur
- For many purposes, it is sufficient that all interacting machines agree on the same time
- It is not essential that time is the real time
- Clocks that agree on time and within a certain time limit are “physical clocks”
- Computers that follow their own time are termed to be following a “logical clock”

#### “Happens-Before”
- “Happens-before” relation:
	- A → B is read “A happens before B” — this means all processes agree that A occurs before B
- The happens-before relation can be observed directly in two situations:
	- If A and B are events in the same process and A occurs before B, then A → B
	- If A is the event of a message being sent by one process, and B is the event of the message being received by another processes, then A → B
- “Happens-before” is transitive
- If two events X and Y happen in different processes that do not exchange messages (not even indirectly) then neither X → Y nor Y → X is true — they are instead “concurrent”
- A time value C(A) is assigned to all processes for every event A:
	- If A → B, then C(A) < C(B)
	- The clock time must always go forward, never backward

#### Shortcomings of Lamport’s Clock
- It observes if A → B then C(A) < C(B) but C(A) < C(B) does not imply A → B always
- The root of the problem is that clocks advance independently or via messages, but there is no history as to where this advance comes from

### Vector Clock
- Each process maintains a vector clock V of side N, where N is the number of processes
- The component Vi[j] contains the process Pi’s knowledge about Pj’s clock
- Initially, we have Vi[j] := 0 for i, j << {1, 2, …, N}
- Clocks are advanced as follows:
	1. Before Pi timestamps an event, it executes Vi[i] Vi[i] + 1
	2. Whenever a message m is sent from Pi to Pj:
		- Process Pi executes Vi[i] := Vi[i] + 1 and sends Vi with m
		- Process Pj receive Vi with m and merges the vector clocks Vi and Vj as follows:
		- Vj[k] := { max(Vj[k], Vi[k])+1 if j=k, Max(Vj[k], Vi[k] otherwise }
- This last part ensures that everything subsequently happens at Pj is now causally related to everything that previously happened at Pi