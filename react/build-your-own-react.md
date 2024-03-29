# 从 jsx hello world! 开始

```typescript jsx
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDom.render(element, container);
```

# jsx 转换成 js代码

babel 帮我们把jsx转换成这样的js代码

```typescript jsx
const element = React.createElement("h1", {title: "foo"}, "Hello")
const container = document.getElementById("root")
ReactDom.render(element, container);
```

## createElement

而`createElement`方法实际上返回的是一个对象：

```typescript jsx
const element = {
    type: 'h1',
    props: {
        title: "foo",
        children: "Hello",
    },
}
const container = document.getElementById("root")
ReactDom.render(element, container);
```

## render

```typescript jsx
const element = {
    type: 'h1',
    props: {
        title: "foo",
        children: "Hello",
    },
}
const container = document.getElementById("root")

const node = document.createElement(element.type);
node["title"] = element.props.title;

const text = document.createTextNode("")
text["nodeValue"] = element.props.children

node.appendChild(text);
container.appendChild(node);
```

实际上一个hello world `react`和`babel`帮我们了这些改变成js代码的工作。

# 创建自己写的react

先从 createElement 开始

```typescript jsx
function createElement(type, props, ...children) {
    return {
        type,
        props: {
            ...props,
            children: children.map(child =>
                typeof child === "object"
                    ? child
                    : createTextElement(child)
            )
        }
    }
}

function createTextElement(text) {
    return {
        type: "TEXT_ELEMENT",
        props: {
            nodeValue: text,
            children: []
        }
    }
}

function render(element, container) {
    const dom = element.type === "TEXT_ELEMENT"
        ? document.createTextNode("")
        : document.createElement(element, type);

    const isProperty = key => key !== "children";

    Object.keys(element.props)
        .filter(isProperty)
        .forEach(name => {
            dom[name] = element.props[name]
        })

    element.props.children.forEach(child => render(child, dom));

    container.appendChild(dom);
}

const MyReact = {createElement, render}

// const element = MyReact.createElement(
//     "div",
//     {title: "foo"}, 
//     MyReact.createElement("a", null, "bar"),
//     MyReact.createElement("b")
// )

// 告诉Babel这个是jsx 实际上就是上面这段代码↑
/** @jsx MyReact.createElement*/
const element = (
    <div id="foo">
        <a>bar</a>
        <b/>
    </div>
)

const container = document.getElementById("root")
MyReact.render(element, container);
```

以上 就用MyReact完成了第一章所描述的行为

# 解决渲染dom耗时太长的问题

render函数去渲染dom会耗时特别久，那么就引入浏览器任务切片

