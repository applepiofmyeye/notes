- [L2 Processes, threads and synchronisation](#l2-processes-threads-and-synchronisation)
  - [parallelisation steps](#parallelisation-steps)
  - [processes](#processes)
  - [create a new process in unix](#create-a-new-process-in-unix)
  - [why fork](#why-fork)
  - [inter process communication](#inter-process-communication)
  - [process interaction with OS](#process-interaction-with-os)
  - [disadvantanges of processes](#disadvantanges-of-processes)
- [threads](#threads)
  - [use rlevel threads](#use-rlevel-threads)
  - [kernel threads](#kernel-threads)
  - [mapping of processes to threads](#mapping-of-processes-to-threads)
  - [visiblity of data](#visiblity-of-data)
  - [number of threads / cores](#number-of-threads--cores)
  - [synchronisation](#synchronisation)
  - [shared resources](#shared-resources)
- [L3: hardware parallelism](#l3-hardware-parallelism)
  - [computer architecture](#computer-architecture)
  - [types of parallelism](#types-of-parallelism)
    - [bit level parallelism](#bit-level-parallelism)
    - [instruction level parallelism](#instruction-level-parallelism)
      - [pipelining](#pipelining)
      - [superscalar](#superscalar)
      - [SIMD](#simd)
    - [thread level parallelism](#thread-level-parallelism)
      - [SMT -- simultaneous multi threading](#smt----simultaneous-multi-threading)
    - [processor level parallelism (multiprocessing)](#processor-level-parallelism-multiprocessing)
  - [Flynn's Taxonomy](#flynns-taxonomy)
    - [instruction stream](#instruction-stream)
    - [Single Instruction Single Data](#single-instruction-single-data)
    - [SIMD Single instr. Multiple Data](#simd-single-instr-multiple-data)
      - [AVX](#avx)
    - [MISD Multiple Instruction Single Data](#misd-multiple-instruction-single-data)
    - [MIMS multi instr. multi. data](#mims-multi-instr-multi-data)
    - [Variant -- SIMD + MIMD](#variant----simd--mimd)
  - [Multicore Architecture](#multicore-architecture)
    - [hierarchical design](#hierarchical-design)
    - [pipelined design](#pipelined-design)
    - [network based design](#network-based-design)
    - [Future trends](#future-trends)
  - [memory organisation](#memory-organisation)
    - [execution on a processor](#execution-on-a-processor)
  - [types of // computers](#types-of--computers)
    - [distributed memory systems](#distributed-memory-systems)
    - [shared memory system](#shared-memory-system)
      - [cache coherence](#cache-coherence)
      - [memory contention](#memory-contention)
        - [Uniform Memory Access (UMA)](#uniform-memory-access-uma)
        - [Non uniform memory ccess](#non-uniform-memory-ccess)
      - [summary](#summary)

# L2 Processes, threads and synchronisation

## parallelisation steps

- Decomposition of computations -- done by programmer
- Scheduling -- assignment of tasks to processes (or threads) -- done by programmers
- Mapping of processes (or threads) to physical processors (or cores) -- done by the OS and libraries
  - how does it execute on the physical cores? (decided by mapping)
  - OS is the one that schedules them on the cores when theyre available.
    - Most of the time OS does scheduling, but programmer **can** decide which threads run on which cores

![](./L2-parallelisation-steps.png)

- from Application to hardware, need to have several steps
- Applications -> databases -> goes through different system libraries
- For processes to be creayed and operated by OS, need system calls
  - allow us to make processes or threads
- Kernel manages processes or threads and execute them on diff cores
- Programer uses system calls (calling through programms)
  - or libraries can be used to eventually call system cales (indirectly)
- system calls can cause alot of overhead, when you end up often calling them to make processes and threads (to manage all the processes)

![](./L2-application-devices.png)

## processes

- an instance of a program in execution
- process comprises of
  - executable program (PC)
  - global data
    - OS resources, open files, network connections used by program
  - stack or heap
    - stack for how functions are called
    - heap keep dynamically allocated data of our program
  - keep track of what statements need to be executed by the programs
    - which functions have been called, what is the top of the stack
      - need a bunch of registers to help with that
    - General purpose registers + special purpose registers
- own address space -> exclusive access to its data
  - no other process that can access the virtual space of a process
- need a bunch of registers and information that has to be contained at the OS lvl
- when we execute a program, a process is created
  - OS keeps tracj and manages this process.
    - when the process starts and executes, how long they need to wait for etc
- need to exchange data, or communicate between 2 or more exchange dayta ->> need explicit communication like shared register etc

![](./L2-memory.png)

- OS has to keep track of the memory space

  - a virtual address space of the process
    - placed physically in memory by OS

- multi-programming
  - have more processes than availabl number of cores
  - all cores are busy with other processes,
- switching between processes is done by the scheduler of the OS
- multiple processes can potentially execute at the same time
  - need enough cores for these processes to execute in parallel

context switching: someone has to save the state of the process

- OS has to save the state of the process -- the registers, the virtual address space
  - be placed on the hardware before another process that is going to start executing

overheads:

- creating processes
- system calls
  - save state, etc
- context switch

  - save the current state and replace with another process

- time slicing + parallel execution of processes on diff resources
  - on multiple cores at the same time
  - can happen if you have multiple cores
- even with multiple cores
  - the processes and threads will take turns to execute on the same cores
  - some time-slicing execution of processes on different resources

## create a new process in unix

- fork system call
  - process P1 can create a new process P2. call fork from a process
- P2 is an identical copy of P1 at the time of the fork call
  - address space is identical
  - new process works on a copy of the address space
  - this copy doesnt happen at fork, but later when one of the processes modifies the address space
  - this copy needs to be executed from time to time
- P2 gets its own process number
  - `ps` or `top` to see the process number

![example of code](./Screenshot%202025-08-22%20at%2010.19.11 PM.png)

- in P1 fork will return the PID of p2 after the creation of P2
- in P2, fork will return 0
- if we see a return value of fork is 0, we are looking at p2
  - else we are in p1 and can keep track of the child process at p1
- fork has a lot of overheads

  - creating a new process, etc bring in alot of overhead

- new process => ready state, process is waiting to be executed
  - goes into running state
  - after its time slice has ended, goes back into ready state
  - while running, the process might do I/O request,
    - which means it will then go to a waiting state
    - after done, itll be in ready state.
  - after done, it is terminated

![](./Screenshot%202025-08-22%20at%2010.22.24 PM.png)

## why fork

- very useful when the child
  - is cooperating with the parent
  - relies upon the parents data to accomplish its task
  - eg a web server

![eg code of a web server](./Screenshot%202025-08-22%20at%2010.27.54 PM.png)

- more web services were written everyday
- server will wait for connections, accept connections on a socket
- server is waiting for connections, waiting for connections on a socket

  - client will have some requests, need processing of the requests.
    - if this was done in the main process of the server, cannot accept more requests and accept connections at the same time. hence, child processes are needed

- however, now we do not do `int sock = accept();`

  - too many processes = high overhead from context switching, short lived processes (creating and killing the process is expensive)

    - creating too many processes introduces alot of overhead

  - so, what can we do?
    - limit number of processes that can be created
      - a threadpool of processes executing.
        - one process from the pool will be used to accept connection and handle requests by client
        - but because theres no need to create and kill, less overhead.
    - multiplex + listen to multiple requests at the same time
      - receive from multiple sockets at the same time
        - networking system manages keeping down the number of processes

## inter process communication

- need to share info with each other

types

- shared memory
  - need to protect access when reading/writing with locks
- message passing
  - send-receive
    - blocking behaviour: the process that sends waits for the other one to receive
    - synchronous and async
- unix specific
  - pipes and signal

## process interaction with OS

| exceptions                                                                   | interrupts                                                                                 |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| executing a machine level instruction can cause exception, eg. overflow etc. | interrupts the execution of a program. usually hardware related. timer, mouse movement etc |
| synchronous -- occurs due to program execution                               | asynchronous -- independent of a program execution                                         |

## disadvantanges of processes
- crwating a new process is costly
  - overhead of system calls
- management of processes
- communicating between process is costly

> solution: threads

# threads

- an extension of a process
  - a process: multiple independent control flow
    - a thread has a single control flow
  - the thread defines a sequential execution stream within a process
- threads share the address space of the process

![](./Screenshot%202025-08-22%20at%2011.23.30 PM.png)
- diff control flows = different control floes

- because we dont copy address space, thread generation is faster, can just share address spaces
- different threads of a process can be assigned run on different cores of a multicore processor
- 2 types of threads
  - user-level threads -- created by users
  - kernel threads -- managed by OS (doesnt know how many user level threads of the programs)
- OS only works with kernel threads

whats the difference between kernel threads and processes?
- not much.
- processes are similar to kernel threads.
  - amount of kernel threads overhead is lighter than user level

## use rlevel threads
- manage dby thread library
  - OS is unaware of these threads
  - no OS support
  - OS doesnt know about user level threads
- thread library knows about the thread and decide what to do with them
- advantage: we can switch thread context very fast. -- is not in the kernel space, can switch threads without OS 

disadv: 
- OS cannot map different threads of the same process to different execution resources -- no parallelism
  - OS doesnt know they exist, wil not manage context switching

## kernel threads
- OS is aware they exist, can react corresponsingly, 
- efficient use of cores
- efficient use of cores in a multicore system
  - multiple kernel threads are good for multicore systems

## mapping of processes to threads
- many to one -- all user level are mapped to one process
  - disadv: OS doesnt know they exist, will take turns to run, not in parallel
  - library schedulers are usually much less optimised than OS scheduler
  - multiple threads of the same process are mapped to one kernel thread

![](./Screenshot%202025-08-22%20at%2011.34.34 PM.png)

- one to one mapping -- each user level thread is assigned to exactly one kernel thread -- no library scheduler needed
  - each user level thread is mapped to one kernel thread
  - OS scheduler can schedule them to execute on the computer
    - because now OS is aware of kernel threads (which map to user level threads)

![](./Screenshot%202025-08-22%20at%2011.35.31 PM.png)

- many to many mapping -- library scheduler assigns iser level threads to a given set of kernel threads
  - the user-level threads are not called threads in programming langs.
  - people expect one to one mapping for threads
  - many to many are called green threads 
  - green threads can run on a bunch of kernel threads
    - which are usually the number of cores available on the machine.
    - they get multiplexed on these kernel threads. goroutines.
  - a user thread may be mapped to a different kernel thread

## visiblity of data
- global variables of a program and all dynamically alocated data objects can be accessed by any thread of this process
- the function calls for eac hthread are added on the stack
  - each stack needs its own register.
  - each process needs its own thread 
- thread is created and starts executing at Runtime

![](./Screenshot%202025-08-22%20at%2011.50.17 PM.png)

## number of threads / cores
- not too many, kernel threads too many will be handled by OS
- not too little, need parallelism (make full use of resources)

factors:
- suitable to parallelism degree of application
- suitable to available execution resources
- not be too large to keep low the overhead for thread creation, management and termination small

> ## kernel threads are more like processes
> managed by OS, space managed by OS, context switching managed by OS. difference is that some kernel threads can share address space. 


## synchronisation
- threads cooperate in multithreaded programs
- share resources, access shared data structures
- coordinate their execution
  - one thread executes reletive to another
- scheduling is not under program control
  - use synchronisation to restrict the possible interleaving of threads executions

## shared resources
- coordinating access to shared resources
  - need to work with these shared resources
  - basic problem:
    - 2 concurrent threads are accessing a shared variable + that variable is read / modified / written by those threads, then access to the variable must be controlled to avoid erroneous behaviour
  - mechanisms to control access to shared resources
  - patterns for coordinating accesses to shared resources


# L3: hardware parallelism

## computer architecture
- concerns on // execution
- challenges of accessing memory
- understand and optimise performance
- gain intuition about what workloads are good for fast // machines

> is there any parallelism on a single core processor?
> ans: i think yes, multiple threads can execute at the same time on the same core

## types of parallelism
- bit level (single core)
- instruction level (single core)
- processor level (multi core)

### bit level parallelism
- by increasing the processor word size (64-bit architecture)
  - unit of transfer (data) between processor -> memory
  - apply the same instruction to multiple bits at the same time = bit level parallelism
- procesor work with words of diff sizes
  - multiple bits of the same word at the same time

![alt text](<Screenshot 2025-08-29 at 2.43.45 PM.png>)

- if we add 64-bit long integers, using a 32-bit archi, need 2 parallel add operations

### instruction level parallelism
- execute in //
  - pipelining
  - superscalar

#### pipelining
    - // across time
    - split instruction execution in multiple stages
      - instruction fetch, decode etc.
    - allow multiple istructions to occupy different stages in the same clock cycle
      - provided no data / control dependenccies
    - number of pipeline stages == max achievable speedup
  ![pipelining for instr. level //](<Screenshot 2025-08-29 at 2.47.13 PM.png>)

disadvantages
- cannot have dependencies
  - need to be independent instructions
    - hence might have a lot of hazards in the pipeline
- bubbles
  - delays of feeding the instr. into the pipeline
- hazards
- challenging to maximise the use of pipelining (finding independent instructions)

speculation
- try to figure out which branch of a decision instruction will be taken
  - when we have an if statement, guess whether if or else will be taken
  - try to feed into the pipeline as early as possible
    - but if the speculation is wrong, the pipeline has to be flushed, new instructions have to be fed into the pipeline
    - can lead to more bubbles 

out of order execution
- run one instr before something etc.

> cannot increase the efficiency already

![pipeline processor](<Screenshot 2025-08-29 at 2.55.46 PM.png>)

#### superscalar
- duplicate the pipeline
  - n instructions can run at the same stage in the pipeline at the same time
  - how? -- in the hardware they duplicate
- more instructions per cycle -> fewer cycles per instruction 


![superscalar processor](<Screenshot 2025-08-29 at 2.56.07 PM.png>)

- instructions come from the same execution flow (only one thread/process) has to execute and feed instructions into the pipeline.
  - only 1 execution context
  - if no instruction is fed into the pipeline, the OS has to find another control flow with instructions to execute and context switch: bring the instructions into the core, to execute another process

- 1 execution context = only at most 1 thread / process leaves / runs on the core


> how to increase the execution context?
> not enough ILP, but math operations are long
> cannot find enough instructions to run in //, but might be abit slow to bring abit of data to tthe execution processor

#### SIMD
- single instr. multiple data
- add more ALUs to increase compute capability
- more execution units -- speed up execution of instructions
- 8 units of data at the same time
  - less effort to bring data from the Fetch / decode step => ALUs,
  - as compared to fetching from memory
- same instruction is broadcasted to and uxecuted by all ALUs

> but if the data that we need is different, all 8 ALUs run different instructions and need to fetch data 
> can run in // as long as they are using the same data (no need to refetch data)


*Hence, designers thought: maybe allow multiple control flows executing on the core at the same time*

### thread level parallelism

#### SMT -- simultaneous multi threading

- multiple execution contexts maintained on the same core
  - means: can have instr. from multiple control flows that are immediately avail without a context switch for execution on that core

> in superscalar, in case need to execute from another control flow, the OS needs to context switch
>   - clear, flush all the context from the previous context, bring in a new context and run

- now: 2 context doesnt need to switch, can have 2 execution contexts
- 1 unit for each stage of the pipeline (fetch, decode, alu)
  - but now we can pass in 2 sets of instructions to fetch / decode, ALU, etc
  - can share these units

- each core can execute 2 threads at the same time
  - a number of logical cores (2) for each physical core

> existing architectures: superscalar pipeline + SMT

### processor level parallelism (multiprocessing)
- can do whatever thread, instruction, bit level parallelism in 2 cores

- in the end (after all the thread level etc. parallelism), they still will use the same single hardware core
  - if there are more threads, very difficult if it's complex to work on one core

- more cores = more data from memory
  - memory cannot keep up with the demand (cores compete to get data from memory) -- contention

- physical limits of what you can achieve in a chip


## Flynn's Taxonomy

- a taxonomy that is used to categorise diff. kind of architectures based on the instructions and the data streams that they use
  - proposed in 1972
- as new architectures appeared over time,
  - people tried to feed those architectures into the Flynn's taxonomy
- but now there's a mix of different kinds

### instruction stream
- a single executin flow
- ie a single Program Counter (PC)

### Single Instruction Single Data
SISD
- single inst. is executed one after another
- uniprocessor without any pipeline

### SIMD Single instr. Multiple Data 
- multiple data being processed
- Processing units are working on multiple streams of data
- data parallelism: multiple data processed at the same time, but with same instructions

nowadays:
- SISM exists in our cores (data // architectures)
- AVX instructions
- GPGPUs -- used to be more SIMD in the past
  - but the complexity of the processing unit increased a lot 
- can execute more than simd nowadays

- same instruction (from one program flow) executing on a bunch of execution units (ALUs)

#### AVX
- applying the same instruction n times (eg. 4 words added at the same time) -- one operation, one instruction, multiple execution 

- can guide compiler to use avx instructions in compiled code
- avx instr. work on very large words to execute 1 instruction for multiple words at the same time

- 4 operations -> 1 instructions (multiple data)

### MISD Multiple Instruction Single Data
- 1 data stream used at the same time to execute different instructions at the instruction code
- no actual implementation

### MIMS multi instr. multi. data
- each PU fecthes its own instruction
- each PU operates on its data
  - multiple threads, processes executed at the same time 

### Variant -- SIMD + MIMD
- multiple streaming multiprocessors (nVidia GPUs)
  - each SM is in a SIMD fashion
  - a set of threads executing the same code (effectively SIMD)

## Multicore Architecture
- how to design the architecture of the chip, to put in a number of cores, memory
  - what type of designs do you see 
    - hierarchical design

### hierarchical design
- multiple cores share multiple caches
- cache size increases from the leaves to the root

![hierarchical design of caches](<Screenshot 2025-08-29 at 4.20.09 PM.png>)

crossbar: like a bus
hyper-transport unit

note: each core has about 3 levels of caches
- lvl 1 is per core
- other caches are shared with each core

L1 -- cache
L1i -- intruction cache
L1d -- data cache

L2
L3

![alt text](<Screenshot 2025-08-29 at 4.27.41 PM.png>)

- b: bigger cores
  - have smt
- s: small cores
  - not smt

- smaller cores: for processors that are not time sensitive
- bigger cores: alot of computation
  - eg. game rendering

![how many cores?](<Screenshot 2025-08-29 at 4.30.38 PM.png>)

- 10 on each side, total 20

- on the chip there are 2 diff sockets, cores are groups together in the socket
- usually all the cores grouped in 1 socket
  - but if many cores, the cores are grouped together in sockets

### pipelined design
- data elements are processed by multiple execution cores in a pipelined way
  - use the same cache memory,
  - data is change dby diff cores in diff way
    - in routers / switch: the packet going through router / switch has to be handled depending on the header of the packet. 
    - first core will remove the first header, second core will remove another header, process the packet etc... 
    - every single stage will do some processing on that packet
    - the cores are all doing one specific action is very common in graphic processes
- useful if same computation steps have to be applied to a long sequence of data elements

### network based design
- cores and local caches and memories are connected via an inrerconnection network

### Future trends
- efficient on-chip interconnection
  - enough bandwidth for data trf between cores
  - scaling
  - robust to tolerate failures

## memory organisation
- processors run efficiently when data is resident in caches
  - bring the data as close as possible to the processing unit
  - people added more and more levels over time -- more caches are faster available to the execution unit
- PU needs to process instructions and data which it gets from cache (L1, then if dh then L2 then if dh then in L3)
- cache reduces trips to the memory
  - getting data from m emory is slow
  - memory is slow (it's like a s3 ice glacier vibes)
- avoid gg to memory at all costs
- if a data is stored in L1 cache, does it exist in L2 and L3 too?
  - depends on architecture, sometimes yes sometimes now

- memory latency
- memory bandwidth

### execution on a processor

![execution of addition operations](<Screenshot 2025-08-29 at 4.48.05 PM.png>)
- assume 8 clocks to transfer data (one add operation = one clock) + up to 3 outstanding load requests

- the 4th add operation needs more time to put the load on the bus: up to 3 outstanding load requests, 4th op need to wait for memory to become available. load instruction cannot be issued to memory.
- as more and more requests (more cores) compete for memory bus
- memory bandwidth and memory latency compared to cpu is much slower
  - organise computation to fetch data from memory less often
    - reuse data previously loaded by the same thread
    - share data across threads
    - cheaper to compute then to fetch data from memory

## types of // computers
![types of parallel computers](<Screenshot 2025-08-29 at 4.52.54 PM.png>)

### distributed memory systems

![alt text](<Screenshot 2025-08-29 at 4.52.54 PM.png>)
- each node is an independent unit
  - with processor, memory and, sometimes, peripheral elements
- physically distributed memory module
  - memory in a node is private

### shared memory system
- multiple PUs that share memory 
  - 1 memory or multiple pieces of memory (that appear to be 1)
  - accessed through a shared memory provider
- any of the PU can access the. memory through the load store operater
- some PUs may be faster for accessing memory
- program is unaware of the actual hardware memory architecture

#### cache coherence

![alt text](<Screenshot 2025-08-29 at 4.57.53 PM.png>)

- multiple copies of the same data exist on different caches
  - might not all have the same data, but if two caches save the same data, the 2 data caches will sync
    - how? -- a protocol -- cache coherence protocol -- ensures that the mutation of a data in one PU will be reflected across all caches
      - naively: PU updating the data will write to memory, PU using the new data will read from memory -- 2 memory calls = high latency (modern day cache coherence is v efficient)
    - avoid sharing lines of caches across PUs
- local update by processing unit -> other PUs should not see the unchanged data

#### memory contention
##### Uniform Memory Access (UMA)
- uniform access time to access main memory (same for every processor)
- suitable for small numbe rof processing unity -- due to contention -- fight for memory


##### Non uniform memory ccess
![alt text](<Screenshot 2025-08-29 at 5.05.05 PM.png>)
- physically distr. memory of all processing units are combined to form a global shared-memory address space: -- also called distributed shared memory
- accessing local memory is faster than remote memory for a processor
  - non uniform access time

if the memory is in another socket: higher latency 
- reduces contention
- using the same line of cache for data -- cache coherence

#### summary
adv:
- no. need to partition code or data
- no need to physically mode data among processors -- communication is eddicient
disdv: 
- special sync constructs are required
- lack of scalability fue to contention