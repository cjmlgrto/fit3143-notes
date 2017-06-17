[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Data Parallel Architectures
* Data Parallel Architecture consists of:
	* Memory, which passes on to
		* Consists of 8 bit wide memory sells
	* Two registers, which passes on to
		* Consists of 8 bit wide registers each
	* 8 Bit ALU, which passes back to memory
		* Consists of 8 single but ALUs

## Connectivity
How do we access data in different processors?

### Nearest Neighbours 
* Mapping spatially coherent data into SIMD systems
	* Spatially correlated, like images
* Common to connect NSEW, but diagonal has also been implemented
	* Applied to massively parallel systems
	* Scalable
	* Simple to implement

### Trees and Graphs
* Problems expressed as graphs
	* e.g. database searching, model machines
* No mathematically regular structure
* Binary and quad trees common
* Communication bottlenecks going through roots of subtrees

### Pyramid
* Combination of mesh and tree
	* Supports nearest neighbour plus tree communication
	* Local communication of mesh
	* Global communication of tree
* Useful for data stored at multiple resolutions
	* Like images

### Hypercube
* 2^N processors — each of which has N links
* Fault tolerant
* Shorter pathways than mesh

## Different Data Parallel Architectures
* SIMD — program pipelined into multiple processors then back
* Systolic/Pipelined — grab data, then processes through stages of processors
* Vectorising — vector unit can operate on vector memory
* Associative and Neural — objects, databases, etc

### Principle Characteristics of Data-Parallel Systems

| Property | SIMD | Systolic | Pipeline | Neural | Associative |
| --- | --- | --- | --- | --- | --- |
| Programmability | Good | Fixed | Fixed | Good | Poor | Good |
| Availability | Good | Poor | Poor | Good | Poor | Poor |
| Scalability | Good | Fixed | Fixed | Fixed | Fixed | Good |
| Applicability | Wide | Narrow | Narrow | Wide | Narrow | Wide |