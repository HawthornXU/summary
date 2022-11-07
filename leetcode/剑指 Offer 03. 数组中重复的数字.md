https://leetcode.cn/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof/
### map存储找重复
需要m个空间储存 好处是不改变原数组
```js
/**
 * @param {number[]} nums
 * @return {number}
 */
var findRepeatNumber = function(nums) {
    const counter = {};
    for(let number of nums) {
        if (!counter[number]) {
            counter[number] = 1;
        } else {
            return number;
        }
    }
    return null;
};
```

### 数字 n 放置到 n 位置上， 原地存储找重
不需要另外使用内存空间， 改变原数组
````js
var findRepeatNumber = function(nums) {
    let i = 0;
    while (i < nums.length) {
        if (nums[i] === i) {
            i++;
            continue;
        } else if(nums[nums[i]] === nums[i]) {
            return nums[i];
        } else {
            [nums[nums[i]], nums[i]] = [nums[i], nums[nums[i]]];
        }
    }
};
````
