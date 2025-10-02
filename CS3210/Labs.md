# lab 2
- if you ask perf to sample too many events it cannot give the exact, only gives an estimate based on the sampling it does
- i7-7700 has higher cycles than xs-4114
	- clock speed is different
- i7-13700 is faster despite having lower freq, 
	- the cycles are less but the insn per cycle is more due to more successful branch predictions

# Lab 3
- a grid runs on a kernel 
- one grid "square" is a group of blocks. 
	- each block is scheduled into exactly one SM. total of 1024 threads
- each block is grouped into groups of 32 threads called warps.

- each SM runs thread blocks

- in terms of speed: registers < shared < local = global
	- local memory is same as global memory: is a spillover of the registers. theyre in the same physical location as global memory
	- local memory: shouldnt use.

ssh joeylee@xlog.comp.nus.edu.sg
srun --gpus=1 bash -c "hostname; nvidia-smi"