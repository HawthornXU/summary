https://github.com/type-challenges/type-challenges/blob/main/README.zh-CN.md
# Omit
omit 可以 去掉一个类或接口不需要的类型 由此可以重写属性类型

```typescript
interface People {
    id: string;
    name: string;
    password: string;
    createdAt: Date;
    updatedAt: Date;
}

interface User extends Omit<People, "createdAt" | 'updatedAt'> {
    createdAt: string;
    updatedAt: string;
}
```
# Pick
pick 和 omit 相反 是挑选需要的属性值
