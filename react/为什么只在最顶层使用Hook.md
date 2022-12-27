每次调用hooks 都会生成hook对象，其结构如下

```typescript
export type Hook = {
memoizedState: any,
baseState: any,
baseQueue: Update<any, any> | null,
queue: UpdateQueue<any, any> | null,
next: Hook | null
};
```

着重关注memoizedState和next属性，不同 hooks 的 hook 对象的memoizedState保存着不同对象

* useState：hook.memoizedState =  state;

* useEffect ：hook.memoizedState = effect

* useMemo：hook.memoizedState = [nextValue, nextDeps];

* useCallback：hook.memoizedState = [callback, nextDeps];

* useRef：hook.memoizedState = ref;

一般我们会在函数组件中多次使用hooks，这会产生多个 hook 对象，这些对象通过next属性连接形成链表，连接是在挂载时进行的，主要函数是 mountWorkInProgressHook。

```typescript
function mountWorkInProgressHook() {
  const hook = {
    memoizedState: null,
    baseState: null,
    baseQueue: null,
    queue: null,
    next: null,
  };
  if (workInProgressHook === null) {
    // This is the first hook in the list
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // Append to the end of the list
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```
挂载时，hooks 都会调用该函数生成 hook 对象并连接。如果 workInProgressHook === null，说明是hook链中的第一个，将其赋值给currentlyRenderingFiber.memoizedState并更新workInProgressHook，这说明函数组件的Fiber的memoizedState保存着第一个hook的引用。当workInProgressHook !== null，则将新生成的 hook 赋值给上个hook的next属性并更新workInProgressHook。

更新时，hooks 都会调用 updateWorkInProgressHook 函数生成新的hook对象并连接形成hook链以供下次更新时使用。

正确使用hooks时，遍历hook链，复用hook上次更新的hook，没有问题，但如果某个hooks的调用与否是有条件的，那么情况就不同了

**不要在循环，条件或嵌套函数中调用 Hook的原因是如果hooks的执行顺序发生变化会导致 hooks 中使用错误的 hook对象。**


https://juejin.cn/post/7020811068955951135
