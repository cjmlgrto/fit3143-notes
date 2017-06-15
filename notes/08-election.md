[← Return to Index](https://github.com/cjmlgrto/fit3143-notes/)

# Election Algorithms
* Distributed algorithms often elect one process to act as a coordinator or a server
* Election assumptions:
	* We assume each process is assigned a unique process number
	* A process knows all process numbers
	* A process does not know which process number is alive
* An election:
	* Select the maximum process number among active processes
	* Appoint the process the coordinator
	* All processes must agree upon this decision

## The Bully Algorithm
* When a process P notices that the coordinator is no longer responding to requests, it initiates an election
* P sends an _Election Message_ to all processes with higher numbers
* If one of the higher-numbered answers, that process becomes the new leader
* If no one responds, P wins the election and becomes coordinator
	* Note: timeout overhead, and delayed responses
* At any moment, a process can get an Election Message from one of its lower-numbered colleagues — the receiver sends an OK back to the sender
* The receiver then holds an election, until it is already holding one
* Eventually, all processes give up but one — and that is the new coordinator
* It announces its victory by sending coordinator messages to all processes
* If a process that was previously down comes back up, it holds an election
* If it happens to be the highest-numbered process currently running, it will win the election and take over the coordinator’s job
* The highest-numbered process always wins, hence the name “bully algorithm”

## The Ring Algorithm
* Processes form a logical ring
* Each process knows which process comes after it
* If a process finds the coordinator dead, it sends an Election Message which contains its own process number to its successor in the ring
* If the successor is down the message is sent to the next process along the ring
* Each process appends its own process number to the message upon receiving an Election message and forwards it to the successor
* When the message returns to the process that originally sent the message, the processes changes the message type from Election Message to Coordinator Message and circulates the message again
* The purpose of the Coordinator Message is to inform all processes that the highest-numbered process is the new coordinator and all the processes contained in the message are members of the ring