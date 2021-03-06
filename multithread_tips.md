### 竞态(race condition)
#### 定义
当两个线程竞争同一资源时，如果对资源的访问顺序敏感，就称存在竞态条件。导致竞态条件发生的代码区称作临界区。
```
public class Counter {
　　protected long count = 0;
　　public void add(long value){
　　　　this.count = this.count + value;
　　}
}
```
以上这段代码，如果有A、B线程交替执行。
-   this.count = 0;
-   A:    读取 this.count 到一个寄存器 (0)
-   B:    读取 this.count 到一个寄存器 (0)
-   B:     将寄存器的值加2
-   B:    回写寄存器值(2)到内存. this.count 现在等于 2
-   A:    将寄存器的值加3
-   A:    回写寄存器值(3)到内存. this.count 现在等于 3


### 未定义行为（undefined behavior）
#### 定义
是指行为不可预测的计算机代码。这是一些编程语言的一个特点，最有名的是在C语言中。在这些语言中，为了简化标准，并给予实现一定的灵活性，标准特别地规定某些操作的结果是未定义的，同时，标准也从没要求编译器判断未定义行为，所以这些行为有编译器自行处理，在不同的编译器可能会产生不同的结果，又或者如果程序调用未定义的行为，可能会成功编译，甚至一开始运行时没有错误，只会在另一个系统上，甚至是在另一个日期运行失败。当一个未定义行为的实例发生时，正如语言标准所说，“什么事情都可能发生”，也许什么都没有发生。

#### C和C++的未定义行为的一些例子

- 尝试修改字符串常量会产生未定义行为
```
char * p = "wikipedia"; // C++11中错误，C++98/C++03不推荐使用
p[0] = 'W'; // 未定义行为

//正确的做法是定义成字符数组

char p[] = "wikipedia"; /* 正确 */
p[0] = 'W';
```
- 除以零会导致未定义行为
- 某些指针操作可能导致未定义行为
```
int arr[4] = {0, 1, 2, 3};
int* p = arr + 5;  //未定义行为
```
- 到达返回数值的函数（除main函数以外）的结尾，而没有一个return语句，会导致未定义行为
```
int f()
{
}  /* 未定义行为 */
//例
printf("%d %d\n", ++n, power(2, n));    /* 未定义行为 */
```
- 指针相关的常见未定义行为有如下内容：
  - 引用 nullptr 指针；
  - 引用一个未初始化的指针；
  - 引用 new 操作失败返回的指针；
  - 指针访问越界（解引用一个超出数组边界的指针）；
  - 引用一个指向已销毁对象的指针；
  
更多[undefined behavior](http://stackoverflow.com/questions/367633/what-are-all-the-common-undefined-behaviours-that-a-c-programmer-should-know-a)参考


