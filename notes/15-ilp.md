[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Instruction-Level Parallelism
- ILP stands for Instruction-Level Parallelism
- ILP of a program is a measure of the number of instructions a program can execute simultaneously
- ILP involves executing multiple independent instructions in parallel
- Increases the performance of processors by making the most efficient use of processors

## Pipelining
- Pipelining involves sequentially passing processes through a series of stages
- In ILP, the pipeline involves fetching instructions from memory, decoding the instruction, executing the instruction and then writing its result to a register file
- In this way there is a continuous flow of each stage at all times and each process begins in a sequential order

## Superscalar vs VLIW
- VLIW involves software scheduling and operates on multiple instructions/word
- Superscalar involves hardware scheduling and scheduling is dynamic

## Data Dependencies
- Load-use dependency, which involves reading data stored directly after it has been loaded from an address
- Define-use dependency, which involves reading data stored directly after it has been defined
- Write after Read dependency, which involves writing data into a register directly after it has been utilised, and it is only correct so long as the register has been utilised before being written into
- Write after Write dependency — when an instruction uses a value that is overwritten in subsequent instructions, the initial must be done before the latter for the current result to be achieved

## Instruction Scheduling
- Static scheduling, which involves finding independent instructions at compile time (it is done by the compiler in software)
- Dynamic scheduling, which involves finding independent instructions at runtime (which is done by the processor — the hardware)
- Hybrid, which involves a combination of both static scheduling and dynamic scheduling

## Preserving Sequential Consistency
* When instructions are executed in parallel, processor must be careful to preserve the sequential consistency

## Types of Consistency
* **Strong** — preserves the actual execution order
* **Weak consistency** produces correct result, can execute out of order providing the code still delivers the correct result

## How fast can we go?
* Performance is limited by
	* Underlying algorithm (dependencies)
	* Compiled code (false dependencies, stating)
	* Actual hardware (resource restrictions)