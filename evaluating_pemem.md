# Evaluating Persistent Memory Range Indexes
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


### 

#### 5.4 Single-threaded Performance

