# rxjs总结

<!-- 个人总结，不绝对正确 -->

## Reactive Programming（响应式编程）

简单来说当变化发生时，由变化自身通知发生变化。

## Functional Programming（函数式编程）

简单的说是把任务拆分成函数`f(x)`，组合函数得出我们要的结果:

```typescript
(5 + 6) - 1 * 3
```

可以函数化为:

```typescript
const add = (a, b) => a + b
const mul = (a, b) => a * b
const sub = (a, b) => a - b

sub(add(5, 6), mul(1, 3))
```

函数一定有返回值，且不做与返回值无关的事情（给予相同的参数，永远返回相同，且无副作用）

## 观察者模式&迭代器模式

Observable迭代的推送消息，Observer观察推送的值，前者为生产者，后者为消费者；rxjs结合了这两个设计模式。

详细：https://blog.jerry-hong.com/series/rxjs/thirty-days-RxJS-04/

## Observable &  Observer

```typescript
import { Observable } from 'rxjs';
 //创建可观察对象
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
 // 订阅可观察对象取其中的值
// observable.subscribe() 内部实现就是观察者
// observer 实现的三个方法分别观察 Observable 的状态 
console.log('just before subscribe');
observable.subscribe( x => { console.log('got value ' + x); },
  err => { console.error('something wrong occurred: ' + err); },
  () => { console.log('done'); }
);
console.log('just after subscribe');
```

可以在控制台得到：

```bash
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```

## Subscription 

observable.subscribe()返回的实例就是Subscription  ，Subscription.unsubscribe() 可以取消订阅，停止像timer、interval 的无穷observable，以回收资源。

Subscription 还可以结合操作符合并订阅

## Operators

### creation operator

创建Observable 实例的操作符

#### of

```typescript
of(1, 2, 3)
```

同等与：

```typescript
new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});
```

#### from

```typescript
from([1,2,3])
```

等同于：

```
of(1, 2, 3)
```

from传入的可以是任意可迭代对象(`iterable`):Array, Set, WeakSet, String, Promise, Map等

#### fromEvent

```typescript
fromEvent(document.body, 'click').subscribe((e) => {})
```

类似于原生代码：

````typescript
document.body.addEventListener('click', (e) => {});
````

#### fromEventPattern *

<!--不清楚具体应用-->

类似fromEvent，创建有事件行为（有 addEventListener 及 removeEventListener行为的对象）的Observable 

```typescript
fromEventPattern(addListener,removeListener)
```

#### ~~empty~~ *

<!--不清楚具体应用-->

应用：https://rxjs.dev/api/index/function/empty

不发出任何值，直接触发Observer 完成

即将弃用 改用常量EMPTY

```
empty().subscribe({
  next: () => console.log('Next'),
  complete: () => console.log('Complete!')
});

// Outputs
// Complete!
```

等同于

```
import { EMPTY } from 'rxjs';

EMPTY.subscribe({
  next: () => console.log('Next'),
  complete: () => console.log('Complete!')
});
```

#### ~~never~~ *

类似empty，但不发出值，也不会结束，不会抛出异常

改用常量NEVER

#### throwError

不发出任何值，直接触发Observer error.

#### timer

`timer(n)`表示n毫秒后发触发next ， 发出 0 

`timer(n,m)`表示n毫秒发触发next，随后m毫秒触发next， 从0开始每次加1发出值

`timer(new Date(),m)`表示date时刻发触发next，随后m毫秒触发next，从0开始每次加1发出值

#### interval

`interval(n)`等同于`timer(n,n)`

### Filtering Operators

过滤操作符、订阅前过滤条件，在pipe()中调用

#### take

take(n) ，n次后取消订阅：

```typescript
fromEvent(document.body, 'click').pipe(take(1)).subscribe(console.log);
```

相当于原生代码：

```typescript
var handler = (e) => {
	console.log(e);
	document.body.removeEventListener('click', handler); 
}

document.body.addEventListener('click', handler);
```

## 参考

[30 天精通 RxJS]: https://blog.jerry-hong.com/series/rxjs/thirty-days-RxJS-00

[ rxjs ]: https://rxjs.dev/guide/overview

