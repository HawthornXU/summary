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

```typescript
empty().subscribe({
  next: () => console.log('Next'),
  complete: () => console.log('Complete!')
});

// Outputs
// Complete!
```

等同于

```typescript
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

#### range

range(n,m) 立即按顺序从n开始发出m个值 每次加1

### Transformation Operators

转换操作符，在pipe()中调用

#### map

类似数组的map,所有通过的值经过map中的表达式转换成其他值发出。

```tsx
const source = interval(1000)
const newest = source.pipe(map(x=> x + 2));

source.subscribe( res =>  console.log(res));
// 0
// 1
// ...
newest.subscribe( res =>  console.log(res));
// 2
// 3
// ...
```

#### mapTo

把经过的值改为固定值

```tsx
interval(1000).pipe(map(2)).subscribe( res => console.log(res));
// 2
// 2
// ...
```

#### scan

相当于js中的reduce，可以把发出的值累加。

```typescript
var example = from('hello').pipe(
			scan((origin, next) => origin + next, ''));

example.subscribe(
   (value) => { console.log(value); },
     (err) => { console.log('Error: ' + err); },
     () => { console.log('complete'); });
// h
// he
// hel
// hell
// hello
// complete
```

#### buffer

buffer(n:Observable) , 根据参数Observable 的发出频率，按组发出通过的Observable 的值。

```typescript
interval(300).pipe(
	buffer(interval(1000))
	).subscribe( (value) => { console.log(value); },
    (err) => { console.log('Error: ' + err); },
     () => { console.log('complete'); }
);
// [0,1,2]
// [3,4,5]
// [6,7,8]...
```

#### bufferTime

bufferTime 可以优化上一个例子：

```typescript
interval(300).pipe(
	bufferTime(1000) )
```

#### bufferCount

bufferCount(n)，根据传入的数字分组发出值

```typescript
from([0,1,2,3,4,5]).pipe(bufferCount(2)).subscribe( (value) => { console.log(value); },
    (err) => { console.log('Error: ' + err); },
     () => { console.log('complete'); }
);
// [0,1]
// [2,3]
// [4,5]
// complete
```

#### concatMap

等于map + concatAll，把送过来的Observable 转成值后处理

可以根据特性发送request ,可以取到每次的值。

```typescript
fromEvent(document.body, 'click').pipe(concatMap(e=> { /** 发http请求***/ })).subscribe()
```

#### switchMap

等于map + switchAll

可以根据特性发送request ,频繁发送时可以取到最新的值

#### mergeMap

 等于mergeAll+map

**switchMap, mergeMap, concatMap可以把传回的promise 直接转成observable**



### Filtering Operators

过滤操作符、订阅前过滤条件，在pipe()中调用


#### filter

通过的值符合表达式才能发出

````tsx
interval(1000).pipe(filter( x => x === 2)).subscribe( res => console.log(res));
// 2
````

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

#### skip

skip(n) ，略n次后发出值

#### takeLast

takeLast(n)取Observable调用complete()前n个值，立刻同步发出

#### last

last() 同 takeLast(1)，或者接收两个参数

```typescript
const source = from(['x', 'y', 'z']);
const example = source.pipe(last(char => char === 'a','not exist'));
//output: "'a' is not exist."
example.subscribe(val => console.log(`'a' is ${val}.`));
```

#### first

取第一个符合其中表达式的值发出，无表达式直接取第一个

```typescript
interval(1000).pipe(first( x => x > 2)).subscribe( res => console.log(res));
// 3
```

#### takeUntil

接受一个Observable，其中的Observable发出值后，所过滤的Observable停止订阅

```typescript
const source = interval(1000);
const clicks = fromEvent(document, 'click');
const result = source.pipe(takeUntil(clicks));
result.subscribe(x => console.log(x));

// 订阅到网页受到点击
```

#### debounceTime

如其名称多用于防抖

```typescript
fromEvent(searchInput, 'input').pipe(
   debounceTime(300),
   map(e => e.target.value)
).subscribe((value) => {
    // 这里发送 request
  })
```

#### debounce

debounce接受一个Observable，使用方法类似buffer，bufferTime相对应debounceTime，单独控制每一个发出值的性状

#### throttleTime

用于节流

```typescript
var example = interval(300).pipe(take(5),throttleTime(1000));

