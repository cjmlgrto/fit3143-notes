[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Parallel Computing
* **Parallel computing** is a form of computation in which many instructions are carried out simultaneously
* Operates on the principle that large problems can often be divided into smaller ones which are then solved concurrently
* There are several different levels of parallel computing: bit-level, instruction-level, data-level and task-level parallelism

## Flynn’s Taxonomy
### SISD Processor (Single Instruction, Single Data)
* Serial, non-parallel computer
* Single instruction: only one instruction stream is being acted on the CPU during any one clock cycle
* Single data: only one data stream is being used as input during any one click cycle
* Deterministic execution
* e.g. most PCs, single CPU workstations and mainframes

### SIMD Processor (Single Instruction, Multiple Data)
* A type of parallel computer
* Multiple data: each processing unit can operate on a different data element
* This type of machine typically has an *instruction dispatcher* — a very high-bandwidth internal network, and a very large array of small-capacity instruction units
* Best suited for specialised problems characterised by a high-degree of regularity, such as image processing
* Synchronous and deterministic execution
* Two varieties: Processors Arrays and Vector Pipelines

### MISD Processor (Multiple Instruction, Single Data)
* A single data stream is fed into multiple processing units
* Each processing unit operates on data independently via independent instruction streams
* Few actual examples of this type of parallel computer have ever existed
* Some conceivable uses might be:
	* Multiple frequency filters operating on a single signal stream
	* Multiple cryptography algorithms attempting to crack a single coded message

### MIMD Processor (Multiple Instruction, Multiple Data)
* Most common type of parallel computer — most modern computers fall into this category
* Multiple instruction: every processor may be executing a different instruction stream
* Multiple data: every processor may be working with a different data stream
* Execution can be sync or async , deterministic or not
* Examples: most current supercomputers, networked parallel computer “grids” etc.

## Parallel Computer Memory Architectures
* **Shared Memory** — single (physical or virtual) memory address space
* **Distributed Memory** — each process has its own memory address space
* **Hybrid** — a combination of both

## Performance
### General Speedup Formula
* Speedup = (Sequential execution time) ÷ (Parallel execution time)
* Execution time components:
	* Inherently sequential computations
	* Potentially parallel computations
	* Communication operations
	* i.e. Speedup = (Serial + some parallel) ÷ (Some serial + parallel + communication)

### Amdahl’s Law
* Speedup = 1 ÷ (1 - fraction of the program that is parallelisable)
* An interesting outcome:
	* If the sequential portion of a program is 10% of the runtime, we can get no more than a 10x speed-up, regardless of how many processors are added
	* This puts an upper limit on the usefulness of adding more parallel execution units
* Efficiency = (Speedup) ÷ (Number of processors)

#### Examples
* 95% of a programs execution time occurs inside a loop that can be executed in parallel — what is the maximum speedup we should expect from a parallel version of the program executing on 8 CPUs?
	* 1 ÷ [5% + (1 - 5%)/8] ~= 5.9
* 20% of a program’s execution time is spent within inherently sequential code — what is the limit to the speedup achievable by a parallel version of the program?
	* limit as p approaches ∞ of
		* 1 ÷ [0.2 + (1-0.2)/p] = 1/0.2 = 5

#### Limitations
* Ignores communication operations, and as such overestimates speedup
* Assumes the fraction of the code that is sequentially never changes

#### Amdahl Effect
* Typically, communication operations have lower complexity than overall speedup time
* As speedup time increases, parallel time dominates communication time
* As n increases, speedup increases
* As n increases, impact of fraction of code that is serial that could be parallel decreases

### Gustafson’s Law
* Any sufficiently large problem can be efficiently parallelised
* Closely related to Amdahl’s Law, which gives a limit to the degree to which a program can be sped-up due to parallelisation
	* S(P) = P - a*(P-1)
	* Where P is the number of processors
	* S is the speedup
	* a is the non-parallelisable part of the process
* Gustafson’s law addresses the shortcomings of Amdahl’s Law, which cannot scale to match availability of computing power as machine size increases

## Parallel Programming
* Usually, converting a serial problem into parallel involves breaking down a for loop into tasks that can be accomplished concurrently
* The extra task is to determine which processor is currently executing the function, to properly allocate tasks and sent it back to a “master”

### 2D Array Calculation
#### Serial Loop 
```
do j = 1, n
	do i = 1, m
		a(i, j) = calculate(i, j)
	end do
end do
```

#### Parallel Kernel
```
do j = mystart, myend
	do i = 1, m
		a(i, j) = calculate(i, j)
	end do
end do
```


### Pi Calculation (Monte Carlo Method)
#### Serial Loop
```
npoints = 1000
circle_count = 0
do j = 1, npoints
	generate 2 rand numbers between 0 and 1
	xcoordinate = rand1; ycoordinate = rand2;
	if (xcoordinate, ycoordinate) inside circle
		circle_count += 1
end do
PI = 4.0*circle_count/npoints
```

#### Parallel Kernel
```
npoints = 1000
circle_count = 0
p = number of tasks
num = npoints/p
find out if I am MASTER or WORKER
do j = 1, num
	...
end do
if I am MASTER
	receive from WORKERS their circle_counts
	compute PI
else if WORKER
	send to master circle_count
endif
```

### 1D Wave Equation
* Partition wave equally and distribute portions to each task
* Communication need only occur between neighbouring tasks/partitions
* Use the 1D wave equation to calculate amplitude

```
find out number of task and task identities

# identify left and right neighbors
left_neighbor - rank -1
right_neighbor = rank + 1
if rank = first then left_neighbor = last
if rank = last then right_neighbor = first

find out if I am MASTER or WORKER
if MASTER
	initialise array
	send each WORKER starting info and subarray
else
	receive starting info and subarray from MASTER

# update values for each point along string
# in this example, master participates in calculations
do t = 1, nsteps
	send left endpoint to left neighbour
	receive left endoint from right neighbor
	& vice versa
	do i = 1, npoints
		(wave equation)

# collect results and write to file
if MASTER
	receive results from each WORKER
	write results to file
else
	send results to MASTER
```