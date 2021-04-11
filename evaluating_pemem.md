# Evaluating Persistent Memory Range Indexes

### ABSTRACT
    - PM因为其持久性、低延时、等特点从根本上改变了索引等构建方式
    - 之前的研究工作缺乏真实PM硬件，都是基于DRAM模拟仿真

### 1. INTRODUCTION
PM 有跟DRAM和Flash截然不同的特性
* 比Flash有更高的耐久性（endurance）
* 比DRAM有更高的Latency，但比Flash低一个数量级
* 带宽比Dram低，且读写速度不对等

本文目标：
* 定质、定量定评估对比专为PM设计定range index;
* 分析不同定设计在PM真实硬件环境下定影响；
* 提炼有用定见解、设计指南；

关注B+tree based index structures;
- Concurrency control
    * Lock-based (FpTree, N-V tree)
    * Lock-free (bzTree)
    * HTM (FpTree)

测试后都发现
- 跟之前定研究结论相反，***PM带宽资源***非常珍贵，且对性能有很大定影响. 数据结构定设计要尽量避免耗尽整个带宽资源。
- 为了确保正确性、可用性，索引必须使用可靠的编程模型（PM Lib提供），之前的研究都规避了这个细节。相对于之前使用DRAM模拟得到都数据，
  在真实硬件上得到都数据显著减慢。


## 2. PERSISTENT MEMORY
PM 特点
- (1) byte-addressability,
- (2) non-volatility 
- (3) performance in the range of DRAM’s.
- (4) 在内存总线上，可以像普通内存一样使用（支持load, store)

### 2.1 Optane DC Persistent Memory

#### Performance
- Scales much better than DRAM 
    - 每个DIMM能提供512G的容量, 每个CPU能配置高达3T的DCPMM（6个槽位）
- Higher Latency
   - DDR4 (75ns) , Optane 300ns
- bandwidth
   - sequncial : 40GB/s(read), 10GB/s(write) (3x and 11x slower than DDR4)
   - random: 7.4GB/s (read), 5.3GB/s(write)  (8x and 14x slower than DDR4)
   - 

考虑到DRAM和PM之间的性能差距，利用更多的内存通道（即配备更多的dcpmm）来达到峰值带宽变得更加重要。
***图1***比较了两个和六个DCPMM下不同线程数的带宽。
CPU支持六个通道，每个通道有两个插槽，一个用于DRAM，一个用于PM。因此，dcpmm的数量也表示内存通道的数量。
与DRAM类似，添加更多的dcpmm可以显著提高PM的读性能，而对写带宽的影响相对较小。
值得注意的是，在大多数情况下，使用两个线程就足以饱和PM写带宽。
在我们的实验中，除非另有说明，我们使用六个DCPMM。
正如我们将在第5节中看到的，PM带宽是一种稀缺资源，而更高的带宽（更多的dcpmm）对于获得高索引性能至关重要。
然而，拥有尽可能多的dcpmm并不总是更好的，因为最佳设置高度依赖于用例。
例如，由于每个CPU的可用内存插槽数量是有限的，因此可以降低的dcpmm数量，而得到更多的DRAM模块。这意味着用PM的较低成本和带宽增加***换取***更昂贵的设置和更大的DRAM容量，从而得到其较低的延迟访问。


#### Oprating modes
- Memrory
    - load & store 
    - leverage CPU caches 
    - acts as 大内存，不支持持久化, DRAM作为缓存使用
- App Direct
    - load & store 
    - leverage CPU caches 
    - 持久化，APP 需要审慎处理持久性、恢复、并发、性能；
- Index设计难点
    - leverage no-volatility
    - guaranteeing failure atomicity

### 2.2 Programming Persistent Memory
PM的使用给数据持久性、内存管理和并发性带来了新的挑战。
解决这些难题需要使用一个健全的编程模型[32]，该模型包括使用缓存线刷新指令和PM感知指针、分配器和并发控制机制。
这是设计持久数据结构的关键部分，通常使用PM编程库来完成[19,31,42]。本节概述了这些问题；我们将在第3节和第5节中详细阐述。

