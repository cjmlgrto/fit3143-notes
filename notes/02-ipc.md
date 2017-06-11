[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Inter-Process Communications
**Inter-Process Communication (IPC)** requires information sharing among two or more processes. There are two methods for this:

1. Original Sharing (Shared Data)
2. Copy Sharing (Message-Passing)

## Message-Passing
* A **message-passing system** is a subsystem of a distributed system that provides a set of message-based IPC protocols
* Shields (abstracts) the details of complex network protocols and multiple heterogeneous platforms from programmers
* Enables processes to communicate by exchanging messages
* Allows programs to be written by using simple communication primitives such as `send` and `receive`
* Used as a foundation for building higher-level IPC systems like RPC and DSM

### Qualities of a Good Message-Passing System
* Simplicity
* Uniform semantics
* Efficiency
* Reliability
* Correctness
* Flexibility
* Securiity

## A Typical Message Structure
```
- Header
    - Addresses
        - Sender address
        - Receiver address
    - Sequence number (or ID)
    - Structural information
        - Type
        - Number of bytes
- Message...
```

### Considerations when building an IPC Protocol
* **Identity** — Who’s sending/receiving?
* **Network Topology** — How many senders/receivers? How many nodes overall?
* **Flow Control** — Is a message guaranteed? Should the sender wait for a reply?
* **Error Control and Channel Management** — What to do when things fail?

## Synchronisation
* A step is called **blocking** if the next step can’t be done unless it is complete
* A step is **non-blocking** if the next step can be done at any time

### Synchronous
* When processes need to be done sequentially and each process is blocking
* Simple and easy to implement
* Contributes to reliability
* No backward error recovery needed

### Asynchronous
* Steps in the processes may occur at the same time, out of order, or at least before other steps complete
* High concurrency
* More flexible than synchronous
* Lower deadlock risk than in synchronous communication

## Buffering
Metaphorically, **buffering** means queuing sent/received messages to and from one processor to another

* **Sync systems** can have either a no-buffer or a single-message buffer
* **Async systems** can have an unbounded capacity buffer or a finite message (also known as a multiple message buffer)

## Multi-datagram Messages
* There’s a limit to the amount of data that can be transmitted at a time. This is known as MTU (Max Transfer Unit)
* Messages bigger than the MTU have to be fragmented into several packets/datagrams
* Thus, messages that need to be sent over multiple packets are called **multi-datagram messages*
* Assembly/disassembly is the responsibility of the message-passing system

## Encoding and Decoding
* Needed if the sender/receiver have different architectures
* Homogeneous encoding/decoding needed for:
	* Using an absolute pointer
	* Knowing which object is stored in where and how much storage it requires

## Process Addressing
### Explicit Addressing
* `send(process_ID, message)`
* `receive(process_ID, message)`

### Implicit Addressing
* `send_any(service_ID, message)`
* `receive_any(process_ID, message)`

## Failure Handling
There are 3 possibilities for failure during message-passing:

* **Loss of request message** — the message sent doesn’t reach the receiver
* **Loss of response message** — the message sent arrives, but reply back does not reach sender
* **Unsuccessful execution of request** — the message sent crashes the receiver

### Examples of Failure Handling

#### 4-Message Reliable IPC Protocol
1. Client sends request
2. Server replies with an acknowledgement
3. Server processes request
4. Server sends reply to client
5. Client replies with an acknowledgement

#### 3-Message Reliable IPC Protocol
1. Client sends request
2. Server processes request
3. Server sends reply to client
4. Client replies with an acknowledgement

#### 2-Message Reliable IPC Protocol
1. Server sends request
2. Server sends reply to client

#### Using Timeouts
1. Client sends request (but then lost)
2. After a specified amount of time, client resends request (but then crashes the server)
3. After another timeout, client resends request
4. Server processes request
5. Server sends back reply (but then lost)
6. After another timeout, client resends request
7. Finally, server receives it, processes it, then sends reply
8. Client receives reply

## Idempotency
Here’s an example of an idempotent function:

```
getSquared(n) {
	return n*n
}
```

And here’s an example of a non-idempotent function:

```
debit(amount) {
	if (balance > amount) {
		balance = balance - amount
		return ('success', balance)
	} else {
		return ('failure', balance)
	}
}
```

If the first function was sent as a request to a server, then simply re-transmitting the message should it fail to be sent would cause no problem. **Idempotency** is when a function can be executed multiple times and would always return the same result:

1. Client asks for the answer to `getSquared(3)`
2. Request fails, so after a timeout, client resends again
3. Server receives the request, and replies `9`
4. Reply fails, so after a timeout, client resends `getSquared(3)` again
5. Server receives the request, and replies `9`
6. Client gets the correct result

However, if the second function were to occur between a client and a server and a failure occurs — then re-transmitting the same request would yield a different result every time.

1. Client asks to `debit(100)`
2. Server, with a `balance = 1000`, receives the request
3. Server processes request, and thus replies `(success, 900)`
4. Reply fails, so after a timeout, client resends `debit(100)` again
5. Server processes request, but this time replies `(success, 800)`
6. Client receives the reply, but for some reason, with a lower `balance` than expected

To fix this problem, we would need to implement **idempotency**. We can do this by:

* Adding a unique **sequence number or ID** with the message
* Using a **reply cache** to store previous requests and their results (so the server need not process them again)

## Group Communication (and their issues)
### One to Many
* Group management
* Group addressing
* Buffered and unbuffered multicast
* Send-to-all and bulletin-board semantics
* Flexible reliability in multicast communication
* Atomic multicast

### Many to Many
* Issues related as above and many-to-one communications also apply here
* In addition, ordered message delivery is an important issue (trivial in one-to-many or many-to-one communications)
* For example, two server processes are maintaining a single salary database — two clients process and send updates for a salary record (what will happen if they reach in a different order? Will sequencing of messages help in this case?)

### Ordered Delivery for Group Communication
#### No Ordering
* Messages are sent in an almost random order

#### Absolute Ordering
* The order in which messages are sent is the order in which messages are received
* Using global timestamp as message identifiers with sliding window protocol

#### Consistent Ordering
* All messages are delivered to receivers in the same order
* However, this order may be different in which messages were sent

#### Causal Ordering
* If the event of sending one message is causally related to the event of sending another message, the two messages are delivered to all receivers in correct order
* Two message sending events are said to be causally related if they are co-related by the happened-before relation