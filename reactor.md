### reactor

The reactor design pattern is an event handling pattern for handling service requests delivered concurrently by one or more inputs. The service handler then demultiplexes the incoming requests and dispatches them synchronously to associated request handlers.

上述定义提到了reactor模式中的模块，包括：
- input (on or more)
- service handler (分发请求)
- request handler (处理请求)


![image](https://segmentfault.com/img/bVlyFO)
**Handle 句柄**标识socket连接或是打开文件；

**Synchronous Event Demultiplexer**：同步事件多路分解器：由操作系统内核实现的一个函数；用于阻塞等待发生在句柄集合上的一个或多个事件；（如select/epoll；）

**Event Handler**：事件处理接口

**Concrete Event Handler**：实现应用程序所提供的特定事件处理逻辑；

**Reactor**：反应器，定义一个接口，实现以下功能：
- 供应用程序注册和删除关注的事件句柄；
- 运行事件循环；
- 有就绪事件到来时，分发事件到之前注册的回调函数上处理；
