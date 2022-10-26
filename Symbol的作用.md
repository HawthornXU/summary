# Symbol
Symbol 是 ES6新增的内置对象 用作属性名， 永远不会重复，

## Symbol.iterator
Symbol.iterator 是 Symbol对象的内置属性， 如果一个值有Symbol.iterator值 那么这个值就是可以被 for...of... 遍历的
例如 list[Symbol.iterator] 不为undefined 那么就可以遍历 反之 obj[Symbol.iterator] === undefined 那么这个obj 就不可遍历
