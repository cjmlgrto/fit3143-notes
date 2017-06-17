[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Distributed Memory MIMD Architectures
* Process level instead of threads
	* Large grain threads
* Messages passed between processes
	* Synchronisation
	* Data movement
* Important questions
	* How to partition the parallel program
	* How to map processes onto processors to minimise IPC

## Design Space
* Reduce time and frequency of thread switches
	* Higher in MIMD machines than in multi-threaded architectures
	* Frequency of computation
	* Processor time spent on communication
* Design focus
	* Organisation of communications subsystem — i.e. network
	* Hardware support for message passing
	* Often dependent on algorithms & software
* Communication over links or channels
* Communication graph mapped onto the underlying network
* Nodes consist of
	* Computation processor + private memory
	* Communication processor
	* Router (or switch unit)

## Multiprocessor Classification
* How the interconnection network is organised
	* Topology
		* Switching technique
		* Routing protocols
* How the three components of the nodes are composed and integrated
	* First generation — no communications processor
	* Second generation — separate comms switching units
	* Third generation — separate processors connected to one switching network
* The message passing computational model supported by the multicomputer
	* Library calls (MPI, PVM, etc)
	* CSP support
	* Push, pull or active message
* The optimal grain of computation
	* Coarse grain — traditional languages
	* Medium grain — CSP
	* Fine grain — Dataflow

## Network Performance
* Critical to performance of MIMD machines
* Bandwidth and latency are both critical parameters

## Interconnection Networks
* Network size (N)
	* Number of nodes in the network
* Node degree (d)
	* Number of I/O links per node
* Network diameter (D)
	* Maximum number of hops to get from any node to another
* Bisection width (B)
	* Related to wiring density & overall bandwidth
	* Minimum number of links broken to break network in 2 equal halves
* Arc connectivity
	* Minimum number of arcs removed to break into two disconnected networks
	* Measure of fault tolerance and contention
* Cost
	* Number of communication links in the network

### Interconnection Topologies
* 1D
	* Linear Array
* 2D
	* Ring
	* Star
	* Tree
	* 2D Mesh
	* Torus
* 3D
	* Cube
	* Connected cycles
	* Completely connected
* Hypercubes
	* 2D
	* 3D
	* ND

### Multistage Interconnection Networks
* Are hybrid of cross-bar and buses
* They can provide contention-free access (excluding multiple requests on the same module) although contention does occur at certain traffic levels
* Unlike buses they are scalable
* There are many different interconnection networks and topologies of the same network
* The simplest is the shuffle exchange network
* In non-redundant networks, the depth of the network is O(log(n))

### Summary of Static Network Parameters
| Topology | Node Degree | Diameter | Bisection Width | Arc Connectivity | Cost |
|
| Linear Array | 1 or 2 | N - 1 | 1 | 1 | N - 1 |
| Ring | 2 | N/2 | 2 | 2 | N |
| Star | 1 or N-1 | 2 | 1 | 1 | N - 1 |
| Binary Tree | 1, 2 or 3 | 2log((N+1)/2) | 1 | 1 | N - 1 |
| 2D Mesh | 2, 3 or 4 | 2(N^0.5-1) | N^0.5 | 4 | 2(N-N^0.5) |
| Torus | 4 | N^0.5 | 2N^0.5 | 4 | 2N
| 3D cube | 3, 4, 5 or 6 | 3(N^0.33-1) | N^0.66 | 3 | 2(N-N^0.66) |
| Hypercube | logN | logN | N/2 | logN | (NlogN)/2 |
| Completely connected | N - 1 | 1 | N^2/4 | N - 1 | N(N-1)/2 |

## Switching Techniques
* Network structure defines pathways for the data
* **Switching techniques** defines how to move data through the network
	* Remove message from input buffer
	* Place in the output buffer

### Packet Switching
* Data divided into packets
	* Header, body, tail
* Whole packet stored at each node before being set on
* Latency = (Packet length * distance)/Channel bandwidth

### Circuit Switching
* Path between source and destination built up
	* Circuit establishment phase
		* Send probe through network — opens the path
	* Transmission Phase
		* Assumes routing path already set up
	* Termination Phase
		* Circuit is ripped up
* Latency = (Probe Length * Distance)/Channel Bandwidth + (Message Length / Channel Bandwidth)
* Advantages
	* Don’t need to build packets
	* No buffering required
	* Routing done once per message

### Virtual Cut-through
* Combination of packet switching and circuit switching
* Packet broken into Flow Control Digits (_FLITS_)
* If channels are free, FLITS are forwarded in circuit switched mode
* If channels are not free, message broken and buffered
* Latency = (Length of header FLIT * distance) / Channel Bandwidth + (Message Length/Channel bandwidth)
* As long as required channels are free message forwarded between nodes FLIT by FLIT
* If channel is busy, FLITS are buffered at intermediate nodes
* If buffers are large enough, entire message is buffered and behaves like packet switching
* If buffers are not large enough, message will be buffered across several nodes holding the links

### Wormhole Routing
* Special case of virtual cut-through
* Buffer size is size of a FLIT
* Packet replication
	* Used to implement broadcasts
	* Sending packets out on multiple output channels concurrently
	* Cannot be done with circuit switching

### Virtual Channels
* Wormhole routing better than circuit switching when switch contention is high
	* Circuit switch blocks path for whole message
	* Wormhole routing breaks into FLITS but if path is blocked then intermediate channels are also blocked
	* Note: contention may be for intermediate nodes
* Virtual Channel allows multiple independent messages to share the same physical channel
	* More than one FLIT buffer per channel
	* Number of buffers > number of channels
	* Channel is multiplexed (shared)

#### Advantages
* Increase network throughput by reducing physical channel idle time
* Can be used to avoid deadlocks
* Allow mapping of logical topology onto physical network
* Can reserver some logical channels for critical functions
	* e.g. debugging, monitoring

#### Deadlock avoidance
* Pre-emption of messages by rerouting
	* Solve problem by using alternate path for one message
* Pre-emption of messages by discarding
	* Retransmit on alternate path
* Application of virtual channels
	* Split physical channels into multiple virtual ones
		* Breaking cycles in the dependency graph
	* Sufficient resources to route all messages

## Cyclic Dependency Graph
* Channel dependency graph
* Constructed from the interconnection network and routing algorithm
* If cycles are present then message deadlock may occur

### Breaking Cycles
* Cycles can be removed by having more than one logical channel in graph
* Build a new channel dependency graph with no cycles

## Livelock
* Messages endlessly propagate in the network without reaching their destinations
* Can occur in flow control policies when collision on channels and buffers are avoided by misrouting messages to find an alternative path
* Pack switching and virtual cut through — very sensitive to livelock
* Circuit switching and wormhole routing — usually livelock free

## Routing Protocols
* Deterministic — route depends only source/destination pair
	* Source Routing — source path selects route
		* Minimal
	* Distributed Routing — route computed by intermediate nodes
		* Minimal — shortest path
* Adaptive — route determined at runtime
	* Distributed Routing
		* Minimal
		* Non-Minimal — not shortest path

### Deterministic Routing
* Street Sign Routing
	* Source routing class
	* Message header contain complete path information
	* Routing information in form of direction changes
* Dimension-ordered routing
	* Message travels along certain dimension until they reach that dimension
	* Then proceed on next dimension
* Table lookup routing
	* Each node contains a routing table
		* Contains identifier of neighbouring node that message should be routed
		* One entry per destination address
		* Tables are precomputed and can be large

### Categorisation of Adaptive Routing Protocols
* Minimal
	* Profitable — move closer to goal
		* Progressive — can only move forwards
			* Complete (Idle) — can explore all channels
			* Partial — only use selected channels
		* Backtracking — can backtrack through path
			* Complete (EPB)
			* Partial
* Non-minimal
	* Misrouting — use profitable and non profitable channels
		* Progressive
			* Complete
			* Partial (Turn model)
		* Backtracking
			* Complete (EMB)
			* Partial

#### Profitable vs Misrouting
* Profitable
	* Can only move closer to goal
	* Represents a conservative view
	* Minimum length path
	* Free from livelock
	* Easier to prove deadlock free
* Misrouting
	* Can take other choices
	* Optimistic view
	* Selecting a non-profitable channel in belief that it will lead to profitable ones
	* More fault-tolerant (don’t have to use particular channel)

#### Progressive vs Backtracking
* Progressive
	* Cannot backtrack
	* May deadlock
* Backtracking
	* Can systematically explore all paths
	* Deadlock free
	* More complex (need to know where have been before)
	* If no profitable channel available
		* Wait for channel
		* Tries non-profitable free channel
		* Steps back and starts again

#### Complete or Partial
* Partial choice of channels allows deadlock free route to be computed
* Add nodes used the same ordering of channels and explore them in this order

## Machine Design Choices
* Build MIMD systems from processors that support
	* Fine-grain message passing (e.g. J-Machine)
	* Medium grain message passing (e.g. Transputer — built around Communicating Sequential Processes — CSP)
	* Coarse grain message passing (e.g. COTS — networks have higher latency, processors don’t understand message passing)

### Architecture of a Message-Driven Processor
* Fine Grain concurrent processing
	* Reduce overhead and latency of receiving a message
		* Special hardware
	* Reduce context switch time (i.e. fast context switch)
		* Multiple registers
		* Hardware support for sync
			* Tag associated with word — initially empty
			* If empty word read, then thread is blocked
			* When word is written, blocked thread restored
	* Hardware support for object-oriented programming
		* Supports `call` and `send` to object

### Medium Grained Computing
* Message passing is synchronous
	* Neither sender nor receiver can continue without each other
	* First process reaching a channel operation must be suspended until partner arrives
	* Register uses
		* A = length of message
		* B = channel identity
		* C = pointer to message buffer in memory
* Mapping processes to processors
	* Processes need to be statically mapped to hardware
		* Links
		* Processors
* If there aren’t enough physical links, virtual links are used

