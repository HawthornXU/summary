# 防止被网站被iframe嵌套



## 在head中设置meta属性

`SAMEORIGIN：`表示该页面可以在相同域名页面的 frame 中展示。

```html
<meta http-equiv="X-Frame-Options" content="SAMEORIGIN">
```

`DENY：`表示该页面不允许在 frame 中展示，即便是在相同域名的页面中嵌套也不允许。

```html
<meta http-equiv="X-Frame-Options" content="DENY"> 
```

`ALLOW-FROM uri`表示该页面可以在指定来源的 frame 中展示。

```html
指定具体网站：
<meta http-equiv="X-Frame-Options" content="ALLOW-FROM http://xxx.xxx.com">
```

## 使用JS代码控制

核心代码：` window.top.location===window.self.location`

真为没有被嵌套、假为被嵌套了

## 服务端设置拦截

本质还是在响应头中增加`X-Frame-Options`响应头 添加`SAMEORIGIN`等属性

[防止自己的网页放被在别人iframe嵌套](https://www.cnblogs.com/xiejn/p/11977603.html)