```typescript jsx
    function createElement(type, props, ...children) {
        return {
            type,
            props: {
                ...props,
                children: children.map(child =>
                    typeof child === "object"
                        ? child
                        : createTextElement(child)
                )
            }
        }
    }
    
    function createTextElement(text) {
        return {
            type: "TEXT_ELEMENT",
            props: {
                nodeValue: text,
                children: []
            }
        }
    }
    
    function createDom(fiber) {
        const dom = fiber.type === "TEXT_ELEMENT"
            ? document.createTextNode("")
            : document.createElement(fiber, type);
    
        const isProperty = key => key !== "children";
    
        Object.keys(fiber.props)
            .filter(isProperty)
            .forEach(name => {
                dom[name] = fiber.props[name]
            })
        return dom;
    }
    
    function commitRoot() {
        // add nodes to dom
        commitWork(wipRoot.child)
        // currentRoot = wipRoot
        wipRoot = null
    }
    
    function commitWork(fiber) {
        if (!fiber) {
            return
        }
        const domParent = fiber.parent.dom
        domParent.append(fiber.dom)
        commitWork(fiber.child)
        commitWork(fiber.sibling)
    }
    
    function render(element, container) {
        wipRoot = {
            dom: container,
            props: {
                children: [element]
            },
            // alternate: currentRoot
        }
        nextUnitOfWork = wipRoot;
    }
    
    let nextUnitOfWork = null
    // let currentRoot = null
    let wipRoot = null
    
    function workLoop(deadline) {
        let shouldYield = false;
        while (nextUnitOfWork && !shouldYield) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
            shouldYield = deadline.timeRemaining() < 1
        }
        
        if (!nextUnitOfWork && wipRoot) {
            commitRoot();
        }
        
        requestIdleCallback(workLoop);
    }
    
    requestIdleCallback(workLoop)
    
    // 注意区分三个不同的节点实体，
    // element（通过 createElement创建的 react element），
    // DOM node（最终生成对应的 DOM 节点），
    // fiber node（从element 到 DOM 节点的中间产物，用于时间切片）
    function performUnitOfWork(fiber) {
        // add dom node
        if (!fiber.dom) {
            fiber.dom = createDom(fiber);
        }
        // create new fibers
        const elements = fiber.props.children
        let prevSibling = null;
    
        for (let index = 0; index < elements.length; index++) {
            const element = elements[index];
            const newFiber = {
                type: element.type,
                props: element.props,
                parent: fiber,
                dom: null
            }
            if (index === 0) {
                fiber.child = newFiber;
            } else {
                prevSibling.sibling = newFiber;
            }
            prevSibling = newFiber;
        }
        // return next unit of work
        if (fiber.child) {
            return fiber.child;
        }
        let nextFiber = fiber;
        while (nextFiber) {
            if (nextFiber.sibling) {
                return nextFiber.sibling;
            }
            nextFiber = nextFiber.parent
        }
    
    }
    
    const MyReact = {createElement, render}
    
    // const element = MyReact.createElement(
    //     "div",
    //     {title: "foo"}, 
    //     MyReact.createElement("a", null, "bar"),
    //     MyReact.createElement("b")
    // )
    
    // 告诉Babel这个是jsx 实际上就是上面这段代码↑
    /** @jsx MyReact.createElement*/
    const element = (
        <div id="foo">
            <a>bar</a>
            <b/>
        </div>
    )
    
    const container = document.getElementById("root")
    MyReact.render(element, container);
```

