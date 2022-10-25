# javascript Event loop
js是一门单线程语言，它的同步异步机制就是event loop实现的。 

event loop 是由三部分组成 ：

* `调用栈 call stack`
* `消息队列 message Queue`
* `微任务队列 microtask Queue`

event loop 运行时遇到函数调用会把函数压入调用栈中， 被压入的函数叫做`帧 frame`, 函数返回时会从调用栈中弹出

当 setTimeout，setInterval 压入调用栈时， 其中的回调函数会压入`消息队列 message Queue`, 当调用栈清空时会把消息队列中的任务压入调用栈执行，这也是为什么即使设置为0秒执行，setTimeout也不会立即执行

其他异步操作会压入`微任务队列 microtask Queue` ,在调用栈清空时立即执行，并且处理期间新加入的微任务也会一起执行

微任务的执行 优先于 消息队列
