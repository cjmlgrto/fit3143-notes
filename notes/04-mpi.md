[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)


# Message-Passing Library
* **Message Passing Interface (MPI)** is a specification for developers and users of message-passing libraries.
* The goal of MPI is to provide a widely-used standard for writing message-passing programs

## Reasons for Using MPI
* Standardisation
* Portability
* Performance Opportunities
* Functionality
* Availability

## Programming Model
* MPI works best for distributed memory parallel programming models
* All parallelism is explicit: the programmer is responsible for identifying parallelism and MPI merely serves as a means to send messages between parallel threads
* The number of tasks dedicated to run a parallel program is static (i.e. programs that use MPI have a preset number of processes that can run it)

## Using MPI
* MPI is native to ANSI C
* Multiple versions of MPI available, implemented in various open-source or third-party libraries such as the NVIDIA Cuda Library

### General MPI Program Structure

```c
include "mpi.h"
include <stdio.h>

int main(int argc, char *argv[]) {
	int rank, size;

	MPI_Init(&argc, &argv);
	MPI_Comm_size(MPI_COMM_WORLD, &size);
	MPI_Comm_rank(MPI_COMM_WORLD, &rank);

	// Code to run in parallel

	MPI_Finalize();
	return 0;
}
```

* To compile, run `mpicc input.c -o output`
* To execute, run `mpirun -np 2 output`

## Communicators and Groups
* MPI uses objects called **Communicators** and **Groups** to define which collection of processes may communicate with each other
* Most MPI routines require specifying a Communicator as an argument
* For the OpenMPI library, the default is `MPI_COMM_WORLD`
* Within a communicator, each process has its own unique integer identifier assigned by the system when the process initiates — this is called its task ID or its **Rank** (note: begins at zero)
* The Rank is used by the programmer to specify the source, destination, and the task a certain process must do

## Environment Management Routines
* `MPI_Comm_rank(comm, &rank)` — determines the the rank of the calling process within the communicator
* `MPI_Abort(comm, errorcode)` — terminates all MPI processes associated with the communicator
* `MPI_Get_processor_name(&name, &resultlength)` — returns the processor name
* `MPI_Initialized(&flag)` — set the flag true or false if MPI has successfully initialised
* `MPI_Wtime()` — returns an elapsed wall clock time in seconds (double precision) on the calling processor
* `MPI_Wtick()` — returns the resolution in seconds of `MPI_Wtime()`
* `MPI_Finalize()` — terminates the MPI execution environment

## Point to Point Communication
* MPI point-to-point operations typically involve message passing between two, and only two different MPI tasks: one task to receive, the other to send
* Any type of send routine can be paired with any type of receive routine

### Buffering
- In a perfect world, every send operation would be perfectly synchronised with its matching receive — but this is rarely the case
- As such, a **system buffer** area is reserved to hold data in transit
- System buffer space is:
	- Opaque to the programmer and managed entirely by the MPI Library
	- A finite resource that can be easy to exhaust
	- Able to exist on the sending side, receiving side, or both
	- Something that may improve program performance as it enables async communication
	- Can be user-managed

### Blocking vs Non-Blocking
* A blocking send routine will only return after it is safe to modify the application buffer for re-use
* A blocking send can be synchronous which means there is a handshake occurring wit the receive task to confirm a safe send
* A blocking send can be async if a system buffer is used to hold the data for eventual delivery to the receive
* A blocking receive only returns after the data has arrived and is ready for use by the program
* Non-blocking operations simply “request” the MPI library to perform the operation when it is able
* It is unsafe to modify the application buffer until you know for a fact that the request non-blocking operation was actually performed — there are “wait” routines for this
* Non-blocking communications are primarily used to overlap computation with communication and exploit possible performance gains

### Order and Fairness
- MPI guarantees that messages will not overtake each other
- **Order** —If a sender sends two messages in close succession to the same destination, and both match the same receive operation will receive the first before the second
- **Fairness** — MPI does not guarantee fairness, it’s up to the programmer to prevent “operation starvation”

## MPI Message Passing Routines
* `MPI_Send(&buffer, count, type, dest, tag, comm)` — blocking send
* `MPI_Isend(&buffer, count, type, dest, tag, comm, request)` — non-blocking send
* `MPI_Recv(&buffer, count, type, source, tag, comm, status)` — blocking receive
* `MPI_Recv(&buffer, count, type, source, tag, comm, request)` — non-blocking receive

### Arguments
- `buffer` — variable to be sent/modified
- `count` — number of elements of a particular type to be sent
- `type`  — data type, e.g. `MPI_INT` for integers, etc.
- `dest` — rank of the process to send to
- `source` — rank of the process to receive from
- `tag` — arbitrary ID of the message
- `comm` — communicator, usually `MPI_COMM_WORLD`
- `status`  — indicates source and tag of the message
- `request` — used by the “wait” routine to determine completion of non-blocking calls

## Collective Communication
- Collective communication must involve _all_ processes in the scope of a communicator
- All processes by default, are members of `MPI_COMM_WORLD`
- It is the programmer’s responsibility that all processes within a communicator participate in any collective operations
- Collective operations are blocking
- Collective operations within subsets of processes are accomplished by first partitioning subsets into new groups then attaching the new groups to new communicators

### Types of Collective Operations
* **Sync** — processes wait until all members of the group have reached sync point
* **Data Movement** — broadcast, scatter/gather, all-to-all
* **Collective Computation** — (reductions) one member of the group collects data from other members and performs an operation (min, max, add, multiply, etc.) on the dat

## Group and Communicator Management
### Group and Communicator Objects
- Allow you to organise tasks, based on function, into task groups
- Enable collective communication
- Provide basis for user-defined topologies
- Provide safe communication

### Considerations/restrictions
* Groups/communicators are dynamic
* Processes may be in more than one group/communicator
* MPI Provides over 40 routines related to groups, communicators and  virtual topologies

## Virtual Topologies
### What are these?
- In terms of MPI, a virtual topology describes a mapping/ordering of MPI processes into a geometric “shape”
- The two main types of topologies supported by MPI are a Cartesian (grid) and Graph
- MPI topologies are virtual
- They’re built upon MPI communicators and groups
- Must be “programmed” by the developer

### Why use them?
- Convenience
- Communication efficiency
- Abstraction


