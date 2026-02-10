> 简单记录一下

# 滑动窗口
> 滑动窗口是处理"子数组/子串问题"的高效算法模式。当问题涉及"连续元素的某种性质"（如最大/最小、满足条件等）时，滑动窗口通常可以将 O(n²) 的暴力解优化到 O(n)
> 用途：
> 需要知道**某段范围内的最优解**，比如，用户哪个时间段是最活跃的，股票那段时间涨幅最大


双指针的做法：`O(n*k)`
```js
const func = (users, k) => {
    if(users.length < k) return -1;

    let start = 0, end = k - start - 1;
    let sum = 0;
    while(end < users.length) {
        let temp = 0;
        for(let i = start; i <= end; i++) {
            temp += users[i];
        }
        sum = Math.max(temp, sum);    
        start++;
        end++;
    }

    return sum;
}
```

滑动窗口：`O(n)`
```js
const users = [100, 200, 300, 400, 500]
const func = (users, k) => {
    if (users.length < k) return -1;
    
    // 计算初始窗口和
    let currentSum = 0;
    for (let i = 0; i < k; i++) {
        currentSum += users[i];
    }
    
    let maxSum = currentSum;
    // 滑动窗口：每次减去最左边，加上新元素
    for (let i = k; i < users.length; i++) {
        currentSum = currentSum - users[i - k] + users[i];
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
console.log(func(users, 3))
```

## 定长滑动窗口

其中最关键的一点就是，需要在脑子里模拟一遍算法的运行过程
左端点： `right - left + 1` 的位置

```js
/**
 * 定长滑动窗口 - 模板
 * @param {*} s 模板串
 * @param {*} k 定长
 */
const func = (s, k) => {
  let left = 0, right = 0;
  let ans = 0, count = 0;

  while (right < s.length) {
    // 1. 右端点进入窗口 -> 需满足的条件等
    if (xxx) count++;

    // 2. 更新最值
    ans = Math.max(count, ans);

    // 3. 左端点出窗口
    if (right - left + 1 === k) { // 说明左右端口已经到达了定长的长度， 可以滑动窗口了
      if (xxx) count--;
      left++;
    }
    right++;
  }
  return ans;
};
```

## 不定长滑动窗口

## 单调队列
> 特殊的数据结构，基于双端队列实现，用于维护一个单调递增或单调递减的队列

核心思想：
- 队列中的元素需要满足某种单调性（递增/递减）
- 每次插入新元素时，会从队尾删除不满足单调性的元素
- 队首始终是当前窗口的“最优解”




# 前缀和
用于快速处理数组任意区间内的和

`prefix[j]` 表示从数组第 0 项到第 j 项的和

求出任意区间的和：
`sum = prefix[j] - prefix[i - 1]`
例如：
`[1, 3] -> prefix[3] - prefix[0]`
```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
var subarraySum = function (nums, k) {
  let ans = 0;
  let sum = 0;
  const map = new Map();

  map.set(0, 1);
  for (let i = 0; i < nums.length; i++) {
    sum += nums[i];
    const n = sum - k;
    if (map.has(n)) {
      ans += map.get(n);
    }
    map.set(sum, (map.get(sum) ?? 0) + 1);
  }
  return ans;
};
```