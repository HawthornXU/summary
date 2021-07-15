# angular玩家react过渡总结手册

## 生命周期

https://www.jianshu.com/p/b331d0e4b398 

### componentDidMount

感觉类似angular `viewInit` 组件第一次被渲染时调用 ，称为挂载、mount

```jsx
import React, { useEffect } from 'react';
const reactComponent = (props) => {
    useEffect( () => {
	// do init
	}, []);
}
//或者
class reactComponent extends React.Component {
  componentDidMount() {
  }
}
```

### componentWillUnmount

和ngdestory类似  react组件被卸载时调用

```
  componentWillUnmount() {  }
```

## 父组件传值到子组件

```jsx
// 父
const foo = {bar:'baz'}
<parent>
    <child value={foo}></child>
</parent>
// 子
<child>{this.props.bar}</child>
```

## [*DomSanitizer*](https://angular.cn/api/platform-browser/DomSanitizer#domsanitizer)

react仅提供了dangerouslySetInnerHTML属性来警示传入的字符串 没有做DomSanitizer的工作，需要依赖第三方插件purify后再传入

https://www.npmjs.com/package/sanitize-html

## ngStyle

```jsx
const mystyle= {
    color:#fff;
}
<react style={mystyle}></react>
```

## *NgIf

```tsx
{booleanValue && <react>booleanValue等于真</react>}
```

## 双向绑定

State Hook 或 state 实现双向绑定

### State Hook

```jsx
const [count, setCount] = useState(0);
//useState() 初始化值
setCount(1)
//setCount 方法 刷新页面值
<react>{count}</react>
```

### State

构造函数是唯一可以给 `this.state` 赋值的地方

```jsx
  constructor(props) {
    super(props);
    this.state = {value: 0};
  }
    changeValue(arg){
      this.setState({
        value: 1
       });
    }
    render() {
        return(
        <div>{this.state.value}</div>
        )
    }
```

#### state 的更新会自动合并

```jsx
  constructor(props) {
    super(props);
    this.state = {value1: 0, value2:1};
    this.setState({value1:1});
    // this.state.vaule2 还是原来的
  }
  
```

