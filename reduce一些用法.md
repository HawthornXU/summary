**1、 使用 reduce 替代 map + filter**

设想你有这么个需求：要把数组中的值进行计算后再滤掉一些值，然后输出新数组。很显然我们一般使用 map 和 filter 方法组合来达到这个目的，但这也意味着你需要迭代这个数组两次。

来看看我们如何使用 reduce 只迭代数组一次，来完成同样的结果。下面这个例子我们需要把数组中的值乘 2 ，并返回大于 50 的值：

const numbers = [10, 20, 30, 40]; const doubledOver50 = numbers.reduce((finalList, num) => {    num = num * 2; //double each number (i.e. map)    //filter number > 50  if (num > 50) {    finalList.push(num);  }  return finalList; }, []); doubledOver50; // [60, 80]

**2、 使用 reduce 检测括号是否对齐封闭**

下面这个例子我们用 reduce 来检测一段 string 中的括号是否前后对应封闭。

思路是定义一个名为 counter 的变量，它的初始值为 0 ，然后迭代字符串，迭代过程中碰到(就加 1，碰到)就减 1，如果括号前后对应的话，最终couter的值会是 0。

//Returns 0 if balanced. const isParensBalanced = (str) => {  return str.split('').reduce((counter, char) => {    if(counter < 0) { //matched ")" before "("      return counter;    } else if(char === '(') {      return ++counter;    } else if(char === ')') {      return --counter;    }  else { //matched some other char      return counter;    }      }, 0); //<-- starting value of the counter } isParensBalanced('(())') // 0 <-- balanced isParensBalanced('(asdfds)') //0 <-- balanced isParensBalanced('(()') // 1 <-- not balanced isParensBalanced(')(') // -1 <-- not balanced

**3、 使用 reduce 计算数组中的重复项**

如果你想计算数组中的每个值有多少重复值，reducer 也可以快速帮到你。下面的例子我们计算数组中每个值的重复数量，并输出一个对象来展示：

var cars = ['BMW','Benz', 'Benz', 'Tesla', 'BMW', 'Toyota']; var carsObj = cars.reduce(function (obj, name) {    obj[name] = obj[name] ? ++obj[name] : 1;  return obj; }, {}); carsObj; // => { BMW: 2, Benz: 2, Tesla: 1, Toyota: 1 }
