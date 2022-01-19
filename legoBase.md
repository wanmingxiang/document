## DISAGGREGATED DATABASE ARCHITECTURE
###设计逻辑
- 缩小monolithic和 disagregated 架构性能差距
* 完全byPass的远程Page的访问
* 保留LRU的淘汰逻辑
* 特定数据裤的优化

- 本地和远端故障处理独立

## DATABASE-AWARE MEMORY DISAGGREGATION


## FAULT TOLERANCE AND STATE RECOVERY

### Two-tier ARIES Protocal

* Write Log Entry 2 Log Buffer;
* Update Page;
* 挂链到Flush-LBP；
* when commit, write WAL 2 Storage；

* LFT Daemon 将FLUSH_LBP的page定期的刷到remoteBuffer;
* 打tier-1 check point (Largest LSN);

* gmCluster 将DirtyPage挂到FLUSH_RBP
* HFT 刷盘到Storage；
* 结束后，天tier-2 checkPoint;

tier-1 chkpt 打完，Log Entry 不能删除(只能等到tier-2 chkpt）

### Consistent and Fast Flushes

- 2个挑战
 * CNode gmCluster并发刷同一个Page；
 * 

