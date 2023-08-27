

计算指令彼此独立的情况下，让计算单元并行比较容易，但通常计算指令之间存在彼此依赖。pipeline 是CPU能让多个任务并行的方法；

```
a1 += (v1[i]+v2[i])*(v1[i]-v2[i])
```
![image-20230705214305695](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230705214305695.png)

![image-20230705214341098](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230705214341098.png)

**破坏流水线**
流水线依赖于连续的指令流；
指令被获取、解码和执行
条件跳转(分支)会破坏这个顺序
CPU必须等待，直到它知道下一步要取哪条指令

****

CPU通常根据历史数据做预测，预测失败CPU开销巨大

``` __builtin_expect(cond) ```  业务不会一成不变，不一定总带来正向的效果



情况一：bool变量求值一般是short-curcuited 

方案： 转为算术、位运算。通过这种方法，把2个条件变量转换为1个条件变量；

```c
 		for (size_t i = 0; i < N; ++i) {
            if (b1[i] || b2[i]) {
                a1 += p1[i];
            } else {
                a1 *= p2[i];
            }
 		}
```

```c
        for (size_t i = 0; i < N; ++i) {
            if (bool(b1[i]) | bool(b2[i])) {
                a1 += p1[i];
            } else {
                a1 *= p2[i];
            }
        }
        
```

情况二： BranchLess ，将bool变量转换为整数

​	   用计算换误判带来的代价：

	  - 多出的计算量不大
	  - 分支预测不准

```c
        for (size_t i = 0; i < N; ++i) {
            if (b1[i]) {
                a1 += p1[i] - p2[i];
            } else {
                a2 += p1[i] * p2[i];
            }
        }
```

```c
        for (size_t i = 0; i < N; ++i) {
            unsigned long s1[2] = {             0, p1[i] - p2[i] };
            unsigned long s2[2] = { p1[i] * p2[i],             0 };
            a1 += s1[bool(b1[i])];
            a2 += s2[bool(b1[i])];
        }
```

```c
        for (size_t i = 0; i < N; ++i) {
            if (b1[i]) {
                a1 += p1[i] - p2[i];
            } else {
                a2 += p1[i] * p2[i];
            }
        }
```

```c
        for (size_t i = 0; i < N; ++i) {
            unsigned long s1[2] = {             0, p1[i] - p2[i] };
            unsigned long s2[2] = { p1[i] * p2[i],             0 };
            a1 += s1[bool(b1[i])];
            a2 += s2[bool(b1[i])];
        }
```









