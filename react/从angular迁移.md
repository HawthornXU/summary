# 父组件传值到子组件

```jsx
// 父
const foo = {bar:'baz'}
<parent>
    <child value={foo}></child>
</parent>
// 子
<child>{this.props.bar}</child>
```

