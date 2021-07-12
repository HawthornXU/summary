# angular玩家react过渡总结手册

## 生命周期

### NgInit

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

