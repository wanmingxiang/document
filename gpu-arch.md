![image-20230910222056844](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910222056844.png)

![image-20230910222144475](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910222144475.png)

![image-20230910222302864](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910222302864.png)



![image-20230910222450187](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910222450187.png)

![image-20230910222524830](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910222524830.png)

![image-20230910222622858](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910222622858.png)

![image-20230910231405403](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910231405403.png)

## 4 graphic processing cluster 

![image-20230910231442037](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910231442037.png)

## Each GPC has 4 streaming muti processor 

![image-20230910231840147](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910231840147.png)

## streaming muti processor have 4 BLOCKS,  4 * 8 = 32 CORES PER BLOCK,  

![image-20230910231946263](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910231946263.png)



## WARPS

1, streaming multi process 调度 threads in groups(32 parallel threads) ,  which are called warps;

2, 每个 SMP 有4个 warp schedulers；

3，每个scheduler 可以Run 2 instruction per warp every clock;

![image-20230910232250237](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910232250237.png)



![image-20230910232731344](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910232731344.png)



![image-20230910232827719](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910232827719.png)

![image-20230910233939448](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910233939448.png)

![image-20230910234633640](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910234633640.png)

![image-20230910234952323](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910234952323.png)

![image-20230910235223767](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910235223767.png)

![image-20230910235438325](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910235438325.png)

![image-20230910235651435](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230910235651435.png)

![image-20230911000003125](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230911000003125.png)

![image-20230911000213156](C:/Users/wanmi/AppData/Roaming/Typora/typora-user-images/image-20230911000213156.png)





https://www.youtube.com/watch?v=MC223HlPdK0&list=RDLV98Xis1W1mMk&index=5

## gpu direct

https://www.youtube.com/watch?v=vMODDx1Gwsw&t=102s

![image-20230911002901233](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230911002901233.png)

![image-20230911003335533](https://pics-1319111454.cos.ap-nanjing.myqcloud.com/image-20230911003335533.png)