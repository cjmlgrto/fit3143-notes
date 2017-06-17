[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Parallel Computing Alternatives
## Parallel Virtual Machine
* Based on a dynamic computing model
	* Nodes can be added/deleted on the fly
	* Parallel task spawned/killed during the computation
	* Provides fault tolerance and adaptability
* Not as rich in message passing features as MPI
* However being a virtual machines provides features for creating dynamic parallel programs

### PVM Architectures
* Each host participating in the VM executes the `pvmd` daemon
* The `pvdmd`s of all the hosts combine to form the VM
* Applications with PVM primitives can contact their local `pvmd` to interact and/or execute actions across the VM

### Building Fault-Tolerant Programs
* PVM provides a monitoring and notification system
	* Any/all tasks in an app can be asked to notify of specific events
	* These include exiting of a task in an app
	* Failure or deletion of a node in the cluster could also generate a notification
* Tasks can watch their neighbouring tasks in a logical ring
* Monitoring is actually done by PVM, the logical ring reduces the number of messages

### Adaptive Programs
* Cluster programs can be made to adapt to cluster
* A parallel application may dynamically adapt to the size of the VM
	* Adding and releasing nodes based on the computational needs of the app

## LINDA Model
* Concurrent programming model
* **Tuple-space** — an abstraction of distributed shared memory — acts as the primary way cooperating processes communicate
	* Tuple-spaces are associative
	* Destructive and nondestructive reads and different coherency semantics are possible
* Apps use the LINDA model by embedding explicitly, within cooperating sequential programs, constructs that manipulate (insert/retrieve tuples) the tuple-space
* i.e. Apps can communicate with one another in parallel by modifying the tuple-space like it’s shared memory

### LINDA Applications
* From an app point of view, Linda is a set of programming language extensions for facilitating parallel programming
* Provides a shared-memory-abstraction for process communication without requiring the underlying hardware to physically share memory

### LINDA Systems
* Usually refers to a specific implementation of software that supports the Linda programming model
* System software is provided that establishes and maintains tuple-spaces
* And is used in conjunction with libraries that appropriately interpret and execute Linda primitives
* Depending on the environment (shared-memory multiprocessors, message-passing parallel computers, networks of workstations, etc.), the tuple-space mechanism is implemented using different techniques with varying degrees of efficiency
* For instance, a proactive approach would be to seize computational tasks from a well-known location based on availability and suitability

## OpenMP
* **Open** specifications for **Multi-Processing** via collaborative work between interested parties from the hardware and software industry, government and academia
* An API to explicitly direct *multi-threaded, shared memory parallelism*
* Comprises 3 primary API components:
	* Compiler Directives
	* Runtime Library Routines
	* Environment Variables
* Portable: The API is specified for C/C++ and Fortran

### OpenMP Limits
* Meant for distributed memory parallel systems (by itself)
* Necessarily implemented identically by all vendors
* Guaranteed to make the most efficient use of shared memory
* Required to check for data dependencies, data conflicts, race conditions or deadlocks
* Required to check for code sequences that cause a program to be classified as non-conforming
* Meant to cover compiler-generated automatic parallelisation and directives to the compiler to assist such parallelisation
* Designed to guarantee that input or output to the same file is synchronous when executed in parallel — the programmer is responsible for input/output

### OpenMP Goals
* Standardisation
* Lean and Mean
* Ease of Use
* Portability

### Programming Model
* Shared Memory, Thread-Based Parallelism
	* OpenMP is based upon the existence of multiple threads in the shared memory programming paradigm
	* A shared memory process consists of multiple threads
* Explicit parallelism:
	* OpenMP is an explicit (not automatic) programming model
	* Offering the programmer full control over parallelization

### Fork-Join Model
* OpenMP uses the fork-join model of parallel execution
* All OpenMP programs begin as a single process: the _master thread_
	* The master thread executes sequentially until the first parallel region construct is encountered
* `FORK` — the master thread then creates a _team_ of parallel threads
* The statements in the program that are enclosed parallel region construct are then executed in parallel among the various team threads
* `JOIN` — when the team threads complete the statements in the parallel region construct, they synchronise and terminate, leaving only the master thread

### OpenMP Features
* Compiler Directive based
	* Most OpenMP parallelism is specified through the use of compiler directives which are embedded in C/C++ or Fortran source code
* Nested Parallelism support
	* The API provides for the placement of parallel constructs inside of other parallel constructs
	* Implementations may or may not support this feature
* Dynamic Threads
	* The API provides for dynamically altering the number of threads which may be used to execute different parallel regions
* Implementations may or may not support this feature

### I/O and Memory
* OpenMP specifies nothing about parallel I/O
* This is particularly important if multiple threads attempt to read/write from the same file
* If every thread conducts I/O to a different file, the issues are not as significant
* It is entirely up to the programmer to ensure that I/O is conducted correctly within the context of a multi-threaded program
* Memory Model: `FLUSH` often?
	* OpenMP provides a “relaxed consistency” and “temporary” view of thread memory
	* i.e. Threads can “cache” their data and are not required to maintain exact consistency with real memory all of the time
	* When it is critical that all threads view a shared variable identically, the programmer is responsible for ensuring that the variable is `FLUSH`ed by all threads as needed

## GPGPU
* Traditional CPU design is suited to sequential processing
* GPU is used to execute application non-graphic processes in parallel
* Operates with the addition of programmable stages and higher-precision arithmetic to the rendering pipelines
* Limitations
	* Only certain types of application can benefit from this approach
	* Requires the use of special libraries, however developed software is available in some cases

### Stream Processing
* Computing model based on the SIMD paradigm
* Some applications can be easily parallelised within the limits of this style of processing
* Applications can use multiple floating point units on GPU
* The following functions, which are usually required in SIMD, are assumed to be managed by the units in stream processing:
	* Allocation
	* Synchronisation
	* Communication
* It simplifies parallel software and hardware by restricting the parallel computation that can be performed
* A data set constitutes a (data) stream
* A kernel function is a series of operation
	* Commonly defined as a series of nested loops (with no data spec)
* The kernel function is applied to each element in the stream
	* **Uniform streaming** — where the same kernel function is applied to al elements in the stream, is common
* Kernel functions are usually pipelined, and local on-chip memory is re-used to minimise external memory bandwidth
* Since the kernel and stream abstractions expose data dependencies, compiler tools can fully automate and optimise on-chip management tasks

### Data Dependencies and Parallelism
* Two important stream data characteristics (as required by a kernel)
	* Independent
	* Local
* Kernel operations define the basic data unit, both as input and output
	* This allows the hardware to better allocate resource and schedule global I/O
* Definition of the data unit is usually explicit in the kernel, which is expected to have well-defined (e.g. structures) inputs and outputs
	* Furthermore, GPU output values are fixed
* Well-defined and independent compute blocks allow scheduling of bulk I/O operations — leveraging the underlying hardware resources of cache and memory bus
* Kernel locality — values associated within a single kernel invocation are made local and use the fast registers

### Stream Programming Languages
* Generally, the following type of applications will benefit from stream computing:
	* Compute Intensive — where the number of arithmetic operations is larger for each I/O or global memory access operation
	* Data parallelism exists, where the same kernel function can be applied to records of an input stream and a number of records can be processed simultaneously without waiting for results from previous records
	* Data locality, where data is produced once, read once or twice later in the application, and never read again. Intermediate streams passed between kernels as well as intermediate data within kernel functions can capture this locality directly using the stream processing programming model

### OpenCL
* Open specifications, designed for number crunching
* OpenCL programming interface provides a good framework to utilise the underlying (parallel) hardware
* Hardware heterogeneity, CPU and GPU support

#### Applications
* Image, Video and Audio processing
* Gaming, Scientific calculations, Simulations (combined with OpenGL for visual stuff)
* Financial Modelling
* Computationally-intensive data-parallel applications

### GPGPU Programming Techniques
#### Map
* The map operation simply applies the kernel (the user-specified function) to every element in the stream
* A simple exam is multiplying each value in the stream by a constant (e.g. increasing the brightness of an image)
* The map operation is simple to implement on the GPU

#### Reduce
* Some computations require calculating a smaller stream (possible, a stream of only 1 element) from a larger stream
* Reduction:
	* The results from the previous step are used as the input for the current step
	* The range over which the operation is applied is reduced until only one stream element remains

#### Scatter
* Equivalent to `a[i] = x` in C
* The scatter operation is best defined on the vertex processor
	* The vertex processor is able to adjust the position of the vertex, which allows the programmer to control where information is deposited on the grid
	* The fragment processor cannot perform a direct scatter operation because the location of each fragment on the grid is fixed at the time of the fragment’s creation and cannot be altered by the programmer
* A scatter implementation would first emit both an output value and an output address followed by gather operation
* Gather performs address check by comparisons

#### Gather
* Opposite of scatter
* Gather is equivalent to `x = a[i]` in C
* Analogous to an image processing GPU, where the fragment processor is able to read textures in a random access fashion
* GPGPU can gather information from any grid cell, or multiple grid cells, as need be

#### Stream Filtering and Sort
* Stream filtering is essentially a non-uniform reduction
* Filtering involves removing items from the stream based on some criteria
* The sort operation transforms an unordered set of elements into an ordered set of elements
	* The most common implementation on GPUs is sorting networks

#### Searching
* The search operation allows the programmer to find a particular element within the stream or possibly find neighbours of a specified element
* The GPU is used to run multiple searches in parallel

