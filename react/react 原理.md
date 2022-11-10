https://pomb.us/build-your-own-react/

# 工作流程
```typescript
const element = <h1 title="foo">Hello</h1>
const container = document.getElementById("root")
ReactDOM.render(element, container)
```
以最基础的react应用为例， 第一行JSX是借助Babel实现的语法糖， 在react教程中就提到实际上经过Babel的代码为：

```typescript
const element = React.createElement(
  "h1",
  { title: "foo" },
  "Hello"
)
```
createElement创建的是一颗dom树，也就是fiber树, 就需要程序递归的把树构建出来.

```typescript
const element = {
    type: "h1",
    props: {
        title: "foo",
        children: "Hello",
    },
}
```
由于这个树会十分巨大，构建对象的时候会十分耗时， 所以会对任务进行切片， react 做任务切片的模块叫scheduler。

具体的切片方式是从fiber根部开始，以fiber节点为单位，依次对第一个子fiber进行构建, 直到没有子节点，去构建相邻的兄弟节点及其子， 如果没有兄弟节点， 就构建父节点的兄弟节点。按照这个流程会返回到fiber根部。那么就可以判断fiber树构建完成。

也就是深度优先遍历，

最后就到了render环节, `ReactDOM.render(element, container)`， 会遍历 fiber 新增dom节点添加到页面上

ReactDOM.render 换成其他库 就可以渲染其他应用的界面， 例如ReactNative。
