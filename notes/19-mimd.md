[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# MIMD Architectures
* Distributed Memory MIMD
	* Replicate processor/memory pairs
	* Connect them via an interconnection network
* Shared Memory MIMD
	* Replicate the processors
	* Replicate the memories
	* Connect them via an interconnection network

## Distributed Memory Machine
* Access to local memory module is faster than remote
* Hardware remote access via
	* Load/Store primitive
	* Message passing layer
* Cache memory for local memory traffic
* Message
	* Memory to memory
	* Cache to cache

### Advantages
* Local memory traffic less contention than in shared memory
* Highly scalable
* Don’t need sophisticated synchronisation features like monitors, semaphores
* Message passing serves dual purpose
	* To send data
	* Provide synchronisation

### Disadvantages
* Load balancing
* Message passing can lead to sync failures, including deadlock
	* Blocking Send → Blocking Receive
	* Blocking Receive → Blocking Send
* Intensive data copying of whole structures
* Small message overheads are high

## Shared Memory Architecture
* All processors have equal access to shared memory
* Local Caches reduce
	* Memory traffic
	* Network traffic
	* Memory access time

### Advantages
* No need to partition code or data
* Occurs on the fly
* No need to move data explicitly
* Don’t need new programming languages or compilers

### Disadvantages
* Synchronisation is difficult
* Lack of scalability
	* IPC becomes bottleneck
* Scalability can be addressed by
	* High throughput, low latency network
	* Cache Memories
		* Causes coherence problem
	* Distributed Shared Memory architecture

## Distributed Shared Memory
* Non-Uniform Memory Access (NUMA)
* Cache Coherent Non-Uniform Memory Access (CC-NUMA)
* Cache-Only Memory Access (COMA)

## Problems of Scalable Computers
* Tolerate and hide the latency of remote loads
	* Worse if output of one computation relies on another to complete
* Tolerate and hide idling due to synchronisation among processors

### Tolerating Latency
* Cache memory
	* Simply lowers the cost of remote access
	* Introduces cache coherence problem
* Prefetching
	* Already present, so cost is low
	* Increases network load
* Threads + fast context switching
	* Accept that it will take a long time and cover the overhead
* These solutions don’t solve sync issues
	* Latency-tolerant algorithms can however

## Design Issues of Scalable MIMD
* Processor Design
	* Pipelining, parallel instruction issue
	* Atomic data access, pre-fetching, cache memory, message passing, etc.
* Interconnection Network Design
	* Scalable, high bandwidth, low latency
* Memory Design
	* Shared memory design
	* Cache coherence
* I/O Subsystem
	* Parallel IO