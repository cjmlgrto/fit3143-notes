[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# SIMD Architectures
* 2D array of processing elements
* All processors execute the same instruction in parallel
* Each processor has its own local memory
* Processors are programmable
* Data can propagate through the processor array

## Granularity
* Fine-grain
* Course-grain
* Dependent on data density passed to the processors

## SIMD Connectivity
* Near-neighbour
* Tree
* Pyramid
* Hypercube

## Processor Complexity/Precision
* Single bit (fine-grain used for image processing)
* Integer (A compromise solution used for vision or general computing)
* Floating-point (Coarse-grain usually used for scientific computing)

## Local Autonomy
* Different sections at the hardware level can have different design as above 
* (i.e. a hypercube with single bit and another with a fine-grain granularity at near-neighbour with floating point, etc.)
