###oracle adg
####存储镜像复制
* 块断裂 
断电发生，写断裂，导致数据库无法启动 
因为存储采用到镜像复制，两边数据都是损毁状态，切换到备库，依然无法成功。
* 带宽
GB | DB 对带宽要求高；成本；
* 无法实时看到复制进度
没有办法看到数据是不是复制成功；
什么时候判定能切成功

#### 7.3 Standby-DB
1，手工将归档日志传输到备库；
2, 停主库业务，将实时日志传输到备库；
3，应用完实时日志后，切主；
// 8i后上述流程自动话

#### 9i Data-Guard 
1, 自动创建表空间；
2，支持归档日志GAP自动处理；（网络抖动、掉电、FAL自动追平GAP）
3，延时应用，并行Apply； (延时期间，主库操作失败，可从备库恢复）；
4，Logic DataGuard；（类似GG）主、备可有不同到表空间，备库是可写到；
   可以跨版本(滚动升级); (不支持大对象）
5，3个保护级别；


#### 10g 
before; 
切归档，才会备份；(相差一个归档）
切Redo， 会全量ckpt; ? *为什么*

Standby Redo log (Real time apply)
主库每次写的时候，备库的StandByRedo也会被写；
 
Recover through resetLogs;(新开一个归档编号）
基于时间点的恢复；

#### 11g
Active DataGuard; (可以在open 状态打开）
DataGuard (只能在mount状态打开）

1, 读请求offload到备库；

2, snapshot standby;
   Guarantee Resotore Point 技术实现；
   临时切换为可读、可写；
   场景：
   业务上线验证；(贴近真实库）
   临时业务验证；
   主库压力大，某些特殊大报表需要写入；

3, 跨平台DG；(从win 复制到Linux后switch over，linux 升为主库 // 需要停IO ）

4，BlockChangeTracking 提升备份性能；

5, 自动坏块修复；

#### 12c 
Far sync
    有限距离，同步复制；
    任何距离，异步；
    不存储数据，无数据库；
    
#### 18c
Data Guard DataBase Nologging (force logging)
1, Standy Nologging for 数据加载性能；
确保备用数据库将接收到为Redo到数据更改，而对主库对加载速度影响最小；
备库可以暂时具有nologed的块。 这些nologed的块将通过standby recovery 自动解决

2, for  数据可用；
确保在提交主负载时备库都具有数据；

#### 19c

自动中断解决

多实例Redo apply性能
 所有Rac节点并行恢复
DML 重定向；
 将写请求路由到主库；
 
 



主库、备库、延时库；


