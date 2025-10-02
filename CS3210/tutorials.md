# tut 1

![](<Screenshot 2025-08-29 at 5.20.01 PM.png>)
## 1. what are the components that reduce latency (inc parallelism)

- DDR4
  - DDR4 (Double Data Rate 4) is the fourth generation of RAM technology, designed to be a faster, more efficient, and higher-capacity replacement for its DDR3 predecessor
- UPI
  - UPI (Ultra Path Interconnect) is a high-speed, point-to-point interconnect that allows multiple CPUs in a server to communicate with each other. It is essential for multi-socket systems, forming the data backbone for complex, high-bandwidth workloads.
- Omni path
  - Intel Omni-Path Architecture (OPA) was a high-speed, low-latency fabric technology designed to connect multiple Intel Xeon processors for high-performance computing (HPC) clusters, offering high bandwidth and scalability.
- Omni path HFI
  - host fabric interface
  - enables tight integration and efficient communication between CPUs, memory, and storage


- Multicore processing
- 6 channels of DDR4 -- multi data multi instruction
  - also multiple ALUs = multi data

### answer
1. superscalar
2. thread level //: entire diagram represents the pipeline for one core with 2 hardware threads -- they share the resources in the core
3. process-level //: multiple cores: each core runs individually == //

### notes
- UPI: is used to connect between 2 cpus (connect between 2 processor) high bandwidth, low latency
- OPA: to connect between 2 different node (1 network node to another network node) -- faster than the internet
- where to find i/o parallelism
  - multiple DDR4 channels: memroy split?? 
    - can run independently of each other, PCIexpress lanes



## 2. flynn's taxonomy
a) personal computer from the 1980s
- probs SISD? no parallelism yet?
b) laptop with multi-core processor
- MIMD, many instructions are executed at a time and there are also data processes //ly
c) AVX instruction
- SIMD, 1 instruction is executed on multiple data 
d) a uniprocessor (single core) with pipelining (ILP)
- sisd? because 1 instruction will be processed by 1 alu
- instruction level parallelism: instructions are run in // across time -- ie at a random point in time the observation is //, but instructions are fed one by one but at different stages
e) students attempting the same exam in an exam hall
- SIMD, 1 instruction executed by many students 
  - Answer: MISD
    - different instruction sets on single data

## 3. memory + multi-core archi
a) shared memory system implies a UMA archi
- no, can be UMA or NUMA, NUMA: non uniform access -- each pu has diff probability of accesing the shared memory (diff priority)
- answer: no! memory access need not be uniform
  - data needed for executing on one core should be within the same NUMA node as the processor (in the shared memory)
  - takes longer to communicate if theyre not in the same NUMA nods: messy + complicated
  - in NUMA: data is localised on the memory that is closer to the cpu core
  - in UMA: data is shared with all memory (stored in both places)
    - but means that the memory allocation is distributed across different memory space
      - higher latency
      - high bandwidth -- can pull from more memory sticks
b) on a numa, actual location of data has no impact for a distributed shared-memory program
- falsee -- if the shared memory is on another socket it takes longer to reach

c) in hierarchical design, the memory org is hybrid (distributed-shared memory)
- not confirmed? because hierarchical design is about the cache, not the memory
- answer: both can be argued
  - not hybrid: caches + memory can be seen as one shared memory (one whole chunk)
  - hybrid: could see cache as its own memory that can be out of sync

## 4. processes and threads
a) a semaphore can be used to replace a mutex without affecting the correctness of a program
- true? if "replace" also means initialising a value (eg 0), wait = -1, signal = +1?
- answer: mutex can be replaced by semaphores, semaphores cannot replace mutexes (different behaviour: mutexes locked by thread one cannot be unlocked by thread 2? idk)
b) implementing an algo using multiple threads always runs faster than implementing the same algorithm using multiple processes
- true? context switching is faster, registers are easily shared between threads > processes.

## part 2
?? sockets vs cores vs threads
- logical cpus = CPUs x threads per core
- sockets
- cores

## intro to slurm
what is slurm
- job allocation system
- fair allocation of compute resources
- exclusive access to nodes for performance measurements

- system:
  - user logs in using the login node, 
  - slurm gets a job, appends it to the task queue
  - the tasks are passed to compute nodes

- NFS
  - Network File System
  - Home directory is not localised to a node: in 02 and 03 will have the same files
  - files are sharded and distributed into different servers
  - hence will be slower

### spriority
`sprio` -- `sprio -l` (shows who has more priority)
- if you use slurm more, your priority goes down

### sstat
`sstat` -- a perf profiler for batch jobs

## Exercise 3
- hostname: 009
- srun hostname: 004
- srun = ask slurm to run something




# tut 2
## qn 1
- Critical path length: max execution time given sufficient processes for max paralelism
- Ave concurrency: sum of work / critical path length
	- means theres no use to have more than 3 processors
### algo for getting max processors
1. draw concurrency-time bar graph 
	1. a row is a processor, 
	2. stack processes if they can be run in parallel.
	3. should be in order of dependency (dependent cannot be run after the dependee)
	4. try to fit processes into as little "rows" as possible

### dependency graph
- less edges = less dependency = more concurrency = can be optimised using more cores

## qn 2
- CPI is not a good metric, not all instructions are made equal. even though CPI might be higher for 1 process vs another, it could be running heavier instructions(?), doesnt mean it is less 
- example: total * = 5 vs total * = 66, the compiler will shift left to +5, and do integer multiplication for x 66. (shift left is faster)
- https://www.agner.org/optimize/instruction_tables.pdf

## qn 3
- MIPS = million instructions per second
- CPI for each instruction: I1 = 1, I2 = 2, I3 = 3
- How long does each program take = Cycles/clock speed. 
	- Cycles = # of cycles/instruction  x # of instruction
	- Clock speed = clock rate = 10x10^9

## qn 4
### Amdahls Law
![[Pasted image 20250917103747.png]]
P = fraction of code that can be parallelised. 
1-P = fraction of code that is sequential
1-P = N/N+N^2
1-P = 1/1+N

### when to use Amdahls Law vs Gustafans law
- Strong scaling vs weak scaling: how much will the program performance improve when you increase the # of instructions + processors? (i hv no idea i js guessing. see whats the diff and when to use each)

## qn 5
![[Screenshot 2025-09-17 at 10.47.26 AM.png]]
### programming models
- Fork Join could be a possible option
	- may be too complex
	- but also for relatively homogenous tasks
- Master worker good option
	- workers are tasked by master.
	- relatively homogenous tasks (same time to finish)
- Task pool could be an option but not the best
	- workers will pull from shared memory
	- usually for heterogenous tasks that finish at unknown times
- Pipelining wrong
	- different stages of a task, data transferred throughout the pipeline. matrix mul only need to mul (SIMD)
- Producer consumer wrong
	- similar to pipelining

