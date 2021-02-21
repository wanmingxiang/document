# Persistent memory 

##  Features
- dram-like latency
- non volatile
- byte addressable
- high endurance(60 DWPD)
    - 每天整个容量擦写60次, 可以用5年, 是企业SSD 60倍
- NUMA-awear
- memory mode | App direct mode

### Memory Mode
- large memory cap
- dram used as a write-back cache
- data is volatile

### App direct
- pmem-awear app 
- non-volatile
- byte addressable

- access way
    - pmem --> block translation table --> lagacy fs --> write() read()
    - fsdax --> dax-awear fs --> mmap
    - dev dax --> mmap
    
### PMem internal
- Asynchronous DRAM Refresh (ADR)
    - data inside ADR is protected against power failure

### Persist data by CPU instructions
- MOV
- CLWB + sfence
- CLFLUSH + sfence

### Instruction reordering

## Chanege

### Efficency
- memcpy on Pmem is not efficency
    - no more than 4 GiB/s per core
- extremely slow for remote access (NUMA)
- Solution: I/OAT DMA engine
    - offload data movement and memory cpy
    - can be used to mv data between DRAM and PMem


### Redesign Journal Layout
- traditional disk hash 4K alignment size
- pmen has 64B aliment size
    - write amplification




