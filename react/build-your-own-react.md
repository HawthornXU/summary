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

const MyReact = {createElement}

const element = MyReact.createElement(
    "div",
    {title: "foo"}, 
    MyReact.createElement("a", null, "bar"),
    MyReact.createElement("b")
)
const container = document.getElementById("root")
ReactDom.render(element, container);
```
