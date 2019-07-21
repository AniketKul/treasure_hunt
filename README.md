![pirate_treasure_hunt_cheers_460x345](https://user-images.githubusercontent.com/21697139/61595883-8165bc00-abb1-11e9-90b2-411db8872669.jpg)


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

#### Test

Flag: -wake or -w

Parameters: None

Return: Standard message Example: >> rummy -wake

This is a simple command to test if your interface is working. It doesn’t do anything but print a statement.

#### Gather

Flag: -gather or -g

Parameters: None or the number of clues to generate

Return: Standard message

Example: `rummy -gather <number>`

This will instruct the Captain to collect all the clues he can find before looking for a crew. This takes the images from data/maps and preprocesses them into usable clues saved in data/clues. The command may take a very long time to execute, depending on the number of images you selected and the number of processor cores available. This command only has to be executed once, and then you can reuse the clues for all your testing purposes. You can create your own or extend the current datatset. The parameter indicates the number of clues to generate. If no parameter is provided, it will use all 20 images. For testing purposes, use only one image, since this is faster and doesn’t waste your time. For your convenience, pregenerated clues can be found in data/clues.

#### Unlock

Flag: -unlock or -u 

Parameters: None

Return: Standard message 

Example: `rummy -unlock`

Since this is a distributed system, different processes might try to access files at the same time, causing a collision or inconsistency. For this reason files are locked allowing only one process to access them at a time. If you execute rummy in a normal way, all locks are released. However, if your program crashes or you exit it with Ctrl-C, some locks may not be released, causing rummy to wait indefinitely on the next execution. In such a case you can remove all locks by executing the unlock command.

#### Prepare

Flag: -prepare or -p 

Parameters: None

Return: Standard message 

Example: `rummy -prepare`

This will instruct the Captain to visit the pub on the port to signup crew members. This command has to be executed only once before your computation commences and may take some time to execute.

#### Add

Flag: -add or -a

Parameters: None or the number of members to sign up 

Return: List of new crew member IDs

Example: `rummy -add <number>`

This will instruct the Captain to add new pirates to the crew. The parameter indicates the number of pirates to add and defaults to one if omitted. A list of IDs for the new crew members are returned, which you will use at a later stage. Note that adding new crew members after being shipped out (refer to the shipout command), adds an additional time penalty to the execution. Hence, try to add all your crew members before shipping out. Note that a list is returned, even if only one member was added.

#### Remove

Flag: -remove or -r

Parameters: The IDs of the crew members to remove

Return: Standard message

Example:

```
rummy -remove "PirateID"
rummy -remove '["PirateID1", "PirateID2", "PirateID3"]'
```

This will instruct the Captain to remove pirates from the crew. Removing members does not introduce an extra time penalty, even when shipped out. You can either pass a single ID or a list of IDs. Remember to put the ID or the list of IDs in quotation marks.

#### Ship Out

Flag: -shipout or -s 

Parameters: None

Return: Standard message 

Example: >> rummy -shipout

This will instruct the Captain to ship out and start searching for the treasure. After this point your distributed processing starts and any new crew members added will induce and additional time penalty.

#### Clues

Flag: -clues or -c

Parameters: None or the ID of a pirate

Return: List of clues

Example:
```
rummy -clues
rummy -clues "PirateID"
```

This will instruct the Captain to hand out clues to individual crew members to go looking for the treasure. If no parameter is specified, a group of clues is returned. This list has objects, one for each pirate currently on the crew containing the pirate’s ID and a list of clues. Each clue has an ID and a string of data that must be processed later on. If you specify a pirate’s ID for the command above, only a single clue object is returned, containing the clue ID and data for the specific pirate. Note that retrieving clues is time consuming and it is advised to retrieve all the clues at once (faster) instead of a single clue at a time (slower). Also note that although the individual clues are assigned to a specific pirate, any pirate can process the clues. The clues are simply linked to a pirate so that you can keep track of constant failures due to corrupt pirates (which you then might want to remove). Once a clue has been released with the above command, it will not be released again. Hence, you can call the command sequentially with a pirate’s ID and a new clue is released on every iteration until none are left.

#### Verify

Flag: -verify or -v

Parameters: A single clue or a list of clues

Return: Standard message on success or list of failed clues

Example:

```
rummy -verify '{"id":"PirateID", "data":[{"id":"ClueID", "key":"ClueKey"}]}'

rummy -verify '[{"id":"PirateID1", "data":[{"id":"ClueID1", "key":"ClueKey1"},
{"id":"ClueID2", "key":"ClueKey2"}]}, {"id":"PirateID2", "data": [{"id":"ClueID3", "key":"ClueKey3"}, {"id":"ClueID4", "key":"ClueKey4"}]}]'
```

This will instruct the Captain to verify if the pirate that solved the clues and checked the island for the treasure is talking the truth. You can pass in the clues from a single pirate or a list of clues from different pirates. Each clue contains the clue ID and the key generated by your algorithm. Again, any solved clue can be passed in under any pirate’s ID. For instance, if a clue was released for pirate 1, pirate 2 may solve the clue, and pirate 3 may verify the key with the Captain. If all keys are successfully verified, a standard message is returned. Otherwise if some keys are incorrect, a list of failed clues is returned in the same format as the clues command. These clues have to be run through your algorithm again and the generated keys have to be reverified. This process continues until all keys for a given map are successfully verified. Once all keys are accepted, the next map is automatically unlocked and the new clues can be retrieved with the clues command and then verified with the verify command. Once all clues from all maps are verified, the treasure hunt is over. The final result will contain and additional attribute finished. Again, verifying keys is time-consuming and it is better to verify all at once, instead of one key at a time.

### Solving Clues

Each pirate or distributed agent has to solve clues. A clue is a string of characters that has to be scrambled in a certain way in order to get a key. This key is given to the captain for verification. Clues are solved through three main procedures, namely dig in the sand, search the river, and crawl into the cave. Each of these procedures require certain tools which are smaller operations, namely shovel, rope, torch, and bucket. These operations are discussed below. The input string is referred to as clue.

#### Shovel

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434

Example Output: 00333444445567788899ABBCCCCCDFF0A2B3C
```

1. Sort clue in ascending order. Digits should come before characters.

2. If the first character in clue is a digit, append the string ”0A2B3C” to clue. Else if the first character is alphabetic, append the string ”1B2C3D” to clue.

3. Remove the first character from clue.

#### Rope

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434

Example Output: A555BC25215CAB15B2ABA5Cd5B22AA5A
```

For each character x in clue: 1. If x is a digit:

(a) If xmod3is0,changexto”5”.

(b) Elseif xmod3is1,changexto”A”. (c) Elseif xmod3is2,changexto”B”.

(d) Else leave x as is. 2. Else if x is alphabetic:

(a) Convert x to hexadecimal and subtract 10. Hence ”A” becomes 0, ”B” becomes 1, up to ”F” which is 5.

(b) If xmod5is0,changexto”C”.

(c) Elseif xmod5is1,changexto”1”.

(d) Elseif xmod5is2,changexto”2”.

(e) Else leave x as is.

#### Torch

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434

Example Output: F9E8D701
```

1. Set x to the sum of all digits in clue.

2. If x is less than 100, square x.

3. Convert x to a string.

4. If the string length of x is less than 10, remove the first character from x and prepend ”F9E8D7” to it.

5. Else if the string length of x is greater equal to 10, remove the first 6 characters from x and append ”A1B2C3” to it.

#### Bucket

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434

Example Output: 80766FC0CB6F86B76C51084AD010CC5868
```

For each character x in clue: 1. If x is a digit:

(a) If x is greater than 5, subtract 2 from it.

(b) Else if x is less equal to 5, multiply it by 2. 2. Else leave x as is.

#### Dig In The Sand

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434 

Example Output: 000000002222222...CCCCCCDFF0A2B3C

Example Note: Due to the length of the output string, only the first and last parts are shown.
```

Dig in the sand for the treasure by using the shovel 100 times. Then use the bucket 200 times to get rid of the ground water. Then use the shovel another 100 times.


#### Search The River

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434

Example Output: 604044FC0CB4F64B404C806068AD060CC80646
```

Search the river for the treasure by using the bucket 200 times to empty the river.

#### Crawl Into The Cave

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434

Example Output: F9E8D7681
```

Search the cave for the treasure by using the rope 200 times and then lighting up the torch 100 times.


#### Solve The Clue

```
Example Input: 40938FC0CB3F48B98C7546AD05CC7434

Example Output: 6D4AD5F3CB5DD79A678D397CDB9AF434
```

In order to solve the clue, the pirate has to first dig in the sand, then search the river, followed by crawling into the cave. The key is retrieved by calculating the MD5 hash of the result from the previous three actions and converting the hash to uppercase letters (digits stay the same).








