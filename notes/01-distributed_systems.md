[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Distributed Systems

A **distributed system** _(abbrev. DS)_ is a collection of independent computers that appears to its users as a single coherent system.

## Types of Distributed Systems

1. **Tightly-coupled Systems** — a network of processors that share a _single memory address space_ (either physical or virtual).
2. **Loosely-coupled Systems** — processors with _each of their own local memory_ that work together via “communication network”.

## Models of Distributed Systems

### Minicomputer Model

- Simple extension of centralised time-sharing systems
- Few minicomputers interconnected by communication network
- Each minicomputer has multiple users logged on
- Model used for remote resource sharing between users
- e.g. ARPAnet

### Workstation Model
- Several workstations connected by communication network
- Basic idea is to share processor power
- User logs onto home workstation and submits jobs for execution, system might transfer one or more processes to other workstations
- Issues: how to find an idle workstation, how to transfer processes, what happens to a remote process, etc
- e.g. Sprite system, Xerox PARC

### Workstation-server Model
- Consists of a few minicomputers and several workstations (diskless or diskful)
- Minicomputers are used for providing services
- For higher reliability and better scalability, multiple servers may be used for a purpose
- e.g. The V-System

### Processor-pool Model
- Sometimes user needs **no** computing power, but once in a while needs a very large amount for a short period of time
- Run server manages and allocates processors to different users
- No concept of a home machine (i.e. user logs not into one machine but to the system as a whole)
- Offers better utilisation of processing power compared to other models
- e.g. Amoeba, Plan9, Cambridge DCS

### Hybrid Model
- To combine the advantages of both workstation-server model and processor-pool model a hybrid model may be used
- It is based on the workstation-server model but with the addition of a pool of processors
- Very expensive!

## Distribution Models

### File Model
- Resources are modeled as files
- Remote resources are accessed simply by accessing files

### Remote Procedure Call Model
- Resource accesses are modeled as function calls
- Remote resources accessed by calling functions

### Distributed Object Model
- Resources are modeled as objects which are a set of data and functions to be performned on the data
- Remote resources are accessed as objects

## Advantages of Distributed Systems

### Reliability
- Only X% degradation on performance if X% of machines are downed
- System as a whole does not go down

### Incremental Growth
- Computing power can be added in small increments

### Sharing
- Allow many users access to a common database and peripherals

### Communication
- Make human-to-human communication easier

### Effective Resource Utilisation
- Spread the workload over the available machines in the most cost-effective way

## Disadvantages of Distributed Systems

### Software
- Difficult to develop distributed software rather than a centralised one

### Networking
- The network can saturate or cause other problems

### Security
- Easy access also applies to secret data

## Challenges in Distributed Systems

### Heterogneity
- Within a distributed, we have variety in networks, computer hardware, OSes, programming languages, etc

### Openness
- New services are added to distributed systems. To do that, specifications of components — or at least the interfaces to components — must be published

### Transparency
- One distributed system looks like a single computer by concealing distribution

### Performance
- One of the objectives of distributed systems is achieving high performance out of cheap computers

### Scalability
- A distributed system may include thousands of computers
- Whether the system works is the question in that large scale

### Failure Handling
- One distributed system is composed of many components
- That results in a high probability of having failure in the system

### Security
- Because many stakeholders are involved in a distributed system, the interaction must be authenticated, the data must be concealed from unauthorised users, and so on

### Concurrency
- Many programs run simultaneously in a system and they share resources
- They should not interfere with each other


