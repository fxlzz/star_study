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