- Persistence
    - 难点  
        - CPU caches are ***volatile*** 
        - there is no way in software to *** prevent cachelines from being evicted ***, 
    - data must be properly flushed from the cache to PM eagerly for safe persistence. 
        - *** CLFLUSH, CLFLUSHOPT or CLWB *** instructions, 
        - whitch will *** flush the specified cacheline contents *** to the memory controller (write buffers) that cover PM. 
    - *** asynchronous DRAM refresh *** 可以保证在电源故障时，写入缓冲区的数据会保留在PM中。
    - CLFLUSHOPT and CLFLUSH 会淘汰已经Flush的cache line (严重影响性能）
    - CLWB 是针对PM的新指令，在不淘汰的基础上Flush
    - 应用程序需要使用SFENCE纺织CPU重排序指令, 确保正确性
    - CPU 在一个时钟周期只能保证8byte的原子写入. (大于8byte的写入可能分散在多个时钟周期，导致部分写入）
    
- Memory management. 
    

### 3.PERSISTENT B+-TREE STRUCTURES
- Traditional B+ tree (保持Node有序)
    - 需要移动数据，来放置新的KV，可能在故障时，导致数据不一致; 
    - 写放大; 
    
#### 3.1 Write-Atomic B+ -Tree (wBTree)
    - reduce cacheline flushes
    - reduce writes to PM
    - unsorted nodes 
    - bitmap (位图用于指示每个插槽是否包含有效) , 可以被原子的写入
    - wBTree在每个节点中使用一个间接插槽数组，每个条目按增序记录相应键的索引位置
    - Consistency
        - Atomic update of bitmap
        - Loging (split)
        - steps of inserting *** out-of-place *** 
            - indirection slot array is flagged Invalid and updated;
            - bitmap is updated in atomic

#### 3.2 NV-Tree
    - selective consistency
        - 叶节点持久化, 确保强一致性
        - 内部节点弱一致性,掉电后需要重建(可以选择DRAM）
        - unsorted leaf nodes 
        - 插入会直接附加一个正标志（或者在删除时附加一个负标志）
        - *** leaf node counter *** (叶计数器)以原子方式递增以反映插入
        - 逆序扫描叶节点以查KEY的最新版本：如果其标志为正，则该KEY存在且可见；否则，该KEY已被删除
        - 内部节点使用连续内存存储(避免指针）, 提高cache命中率(重建代价高）
        - 为了避免频繁重建，内部节点使用稀疏的方式重建(高内存占用）
        - As inner nodes are immutable (except parent-2-leaf nodes), access without locking
        - lock at leaf node, and their parent level when traversing the tree

#### 3.3 BzTree 
    - lock-free (persistent multi-word compare-and-swap (PMwCAS))
    - BzTree stores both inner and leaf nodes in PM.  
    - Inner nodes are immutable (copy-on-write) (除更新child pointers外)
     
#### 3.4 Fingerprinting Persistent Tree (FPTree)     


###  5.EXPERIMENTAL EVALUATION

* HardWare *
- Intel Xeon Platinum 8260L CPU
- 1.5 TB of Optane DC PM (6 × 256 GB DCPMMs) 
- 96 GB of DRAM (6 × 16 GB DIMMs)
- The CPU has 24 cores (48 hyperthreads)
- 36 MB of L3 cache, and is clocked at 2.40 GHz.

#### 5.2 Index Implementations


#### 5.4 Single-threaded Performance
* Lookup
- Fig (7a , 8a)
    - Inner Node 放置在DRAM的数据结构(FpTree, NvTree比完全基于PM的数据结构有更高的**** 吞吐***
    - Fptree 的指纹进一步减少*** cacheline access ***
        - finger prints & bimap
        - 可能命中的数据
    - NVTree Append only, 需要扫描平均一半的entries
    - BzTree Hybrid, 如果在sorted 区域未找到，需要扫描未排序区域

- Fig (9a, 10a)
    - NvTree比FpTree更多的PMRead, 更多的L3 cache miss 
- Device R/W（介质访问), PM R/W (memory controller)
    - best case: App 重逢利用DCPMM buffer, Device R/W 跟 PM R/W 相同。 
    - worst case : Device  R/W 是 PM R/W的4倍；
    - 因为256byte(PM Block) 只有64byte(cacheline)被使用。

- Fig (10a)
    - BzTree 更少的cache miss，原因是small page(1Kb), 8byte Key, 实现采用来linear search, 
      相比binary search对cache 更友好。
    - BwTree 使用binary search, cache miss(higher latency of PM)导致其没有更好的查询性能（吞吐）

* Insert

    - 全部强一致性;
    - Insert, Update, Delete 都必须CLWB 强制刷新CPU cache, CPU Cache 无法隐藏PM Write都高时延
    - 标准差很大（memory controller， DCPMM)
    - 影响写性能都点包括：
        - Flush 次数
        - 每次插入所需要都维护操作
        - 节点分裂
    - FPTree ( , bitmap, fingerprint)   
    - wBTree ( , bitmap, slotArray, validity bit)
    - NVTree ( , counter)
    - BzTree ( , 2 PMwCAS) 
    
    - Bz Fp wB 都节点分裂可能传导到Root节点, NVTree 都分裂需要整个内部节点都重建。(PM Write) 

* Update
    - 更新只操作已经存在都Key，不涉及内存申请和节点分裂（7c可以看出标准差低于写）； 
    - NVTree 的更新比插入慢（先删后增） [对应9c的PM Write]
    - WbTree 的更新比插入快（更少的flush次数）[少一次validity bit]
    - BzTree 也是比写没有PM Alloc， Node Split；
    
* Delete 
    - wB | FP 更查询的性能接近 (Looup flowed by a bit op)
    - NV 
    - Bz PMwCAS会导致更多的flush
    - 
* Scan 
  - from a start Key, read the flowwing records;
  - 只有wBTree 可以顺序的访问，其他都需要额外的排序；
  - 读更少的数据（如FP）不能补偿排序、过滤的开销；

* Skew workloads
    - insert/delete 针对相同Key， 只有第一个会成功（后续的相当于只有一次查询）[忽略]
    - 单线程下并没有带来特别大的提升；
    - 
#### 5.5 Multi-threaded Performance
我们将wBTree的单线程性能作为参考，因为它不支持并发。
在所有实验中，
1.首先加载具有1亿个键值对（8字节键，8字节值）的树，
2.然后测量运行阶段，由工作线程分摊执行1亿个KV操作。
由于PiBench专门使用一个线程来收集统计数据，因此我们将工作线程的数量缩放到23个，并使用32个和47个线程进行测试，以显示树在超线程下的行为。

* Uniform distribution
    - 所有操作随着线程数的增加，都表现出更好都吞吐量；
    - 在超线程下，FPTree 提升尤其明显；

* Skew distribution
    - 跟统一分布负载类似，随着线程增加都有比较高都吞吐；
    - BzTree & FPTree的更新23线程后表现更糟糕；
     
* Skew factor  
    - Lookup : lower contension 意味着访问更多的key(cache miss);
    - NVTree 变化不大： 只读情况下也需要 Node Lock, 影响了并发扩展;
    - Update 的趋势不太一样
    - Skew workload下的性能影响因素：
        - PM Access (单线程下主要因素）
        - Contension level(多线程下的关键因素）
        
* Mixed workload 
    



