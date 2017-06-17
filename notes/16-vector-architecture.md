[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Vector Architectures
* **Vector Processors** are special-purpose computers that can process large sets (arrays) of data in parallel
* The basic idea in a vector processor is to combine two vectors and produce an output vector

## Vector vs Non-Vector Computation
* Given the task of of adding a random number to each value of an array of size N, a non- vector processor would:	* Iterate over each item, modifying each subsequent value one at a time* In contrast, a vector processor could:	* Operate on every item in the array at the exact same time

## Pipelined Computation
* Vector computation can be made much faster through pipelined computation. 
* Pipelined computation cascades operations of a program which results in a smaller gap of time between computing a result needed by other parts of a parallel program.
* The slowest computational component in a pipelined program increases the time to first result, which in turn increases the time it takes to iterate over the steps of a program.
* The repetition time is dependent on the time taken to compute the slowest component in a pipelined program.

## Interleaving
* Interleaving is a way of improving memory access performance by storing memory in parallel.* Its benefits and characteristics are:	* Interleaving can keep feeding a pipelined machine with instructions even if main memory is slow	* Interleaving slows down when subsequent accesses are from the same memory bank	* Interleaving can be rare when prefetching instructions, since it is sequential Interleaving allows simultaneous memory accesses	* Banks are usually selected using some of the low order bits of the address because sequential access will access different banks

## Vector Startup
* Vector instructions may be issued at the rate of 1 instruction per clock period — providing there is no contention they will be issued at this rate
* The first result appears after some delay, and then each word of the vector will arrive at the rate of one word per clock period
* Vectors longer than 64 words are broken into 64 word chunks

## Stride
### The Effect of Stride on Interleaving
* Most interleaving schemes simply take the bottom bits of the address and use these to select the memory bank
* This is very good for sequential address patterns (stride 1) and not too bad for random address patterns
* But for stride n, where n is the number of memory banks, the performance can be extremely bad

### Memory Layout for Stride-free access
* Consider the layout of an 8x8 matrix
* Can be placed in memory in two possible ways — by row order or column order
	* If we know that a particular program requires only row or column order, it is possible to arrange the matrix so that conflict free-access can always be guaranteed
* Skew the matrix
	* each row starts in a different memory unit
	* Possible to access the matrix by row order or column order without contention
* This requires a different address calculation

### Address Modification in Hardware
* Possible to use a different function to compute the module number
* If the address is passed to an arbitrary address computation function which emits a module number — produce stride-free access for many different strides
* There are schemes which give optimal packing and do not waste any space

### Other Typical Access Patterns
* Unfortunately, row and column access order is not the only requirement
* Other common patterns include matrix diagonals and square subarrays

#### Diagonal Access
* To access diagonal
	* Stride is equal to column stride + 1
* If M, the number of modules is equal to power of 2
	* Both column stride and column stride + 1 cannot both be efficient
	* Both cannot be relatively prime to M

## Vector Algorithms
### Gaussian Elimination
```
for i = 1 to n
	imax = index of max(abs[a[i..n,i])
	swap(a[i,i..n], a[imax,i..n])
	if a[i,i] = 0 then singular matrix
	a[i+1..n,i] = a[i+1..n,i]/A[i,j]
	for k = i+1 to n
		a[k,i+1..n] = a[k,i+1..n]-a[k,i]*a[i,i+1..n]
```

* The algorithm as expressed accesses both rows and columns
* The majority of the vector operations have either two vector operands, or a scalar and vector operand, and they produce a vector result
* The `max` operation on a vector returns the index of the maximum element, not the value
* The length of the vector items accessed decreases by 1 for each successive iteration
* Row and Column access should be equally efficient
* Vector pipeline should handle a scalar on one input
* `min`, `max` and `sum` operators required which accept one or more vectors and return scalar
* Vectors may start large, but get smaller — this may affect code generation due to vector startup cost

### Sparse Matrices
* In many engineering computations, there may be very large matrices
	* May also be sparsely occupied
	* Contain many zero elements
* Problems with normal method of storing the matrix:
	* Occupy far too much memory
	* Very slow to process
* Many packages solve the problem by using a software data representation
	* The trick is not to store the elements which have zero values
* At least two sorts of access mechanism
	* Random
	* Row/Column sequential
* Matrix multiplication then consists of moving down the row/column pointers and multiplying elements if they have the same index values
* Hard to implement this type of data representation