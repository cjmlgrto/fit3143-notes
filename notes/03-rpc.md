[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Remote Procedure Call

* Remote Procedure call is an all-purpose IPC protocol
* Essentially, like a subroutine call except the subroutine executes remotely
* Widely accepted due to:
	* simple call syntax
	* familiar semantics
	* specification of a well-defined interface
	* ease of use
	* generality
	* efficiency

## A Typical Model of RPC
1. Client calls for a remote procedure
2. Server receives request and processes it
3. Server sends reply back to client
4. Client continues execution

## Implementing RPC
* Implemented via abstraction
* Caller/client calls subroutine as normal but calls to a **stub**
* A stub is responsible for sending the message via a communication system (**RPC runtime**)
* Which the also connects to a stub on the server
* Stub calls for the server's local routine and executes
* Stubs essentially hide the details of the communication between the caller and the callee

In general, we get:

* Client
* Client stub
* RPCruntime
* Server stub
* Server

## Parameter-Passing
### Call-by-Value
* Values of arguments are copied to the stack
* Subroutine modifies these copies values but does not change original arguments

### Call-by-Reference
* Memory addresses of the values corresponding to the arguments are put into the stack and sent to subroutine
* Subroutine modifies original values as a result

### Call-by-Copy/Restore
* Values of the arguments are copied to the stack then sent to the subroutine
* When subroutine completes, modified values are copied back to the original values at the calling side
* Changes on the values made by the subroutine are copied back to the original

### Data Interchange Formats
* Data passed as parameters/arguments need to be converted to a standard format between the client and the server
* Client can also send exact format to server, but server needs to know how to convert the client's file (and any other clients' files if they differ)

## RPC Messages
### Call Messages
* ID
* Type
* Client ID
* Remote Procedure ID
	* Program Number
	* Version Number
	* Procedure Number
* Arguments...

### Reply Messages
* ID
* Type
* Reply Status
* Result

## Variations of RPC
* Synchronous
* Asynchronous
* One-Way
* Callback
* Broadcast
* Batch-mode
* Lightweight

## Optimisations for Better Performance
* Concurrent access to multiple servers
* Serving multiple requests simultaneously
* Reducing per-call workload of servers
* Reply-caching of idempotent remote procedures
* Proper selection of timeout values
* Proper design of RPC protocol specification

## Concurrent Access to Multiple Servers
### Threads
* Different local threads can be assigned to different servers
* Should be well-implemented to handle correct address routing

### Early Reply Approach
* Call split into two calls: one for request and another for fetching Reply
* Request and result can be tagged for easy access

### Call Buffering Approach
* Middleman between server and client
* Both server and client ping for requests, replies and acknowledgement
* Similar to early reply, calls and replies are split into tagged requests and responses

## Serving Multiple Requests Simultaneously
* Queued requests
* Use a multi-threaded system
