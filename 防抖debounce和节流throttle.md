# 防抖 节流

**防抖** ：常适用场景在输入框, 输入字符后查询后端类似条目，根据使用场景来看，用户打字时输入停顿时查询才有意义。
 * 一段时间的停顿才发出事件

**节流** ：常适用场景在类似滚动条滚动事件触发一系列操作, 由于滚动事件发生很频繁，所以要减少触发频率。
 * 一段时间中只允许一/零次事件发生

## 防抖

```ts
let timer = null;

fooInput.oninput = ($event) => {
    if (timer !== null) {
        return clearTimeout(timer);
    }
    timer = setTimeout(() => {
        console.log($event.target.value);
    }, 500);
}
```

封装一下：
```ts

fooInput.oninput = ($event) => debounce(() => console.log($event.target.value), 500);

const debounce = (fn, delay) => {
    let timer = null;
    return () => {
        if (timer !== null) {
            return clearTimeout(timer);
        }
        timer = setTimeout(() => {
            fn();
        }, delay); 
    }
}
```

## 节流

```ts
let canTrigger = true;
document.onscroll = ($event) => {
    if (canTrigger) {
        setTimeout(() => {
            doSomething();
            canTrigger = true
        }, 500);
    }
    canTrigger = false;
}
```
封装一下：
```ts
document.onscroll = throttle(() => doSomething(), 500);
const throttle = (fn,ms) => {
    let canTrigger = true;
    return () => {
        if (canTrigger) {
            setTimeout(() => {
                fn();
                canTrigger = true
            }, ms);
        }
        canTrigger = false;
    }
}
```
