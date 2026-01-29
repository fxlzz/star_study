## sort 的细节：

```js
const arr = [1, 11, 2, 3, 4, 124];
arr.sort((a, b) => a - b);
// 这里的sort里面的compare函数接收的两个参数有说法：
// a：第一次遍历是 -> 11
// b：第一次遍历是 -> 1
// ----------------------
// a：第二次遍历是 -> 2
// b：第二次遍历是 -> 11
// 是反过来的
```

判断升序还是降序：

- a 和 b 是 compare 函数的两个参数
- 如果函数的返回值是 < 0 则 a 会排在 b 的前面
- 如果函数的返回值是 > 0 则 b 会排在 a 的前面
- 如果是 NaN 或 0 则不改变顺序

```js
arr.sort((a, b) => b - a) // 降序
arr.sort((a, b) => a - b) // 升序
```

## 逻辑运算符的返回结果

### `||`

- 如果前面判定为 true，那么返回值就是前面的值，例如：

```js
console.log(1 || 2); // 1
```

- 如果前面的布尔判定结果是 false，那么返回值是后面的值（一般用这个性质，给一个变量默认值），例如：

```js
console.log(0 || 3); // 3 
```

### `&&`

- 判定为 true 那么返回后面的值，例如：

```js
console.log(2 && 1); // 1
```

- 判定为 false 那么返回错误的值，例如：

```js
console.log(0 && 1); // 0
```

## 判断某个变量是否能被迭代

判断这个变量是否有 `[Symbol.iterator]`，有这个唯一标识，那么说明该变量可以被迭代

```js
const map = new Map(); // 可迭代
const obj = {}; // 不可迭代

console.log(typeof map[Symbol.iterator]); // function
console.log(typeof obj[Symbol.iterator]); // undefined
```

JavaScript 中有许多内置的可迭代对象，如：

- 数组（`Array`）
- 字符串（`String`）
- `Map`
- `Set`
- `arguments` 对象（函数参数）
- DOM 集合（如 `NodeList`）