# 更新删除dom阶段
需要比较 render阶段接收的element生成的fiber 和 上次提交的fiber树
```typescript jsx
    function createElement(type, props, ...children) {
        return {
            type,
            props: {
                ...props,
                children: children.map(child =>
                    typeof child === "object"
                        ? child
                        : createTextElement(child)
                )
            }
        }
    }
    
    function createTextElement(text) {
        return {
            type: "TEXT_ELEMENT",
            props: {
                nodeValue: text,
                children: []
            }
        }
    }
    
    function createDom(fiber) {
        const dom = fiber.type === "TEXT_ELEMENT"
            ? document.createTextNode("")
            : document.createElement(fiber, type);
        updateDom(dom, {}, fiber.props);
        return dom;
    }
    
    const isEvent = key => key.startsWith("on")
    const isProperty = key => key !== "children" && !isEvent(key)
    const isNew = (prev, next) => key => prev[key] !== next[key]
    const isGone = (prev, next) => key => !(key in next)
    function updateDom(dom, prevProps, nextProps) {
        //Remove old or changed event listeners
        Object.keys(prevProps)
            .filter(isEvent)
            .filter(
                key =>
                    !(key in nextProps) ||
                    isNew(prevProps, nextProps)(key)
            )
            .forEach(name => {
                const eventType = name
                    .toLowerCase()
                    .substring(2)
                dom.removeEventListener(
                    eventType,
                    prevProps[name]
                )
            })
        // remove old properties
        Object.keys(prevProps)
            .filter(isProperty)
            .filter(isGone(prevProps, nextProps))
            .forEach(name => {
                dom[name] = "";
            })
        // Set new or changed properties
        Object.keys(nextProps)
            .filter(isProperty)
            .filter(isNew(prevProps, nextProps))
            .forEach(name => {
                dom[name] = nextProps[name]
            })
        // Add event listeners
        Object.keys(nextProps)
            .filter(isEvent)
            .filter(isNew(prevProps, nextProps))
            .forEach(name => {
                const eventType = name
                    .toLowerCase()
                    .substring(2)
                dom.addEventListener(
                    eventType,
                    nextProps[name]
                )
            })
    }
    
    function commitRoot() {
        deletions.forEach(commitWork)
        // add nodes to dom
        commitWork(wipRoot.child)
        currentRoot = wipRoot
        wipRoot = null
    }
    
    function commitWork(fiber) {
        if (!fiber) {
            return
        }
        const domParent = fiber.parent.dom
        if (fiber.effectTag === "PLACEMENT" && fiber.dom !== null) {
            domParent.append(fiber.dom)
        } else if (fiber.effectTag === "DELETION") {
            domParent.removeChild(fiber.dom)
        } else if (fiber.effectTag === "UPDATE" && fiber.dom != null) {
            updateDom(fiber.dom, fiber.alternate.props, fiber.props)
        }
        commitWork(fiber.child)
        commitWork(fiber.sibling)
    }
    
    function render(element, container) {
        wipRoot = {
            dom: container,
            props: {
                children: [element]
            },
            alternate: currentRoot
        }
        nextUnitOfWork = wipRoot;
        deletions = [];
    }
    
    let nextUnitOfWork = null
    let currentRoot = null
    let wipRoot = null
    let deletions = null
    
    function workLoop(deadline) {
        let shouldYield = false;
        while (nextUnitOfWork && !shouldYield) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork)
            shouldYield = deadline.timeRemaining() < 1
        }
        
        if (!nextUnitOfWork && wipRoot) {
            commitRoot();
        }
        
        requestIdleCallback(workLoop);
    }
    
    requestIdleCallback(workLoop)
    
    function performUnitOfWork(fiber) {
        // add dom node
        if (!fiber.dom) {
            fiber.dom = createDom(fiber);
        }
        // create new fibers
        const elements = fiber.props.children
        reconcileChildren(fiber, elements)

        // return next unit of work
        if (fiber.child) {
            return fiber.child;
        }
        let nextFiber = fiber;
        while (nextFiber) {
            if (nextFiber.sibling) {
                return nextFiber.sibling;
            }
            nextFiber = nextFiber.parent
        }
    
    }
    
    // reconcile 调解
    function reconcileChildren(wipFiber, elements) {
        let oldFiber = wipFiber.alternate && wipFiber.alternate.child
        let prevSibling = null;

        for (let index = 0; index < elements.length || oldFiber != null; index++) {
            const element = elements[index];
            const newFiber = null
            const sameType = oldFiber && element && element.type == oldFiber.type
            
            if (sameType) {
                newFiber = {
                    type: oldFiber.type,
                    props: element.props,
                    dom: oldFiber.dom,
                    parent: wipFiber,
                    alternate: oldFiber,
                    effectTag: "UPDATE"
                }
            }
            if (element && !sameType) {
                newFiber = {
                    type: element.type,
                    props: element.props,
                    dom: null,
                    parent: wipFiber,
                    alternate: null,
                    effectTag: "PLACEMENT"
                }
            }
            if (oldFiber && !sameType) {
                oldFiber.effectTag = "DELETION"
                deletions.push(oldFiber)
            }
            if (index === 0) {
                wipFiber.child = newFiber;
            } else {
                prevSibling.sibling = newFiber;
            }
            prevSibling = newFiber;
        }
    }
    
    const MyReact = {createElement, render}
    
    // const element = MyReact.createElement(
    //     "div",
    //     {title: "foo"}, 
    //     MyReact.createElement("a", null, "bar"),
    //     MyReact.createElement("b")
    // )
    
    // 告诉Babel这个是jsx 实际上就是上面这段代码↑
    /** @jsx MyReact.createElement*/
    const element = (
        <div id="foo">
            <a>bar</a>
            <b/>
        </div>
    )
    
    const container = document.getElementById("root")
    MyReact.render(element, container);
```