example.subscribe((value) => { console.log(value); },
    (err) => { console.log('Error: ' + err); },
     () => { console.log('complete'); }
);
// 0
// 4
// complete
```

#### throttle

同样类似buffer，接受一个Observable，单独控制每一个发出值的性状

#### distinct

可以把通过的值去重，不传值直接比对通过的值，传入回调按照回调里的规则比对去重

其内部实现是set缓存，所以不要在无限值的Observable直接用

```typescript
from(['a', 'b', 'c', 'a', 'b']).pipe(distinct()).subscribe( 
    value => { console.log(value)},
    e => {console.log('e')},
    () => { console.log('complete')}
);
// a
// b
// c
// complete
const example = from([{ value: 'a'}, { value: 'b' }, { value: 'c' }, { value: 'a' }, { value: 'c' }])
example.pipe(distinct((x) => {
    return x.value
}).subscribe(    
    value => { console.log(value)},
    e => {console.log('e')},
    () => { console.log('complete')}
);
// {value: "a"}
// {value: "b"}
// {value: "c"}
// complete
```

#### distinctUntilChanged

和distinct一样用于去重，但仅仅会和上一次发出的值比较。

### Join Operators

#### concatAll

用来把多个Observable连起来，需要通过的值是Observable

```typescript
var click = fromEvent(document.body, 'click');
var example = click.pipe(map(e => Rx.Observable.of(1,2,3)),concatAll());
example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 每次点击页面 连续发出 1，2，3
```

```typescript
var obs1 = interval(1000).pipe(take(5));
var obs2 = interval(500).pipe(take(2));
var obs3 = interval(2000).pipe(take(1));

var source = of(obs1, obs2, obs3);

var example = source.pipe(concatAll());

example.subscribe({
    next: (value) => { console.log(value); },
    error: (err) => { console.log('Error: ' + err); },
    complete: () => { console.log('complete'); }
});
// 0
// 1
// 2
// 3
// 4
// 0
// 1
// 0
// complete
```

#### switchAll

和concatAll很像，但concatAll是处理完一个Observable再处理一个，switchAll是来了新的Observable立刻取消订阅旧Observable开始新值的处理

#### startWith

在通过的observer前面加一个值，立即发出

```typescript
interval(1000).pipe(startWith(233)).subscribe(res => console.log(res));
// 233
// 0
// 1
// ...
```

#### mergeAll

接收到值直接合并处理，不会取消订阅旧的。

### Join Creation Operators

#### concat

可以直接concat()，创建Observable ，也可以在pipe()中使用，把不同Observable 依次连接起来发出。

与concatAll的区别是，concatAll直接让Observable 对象通过取出其中的值，concat()把接收的参数Observable 连接起来发出。

#### merge

和concat类似，但concat是前一个Observable complete后，下一个Observable 才开始，merge是直接合并，同时开始。当Observable都结束，整个Observable 才算结束。

#### combineLatest

 接收可迭代的Observable，转换输出上一次的值。

[examples](https://rxjs.dev/api/index/function/combineLatest#examples)

#### zip

组合多个Observable ，每次发出值类似foreach，是每个Observable 的第n个值，有木桶效应，发出的个数等于最小个数的Observable 

[example](https://rxjs.dev/api/index/function/zip#example)

如果其中一个Observable 数量很少，其他的很多，没有在恰当的时机取消订阅，内存会一直记录其他连续发出的值，内存会得不到释放

#### withLatestFrom

和combineLatest类似，但在pipe中使用， 主Observable 发出时才发出，而combineLatest是其中任意一个Observable 就发出。

### Utility Operators

#### dalay

dalay(n:number|Date), 延迟n毫秒发出或在指定日期之后发出

#### delayWhen

接受一个回调函数，返回Observable 可以对每个值单独做处理。

```typescript
// 每次点击延迟 0~5秒
const clicks = fromEvent(document, 'click');
const delayedClicks = clicks.pipe(
  delayWhen(event => interval(Math.random() * 5000)),
);
//每个值分别延迟x*x秒
interval(300).pipe(take(5),
                  delayWhen(x => EMPTY.pipe(delay(100 * x * x))).subscribe( x => 
                                                                          console.log(x))
//--0---1----2-----3-----4
```



### 相关应用

#### 编写dom拖拽

```typescript
const dragDOM = document.getElementById('drag');
const body = document.body;

const mouseDown = fromEvent(dragDOM, 'mousedown');
const mouseUp = fromEvent(body, 'mouseup');
const mouseMove = fromEvent(body, 'mousemove');

mouseDown.pipe(
  map(event => mouseMove.pipe(takeUntil(mouseUp)) ),
  concatAll(),
  map(event => ({ x: event.clientX, y: event.clientY }))
).subscribe(pos => {
  	dragDOM.style.left = pos.x + 'px';
    dragDOM.style.top = pos.y + 'px';
  })
```

#### 视频网站浮动小视窗

[实例](https://stackblitz.com/edit/rxjs-demo-video-float)

#### 鼠标双击

```typescript
fromEvent(button, 'click').pipe(
    		bufferTime(500),
			filter(arr => arr.length >= 2)
            );
```



## 参考

30 天精通 RxJS: https://blog.jerry-hong.com/series/rxjs/thirty-days-RxJS-00

 rxjs : https://rxjs.dev/guide/overview

