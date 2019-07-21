![Uploading pirate_treasure_hunt_cheers_460x345.jpg…]()

## Introduction

My goal is to implement a distributed system which utilizes a group of individual agents. Each agent solves some particular task in order to reach a common goal. I learned how to implement stand-alone agents, how to elect a leader amongst these agents, how to communicate between processes and how to implement a fault-tolerent system.

## Problem Statement

Captain Rummy Rum, the greatest captain of all time, has overheard whispers of a treasure hidden on a remote island. Captain Rummy wants to hire new a crew of pirates to help him find the treasure. Captain Rummy has gathered all the clues indicating different islands that are likely to this treasure. Captain Rummy wants to find this treasure as quickly as possible. Captain Rummy wants to appoint a quartermaster from his crew that will co-ordinate the search in order to reduce the time needed to visit all the islands and find out if the treasure is present there or not. Some of the pirates in the crew want the treasure all to themselves and will try to fool captain and rest of the crew by falsifying the information. Captain Rummy has dealt with such dishonest pirates in his years of sailing and he can detect such lies. Captain Rummy may decide to get rid of such dishonest pirates by simply throwing them overboard. However, replacing these pirates is time-consuming since they have to sail back to the port to recruit new crew members. Hence, captain Rummy may decide to:

a) keep the dishonest pirates with the hope that they might behave themselves

b) throws them overboard and continue his search without any new pirates

c) replace dishonest pirates by going back to the port and continue his search with a full crew

The main objective is to find the treasure. The secondary objective is to the treasure as quickly as possible.

## Provided Information

We are given an existing program `rummy.py` representing Captain Rummy. This is a terminal based program accepting JSON inputs and providing JSON outputs.

We are required to program stand-alone agents, where each a agent represents a single crew member. These agents are individual executables/scripts which are only allowed to communicate with each other and with the captain through an external mechanism. Hence, we are not allowed to create a single program and have individual objects/functions representing the pirates and communicate through function calls, that is not a distributed system. If we run multiple agents, each of these agents should show up individually on the OS task/system monitor.

Some external communication mechanisms that we may want to use are:

• Files: Individual agents read/write to a shared file. We have to provide locking mechanism to avoid collisions during simultaneous writes.

• Database: Individual agents read/write to a shared database. Most databses have a built-in atomic operations to avoid collisions during simultaneous writes.

• Sockets: Use sockets to communicate between processes. This requires some standard and format to send commands and data across the socket.

• RPC Mechanisms: There are numerous available libraries that utilize sockets and HTTP to facilitate remote procedure calls via XML, JSON, or other formats.

• MPI Mechanisms: There are numerous available libraries that provide a message passing interface.

• Custom Mechanism: Create your own custom RPC, MPI, or other mechanism to communicate between processes.

### Leadership Selection

We will have to provide an interface to the provided program in order to facilitate communication between the Captain, the quartermaster and the crew members. We may choose to employ a leader or let every agent work on his own. The following two options are available:

• Central Leadership: Create a quartermaster agent who is the only one that communicates with the captain. All the crew members have to speak to the quartermaster, which in turn will speak to the captain. The crew members are never allowed to communicate with the captain.

• No Leadership: No quartermaster is employed and all pirates talk directly to the captain. You will have to rely on the throughput of the Captain, that is the rummy program, and he is lazy and slow.

### Dishonesty

Some of the pirates are dishonest and want to keep the treasure to themselves. Similar, in a distributed system, there may be malicious agents, corrupt network communication, or incorrect calculations. To simulate these problems, the Captain will hand out clues to the individual pirates. Each pirate has a random probability of being corrupt, which is determined the moment the pirate is added to the crew, and does not change over the lifetime of the pirate. When the Captain hands out clues to the crew, some of the clue data is corrupt, based on the corruption probability of the receiving pirate. Hence, if a non-corrupt clue is solved, the correct key is produced. On the other hand, if a corrupt clue is solved, the incorrect key is produced. The Captain knowns whether or not the key is valid, and if not, will instruct the pirate to solve it again with new clue data (which might or might not be corrupt again). We have to handle these dishonest agents in some manner:

i) Ignore: You may simply ignore corrupt agents. Depending on the probability, some agents might generate a lot of incorrect keys which then have to be resolved. This creates a major computational overhead, but does not require any additional code.

ii) Remove: If an agent is too corrupt, you can simply remove him and put the workload on the remaining agents. This requires only some additional code, but if too many pirates are thrown overboard, your remaining agents may be overused and since they run on a single thread, your CPU will not be fully utilized.

iii) Replace: You can throw corrupt pirates overboard and hire new ones. You thus keep a full crew all the time and can maximize your CPU utilization. Although removing an agent is cheap, adding a new one is costly and should be done sparingly. You may for instance decide to only add agents at the start of the program and not at the end when only one or two clues are left.

`Note`: If you decide to remove agents, you have to figure out exactly when to remove them. There are numerous algorithms to determine fault detection, although a lot of them do not apply to this "simple" situation.

### Networking

Since most of we do not have access to multiple computers, we are not required to implement a "true" distributed system that can run over the network. We can just run all our agents on a single machine. However, implementing the agents to both work locally and over the network is not difficult if we use databases, sockets, RPCs, or MPIs. 

### Choice of Programming Language

It is strongly advised to make use of Python, since the main program, rummy, is also written in Python and provides easy integration.

### Choice of Operating System

It is strongly advised to make use of Linux/MacOS. Hence, we may not utilize any platform-specific libraries or function calls.

## Provided Files

The provided program `rummy.py` is precompiled to prevent people from tampering with the core code and to provide a learning experience with precompiled tools which are often used in distributed systems.

`rummy.py` requires the Python PIL library to read the provided input images. On most Linux systems this should be installed by default. Otherwise, we can install it using `pip install image`.

## No Multi-threading

The distributed environment should be simulated on a single machine. We are not allowed to use any multithreading in your program. In order to simulate the distributed environment, we will have to create individual stand-alone agents/executables which each run on a single thread. Hence, if you have 8 cores on your machine, you can have 8 (or more) executables running independent and therefore utilizing 100% of your processor. However, you are not allowed to create a single executable that uses multiple threads to utilize 100% of your processor. If you want to fully utilize the processor, do not hardcode the number of cores, but rather detect them. Your code will be tested on various machines with a different core count, hence your code should automatically adapt to the system it is running on. Python has easy ways of detecting the system hardware. Remember that there is a difference between number of processors, number of cores, and number of threads.

## Interface

All data retrieval and verification must be done through the provided program rummy. In order to facilitate communication between rummy and your program, you can execute rummy as an external command and then interpret the JSON results. Python and many other languages provide integrated support to execute external commands. rummy has a number of flags and options which are discussed below.

Each result returned from rummy has the following JSON syntax:

`{"status" : string, "message" : string, "data" : string-or-list, "finished" : boolean}`

• status: Always present. Either success or error. Indicates whether or not your command executed successfully.

• message: Always present. A string that provides a human-readable description of the error or the successful command.

• data: Sometimes present. Returns a JSON string or list with the final data of your executed command.

• finished: Is only present if all maps and clues were solved and the treasure was found.

Note that each of the operations below require file read and write operations, encryption and decryption, and processing. Hence, you should try to reduce the number of commands you execute. For instance, if you want to remove three pirates from the crew, remove all three at once (faster) instead of one at a time (slower).






