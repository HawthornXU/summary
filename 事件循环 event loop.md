# javascript Event loop
mdn文档： https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop

js是一门单线程语言，它的同步异步机制就是event loop实现的。 

event loop 是由三部分组成 ：

* `调用栈 call stack`
* `任务队列 macrotask Queue`
* `微任务队列 microtask Queue`

event loop 运行时遇到函数调用会把函数压入调用栈中， 被压入的函数叫做`帧 frame`, 函数返回时会从调用栈中弹出

当 setTimeout，setInterval 压入调用栈时， 其中的回调函数会压入`任务队列 macrotask Queue`, 当调用栈清空时会把任务队列中的任务压入调用栈执行，这也是为什么即使设置为0秒执行，setTimeout也不会立即执行

其他异步操作会压入`微任务队列 microtask Queue` ,在调用栈清空时立即执行，并且处理期间新加入的微任务也会一起执行

微任务的执行 优先于 宏任务队列

## 宏任务 微任务

宏任务： 定时器、 http请求、文件读取

微任务： promise.then

## 一次循环执行顺序

1. 同步任务
2. process.nextTick (node 程序)
3. 微任务
4. 宏任务
5. setImmediate
