# 一、map 和 forEach 区别

+ **返回值**不同
	+ map -> 返回新数组
	+ forEach -> 返回 undefined
+ **用途**不同
	+ map -> 用于数据处理（修改、过滤、重组元素）
	+ forEach -> 用于遍历数组
+ 是否**支持链式**调用
	+ map -> 支持链式调用 
	+ forEach -> 不支持

# 二、防抖节流代码实现

防抖（Debounce）和节流（Throttle）是处理高频事件（如 `resize`、`scroll`、`input`、`mousemove`）的常用技术，用于优化性能。以下是它们的核心实现及应用场景：

### 1、**防抖（Debounce）**

#### **原理**

事件触发后，**延迟一段时间再执行函数**，若期间再次触发则重新计时，确保只在最后一次触发后执行一次。  
**适用场景**：搜索框实时搜索、窗口Resize、表单验证等（只需处理“最终状态”）。

#### **基础实现（简单版）**

```js
function debounce(func, delay) {
  let timer = null; // 定时器
  return function (...args) {
    if (timer) clearTimeout(timer); // 清除上一次定时器
    timer = setTimeout(() => {
      func.apply(this, args); // 延迟执行函数
    }, delay);
  };
}
```

### 2、**节流（Throttle）**

#### **原理**

事件触发后，**在指定时间内只执行一次函数**，忽略中间的高频触发。  
**适用场景**：滚动加载、高频点击、动画更新等（需控制执行频率）。

#### **定时器版（延迟执行）**

```js
function throttle(func, delay) {
  let timer = null; // 定时器
  return function (...args) {
    if (!timer) {
      timer = setTimeout(() => {
        func.apply(this, args); // 延迟执行函数
        timer = null; // 重置定时器
      }, delay);
    }
  };
}
```

# 三、提取 URL 中参数的代码

在 JavaScript 中提取 URL 中的参数，可根据环境（浏览器/Node.js）和需求选择不同方法。以下是常见的实现方式：

### 1、**使用浏览器原生** `URLSearchParams`**（推荐，现代浏览器支持）**

#### 1. **解析当前页面 URL 的参数**

```js
// 获取当前页面的 URL 参数
const url = new URL(window.location.href);
const params = new URLSearchParams(url.search); // 或直接 url.searchParams

// 转为对象（单值，重复参数取最后一个）
const paramObject = Object.fromEntries(params);
console.log(paramObject); // { name: 'Alice', age: '30' }

// 转为数组（保留重复参数）
const paramArray = Array.from(params.entries());
console.log(paramArray); // [['name', 'Alice'], ['age', '30']]

// 获取单个参数（如 'name'）
const name = params.get('name'); // 'Alice'

// 获取多个同名参数（如 ?tag=1&tag=2，返回数组）
const tags = params.getAll('tag'); // ['1', '2']
```

#### 2. **解析自定义 URL 的参数**

```js
const urlString = 'http://example.com/search?keyword=js&page=2';
const url = new URL(urlString);
const params = url.searchParams;
```

### 2、**手动解析字符串（兼容性强，适合旧环境）**

#### 1. **提取所有参数并转为对象（处理单值/重复参数）**

```js
function getUrlParams(url) {
  // 提取查询字符串（去除 '?'）
  const search = url.split('?')[1] || '';
  // 分割参数对
  const params = new URLSearchParams(search); // 或手动分割：search.split('&').map(pair => pair.split('='))

  // 转为对象（单值，重复参数取最后一个）
  const paramObject = {};
  for (const [key, value] of params.entries()) {
    paramObject[key] = decodeURIComponent(value); // 解码参数值（如 %20 转空格）
  }

  // 或保留重复参数为数组（更准确）
  const paramArray = {};
  for (const [key, value] of params.entries()) {
    if (!paramArray[key]) {
      paramArray[key] = [];
    }
    paramArray[key].push(decodeURIComponent(value));
  }

  return paramObject; // 或返回 paramArray
}

// 示例
const url = 'http://example.com?name=Alice&age=30&tag=js&tag=web';
console.log(getUrlParams(url)); // { name: 'Alice', age: '30', tag: 'web' }（单值，取最后一个）
console.log(getUrlParamsWithArray(url)); // { name: ['Alice'], age: ['30'], tag: ['js', 'web'] }
```

#### 2. **核心步骤解析**

1. **分割 URL**：通过 `?` 分离出查询字符串（`search`）。
2. **处理编码**：使用 `decodeURIComponent` 解码参数值（如 `%20` 转空格，`%3D` 转 `=`）。
3. **处理重复参数**：同名参数可能有多个（如 `?tag=1&tag=2`），需用 `getAll` 或手动收集为数组。

### 3、**Node.js 环境（使用** `url` **模块）**

```js
const url = require('url');

// 解析 URL 字符串
const parsedUrl = url.parse('http://example.com?name=Alice&age=30', true); // 第二个参数 true 表示解析参数
const params = parsedUrl.query;

console.log(params); // { name: 'Alice', age: '30' }（单值，重复参数取最后一个）
console.log(params.name); // 'Alice'
```

### 4、**完整工具函数（支持单值/数组，兼容新旧环境）**

```js
function parseUrlParams(url) {
  // 处理输入：默认解析当前页面 URL
  const currentUrl = url || window.location.href;
  const search = currentUrl.split('?')[1] || '';
  const params = new URLSearchParams(search);
  const result = {};

  // 收集所有参数，同名参数存为数组
  for (const [key, value] of params.entries()) {
    if (result[key]) {
      if (!Array.isArray(result[key])) {
        result[key] = [result[key]]; // 单值转数组
      }
      result[key].push(value);
    } else {
      result[key] = value;
    }
  }

  // 解码所有参数值
  for (const key in result) {
    if (Array.isArray(result[key])) {
      result[key] = result[key].map(val => decodeURIComponent(val));
    } else {
      result[key] = decodeURIComponent(result[key]);
    }
  }

  return result;
}

// 示例
const url = 'http://example.com?name=Alice&age=30&tag=js&tag=web';
console.log(parseUrlParams(url)); 
// { name: 'Alice', age: '30', tag: ['js', 'web'] }
```

### 总结

- **首选** `URLSearchParams`：简洁高效，支持现代浏览器和 Node.js（v10+），推荐用于新项目。
- **手动解析**：适合兼容旧环境（如 IE），需注意编码和解构逻辑。
- **Node.js 专用**：直接使用 `url` 模块的 `query` 属性，方便快捷。

根据场景选择合适的方法，核心是正确处理参数编码、重复参数和边界条件（如无参数、空值）。

# 四、怎么理解 ES6 中的 promise 的，使用场景有哪些？

promise 是 ES6 专门处理异步请求的函数，有效的缓解了回调地狱的问题

## 三种状态

- pending 等待状态（创建即为这个状态）
- fulfilled 完成状态
- rejected 失败状态

【注意】：promise 状态一旦确定，不会改变

## 核心特性

1. **链式调用**：借助`.then()`和`.catch()`方法可以实现链式调用，有效避免回调地狱。
2. **错误冒泡**：在 Promise 链中，错误会一直向后传递，直到遇到`.catch()`。
3. **状态不可变**：Promise 的状态一旦确定，就不会再改变。

## 常用场景

1. 调用异步 API `.then()`
2. 并行处理异步 API 使用 `promise.all([p1, p2])`
3. 串行处理异步 API 使用 `链式调用 .then().then()`

【总结】

1. **始终处理错误**：在 Promise 链的末尾添加`.catch()`，防止未捕获的异常。
2. **避免嵌套 Promise**：利用链式调用保持代码的扁平化。
3. **合理使用辅助方法**：

- `Promise.all`：并行处理多个 Promise。
- `Promise.race`：多个 Promise 竞争，取**最先**完成的。
- `Promise.allSettled`：获取**所有** Promise 的结果，无论成功还是失败。
- `Promise.any`：获取**第一个**成功的 Promise 的结果。

# 五、promise.catch()后面的 then()还会执行吗？

分两种情况，

如果 catch() 正常返回，未抛出错误，那么此时 catch()处理完之后， promise 的状态变成 fulfilled 会被 then()继续处理

如果 catch() 处理过程中，抛出了错误，那么 then() 不会执行，会交给下一个 catch()执行

# 六、给一个 dom 同时绑定两个点击事件，一个用捕获，一个用冒泡，会执行几次事件，会先执行那个事件？捕获还是冒泡？

1. **执行次数**：无论事件传播到哪个 DOM 元素，**每个事件监听器都会触发一次**（前提是没有被`stopPropagation()`阻止）。
2. **执行顺序**：

- 事件首先进入**捕获阶段**（从文档根节点向下传播到目标元素）。
- 然后到达**目标元素**（此时捕获和冒泡阶段的事件按绑定顺序触发，标准未强制规定，但多数浏览器按注册顺序执行）。
- 最后进入**冒泡阶段**（从目标元素向上传播到文档根节点）。

对于**同一元素**，捕获事件总是在冒泡事件之前触发（无论绑定顺序如何）

所以，会执行两次，先执行捕获在执行冒泡。

# 七、深拷贝和浅拷贝有什么区别？怎么实现深拷贝？

简单来说，

**浅拷贝：新旧的内存地址是****共享的**

1. 直接赋值

```js
let a = [1,2,3];
let b = a; // 此时 b 和 a 的内存空间是共享的
```

2. `Object.assgin()`

```js
// 这个函数本来是用来合并对象的 
// 基本使用 Object.assgin(target: {}, source)
Object.assgin({name: "zhangsan"}, {age: 18}) // {name: "zhangsan", age: 18}

// 实现浅拷贝
let obj = {
  name: "zhangsan",
  age: 18,
  info: {
    a: 1,
  },
};
const o = Object.assign({}, obj); // o的info和obj的info内存地址是共享的
```

3. `slice() 和 concat()`

```js
const arr = [1, 2, 3, 4, {name: "jack"}];
const arr1 = arr.slice();
const arr2 = arr.concat();
arr[4].name = "tom";
// arr、arr1、arr2最后的对象都是共享的
```

4. 扩展运算符

```js
// 这个其实也差不多，复制引用数据类型就会共享内存空间
const arr = [1, 2, 3, 4, { name: "jack" }];
const arr1 = [...arr];

arr[4].name = "tmo";
console.log(arr1); // 最后的对象还是同一个空间
```

**深拷贝：新旧的内存地址是****隔离****开的**

1. `JSON.parse(JSON.stringify())`

JSON.stringify() 拷贝

-> 会丢失函数、值为undefined、不支持Symbol、Map、Set、WeakMap等ES6+数据结构

-> Date对象会转化成ISO字符串，不在是Date的实例了

```js
const a = [1, 2, 3, [4, 5], { name: "jack", age: 18 }];
const b = JSON.parse(JSON.stringify(a)); // 拷贝这些是没问题的
// 但是加上函数 es6的数据类型 就不适用了
```

2. 递归实现深拷贝`deepClone(oldData, map = new Weakmap())`

```js
function deepClone(oldData, map = new WeakMap()) {
  if (oldData === null || typeof oldData !== "object") {
    // 基本数据类型
    return oldData;
  }

  // 处理循环引用 -> 如果有引用过，那么直接返回，防止栈溢出
  if (map.has(oldData)) {
    return map.get(oldData);
  }

  // 处理其他数据 Date复制完后还是Date的实例对象
  if (oldData instanceof Date) return new Date(oldData);
  if (oldData instanceof RegExp) return new RegExp(oldData);
  if (oldData instanceof Map) {
    const result = new Map();
    map.set(oldData, result);
    oldData.forEach((v, k) => {
      result.set(deepClone(k, map), deepClone(v, map));
    });
    return result;
  }

  if (oldData instanceof Set) {
    const result = new Set();
    map.set(oldData, result);
    oldData.forEach((v) => {
      result.add(deepClone(v, map));
    });
    return result;
  }

  // 处理某个引用对象 
  const res = Array.isArray(oldData) ? [] : {};
  map.set(oldData, res);

  // 拷贝所有自有属性，包括不可枚举和 symbol
  Reflect.ownKeys(oldData).forEach((key) => {
    res[key] = deepClone(oldData[key], map);
  });
  return res;
}
```

# 八、ES6 有哪些新特性？

## 1、变量与作用域

- let 与 const
- 解构

## 2、函数

- 箭头函数
- 参数默认值
- 剩余参数与展开运算符

## 3、对象与数组扩展

- 对象字面量简写
- 动态对象属性名
- 数组方法扩展

## 4、类与继承

- class 关键字
- extends 和 super 关键字

## 5、异步编程

- promise
- async / await

## 6、模块化

ESM import / export

## 7、新增数据结构

Set 、 Map 和 Symbol

# 九、箭头函数和普通函数的区别？箭头函数可以当构造函数吗？

## 1、写法不同

【箭头函数】：省略 `function` 关键字，参数括号和函数体大括号可简化

【普通函数】：必须包含 `function` 关键字和完整的函数体

```js
const add = (a, b) => a + b;

function add(a, b) {
  return a + b;
}
```

## 2、this 指向不同

【箭头函数】：不绑定自己的 `this`，继承自外层上下文（词法作用域）

【普通函数】：`this` 指向调用该函数的对象（动态绑定）

```js
const obj = {
  name: 'Alice',
  sayHi: () => {
    console.log(this); // window 
  };
}
const obj = {
  name: 'Alice',
  sayHi: function() {
    console.log(this); // obj
  } 
};
obj.sayHi()
```

## 3、有无 arguments

【箭头函数】：没有自己的 `arguments` 对象，继承自外层函数

【普通函数】：拥有 `arguments` 对象，包含调用时的所有参数

```js
const arrowFunc = () => {
  console.log(arguments); // 报错（若外层无 arguments）
};

function normalFunc() {
  console.log(arguments); // [1,2,3] 输出参数类数组
}
normalFunc(1,2,3)
```

### **总结：为什么箭头函数不能当构造函数？**

1. **没有自己的** `**this**`：构造函数需要动态创建并绑定 `this` 到新实例，而箭头函数的 `this` 继承自外层，无法绑定到新对象。
2. **没有** `**[[Construct]]**` **内部方法**：JavaScript 引擎禁止使用 `new` 调用箭头函数。

# 十、事件循环

JavaScript 是单线程的，意味着，同一时间只能做一件事情，异步任务（如定时器，promise）无法立即执行，需要通过事件循环调度。

【宏任务】：

- setTimeout
- setTnterval
- I/O
- UI 渲染

【微任务】：

- promise.then()/catch()/finally()
- mutationObserver 监控 dom 元素的变化
- process.nextTick() （Node.js 中特有的微任务，执行在 promise 之前）

【注意】：

```js
new Promise((resolve, rejected) => {
  console.log(1) 
})
// 这个new Promise函数是直接执行的
```

# 十一、垃圾回收机制

计算机上所有的进程，都需要在内存上运行，如果不释放内存资源，那么就很容易导致内存用完了。电脑卡死。

早期，C 语言，C++这些语言需要开发者手动释放内存。为了方便开发，从 Java 开始，每个语言都有自己的自动垃圾回收机制（GC），那么 JavaScript 也不例外。**垃圾收集器会定期（周期性）找出那些不在继续使用的变量，然后释放其内存**

在 JS 中，有两种方式，

1. 标记清除【常用】

2. 当一个变量进入环境时，会标记“进入环境”
3. 退出环境时，会标记“离开环境”
4. GC 会定期清除有“离开环境”标记的变量

```js
function test(){
  var a = 10 ; // 被标记 ，进入环境 
  var b = 20 ; // 被标记 ，进入环境
}
test(); // 执行完毕 之后 a、b 又被标离开环境，被回收。
```

2. 引用计数【bug: 循环引用】

跟踪每一个变量的引用次数，如果是 0 那么说明，没有人去引用这个变量了，GC 就会将其回收

```js
function fn() {
  var a = {};
  var b = {};
  a.pro = b;
  b.pro = a;
}
fn();
```

以上代码 _a_ 和 _b_ 的引用次数都是 _2_，_fn_ 执行完毕后，两个对象都已经离开环境，在标记清除方式下是没有问题的，但是在引用计数策略下，因为 _a_ 和 _b_ 的引用次数不为 _0_，所以不会被垃圾回收器回收内存，如果 _fn_ 函数被大量调用，就会造成内存泄露。

# 十二、var、let、const 有什么区别？

|   |   |   |   |
|---|---|---|---|
||var|let|const|
|作用域|全局变量，定义的变量会加到 window 上，污染全局变量|块级变量|块级变量|
|变量提升|会提升到作用域顶端|会形成暂时性死区|会形成暂时性死区|
|重复声明，同一作用域|可以重复声明|不可以|不可以|
|可变性|可以|可以|不可以，指的是内存的地址不变|

# 十三、`==`与`===`的区别？

1. `**==**`**（相等运算符）**：在比较时会进行**类型转换**，即如果两个值的类型不同，它会尝试将它们转换为相同的类型，然后再比较值是否相等。这种转换遵循 JavaScript 的类型转换规则。
2. `**===**`**（严格相等运算符）**：在比较时不进行类型转换，它会同时比较**值和类型**。只有当两个值的类型相同且值也相等时，才会返回 `true`，否则返回 `false`。

```js
console.log(5 == '5');    // true，因为字符串 '5' 被转换为数字 5
console.log(5 === '5');   // false，因为类型不同（数字 vs 字符串）

console.log(true == 1);   // true，因为布尔值 true 被转换为数字 1
console.log(true === 1);  // false，因为类型不同（布尔 vs 数字）

console.log(null == undefined);  // true，JavaScript 特殊规则
console.log(null === undefined); // false，类型不同

console.log(0 == false);  // true，两者都被转换为数字 0
console.log(0 === false); // false，类型不同
```

# 十四、JavaScript 类型转化规则？

在 JS 中，有两种类型转化方式，一个是强制转化，另一个是隐式转化

## 强制转换（显式转换）

强制转换主要指使用`Number()`、`String()`和`Boolean()`三个函数，手动将各种类型的值，分别转换成数字、字符串或者布尔值。

#### _Number( )_

使用`Number`函数，可以将任意类型的值转化成数值。

下面分成两种情况讨论，一种是参数是原始类型的值，另一种是参数是对象。

**（1）原始类型值**

原始类型值的转换规则如下，

```
// 数值：转换后还是原来的值
Number(324) // 324

// 字符串：如果可以被解析为数值，则转换为相应的数值
Number('324') // 324

// 字符串：如果不可以被解析为数值，返回 NaN
Number('324abc') // NaN 这个和 parseInt("324abc") --> 324 不同

// 空字符串转为0
Number(''/""/``) // 0

// 布尔值：true 转成 1，false 转成 0
Number(true) // 1
Number(false) // 0

// undefined：转成 NaN
Number(undefined) // NaN

// null：转成0 --> 这个是历史遗留问题
Number(null) // 0

// 注意：
parseInt(undefined); // NaN
parseInt(null); // NaN
```

`Number`函数将字符串转为数值，要比`parseInt`函数严格很多。基本上，只要有一个字符无法转成数值，整个字符串就会被转为`NaN`。

```
parseInt('42 cats') // 42
Number('42 cats') // NaN
```

上面代码中，`parseInt`逐个解析字符，而`Number`函数整体转换字符串的类型。

另外，`parseInt`和`Number`函数都会自动过滤一个字符串前导和后缀的空格。

```
parseInt('\t\v\r12.34\n') // 12
Number('\t\v\r12.34\n') // 12.34
```

**（2）对象**

简单的规则是，`Number`方法的参数是对象时，将返回`NaN`，除非是包含单个数值的数组。

```
Number({a: 1}) // NaN
Number([1, 2, 3]) // NaN
Number([5]) // 5
```

之所以会这样，是因为`Number`背后的转换规则比较复杂。

第一步，调`[Symbol.toPrimitive]`方法，优先级最高。如果没有返回原始类型，直接报错。

如果没有`[Symbol.toPrimitive]`这个方法，才会有下面的步骤。

第二步，调用对象自身的`valueOf`方法。如果返回原始类型的值，则直接对该值使用`Number`函数，不再进行后续步骤。

第三步，如果`valueOf`方法返回的还是对象，则改为调用对象自身的`toString`方法。如果`toString`方法返回原始类型的值，则对该值使用`Number`函数，不再进行后续步骤。如果不是，就报错。

`[Symbol.toPrimitive]、valueOf`和`toString`方法，都是可以自定义的。

```
const a = {
  i: 1,
  [Symbol.toPrimitive]() {
    return this.i++; // 这个函数会调用三次
  },
  // valueOf() {
  //   return this.i++;
  // },
  // toString() {
  //   return this.i++;
  // },
};
// 引用类型和普通数据类型 --> 引用数据类型转原始
if (a == 1 && a == 2 && a == 3) {
  console.log("不可能的式子成立了~");
}
```

#### _String( )_

`String`函数可以将**任意类型**的值转化成字符串，转换规则如下。

**（1）原始类型值**

- **数值**：转为相应的字符串。
- **字符串**：转换后还是原来的值。
- **布尔值**：`true`转为字符串`"true"`，`false`转为字符串`"false"`。
- **undefined**：转为字符串`"undefined"`。
- **null**：转为字符串`"null"`。

```
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"
```

**（2）对象**

`String`方法的参数如果是对象，返回一个类型字符串；如果是数组，返回该数组的字符串形式。

```
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
String([]) // "" 
```

`String`方法背后的转换规则，与`Number`方法基本相同，只是互换了`valueOf`方法和`toString`方法的执行顺序。

1. 先调`[Symbol.toPrimitive]`这个方法，跟 `Number` 一样
2. 在调`toString`方法
3. 在调 `valueOf`方法

下面是一个例子。

```
String({a: 1})
// "[object Object]"
// 实际上应该调[Symbol.toPrimitive]这个函数，但是开发者不能直接调，是给JS引擎调的

// 等同于
String({a: 1}.toString())
// "[object Object]"
```

下面是通过自定义`toString`方法，改变返回值的例子。

```
const obj = {
  a: 1,
  [Symbol.toPrimitive]() {
    return "12";
  },
  toString() {
    return "[object, object]";
  },
  valueOf() {
    return "{a: 1}";
  },
};

console.log(String(obj)); // "12"
```

#### _Boolean( )_

`Boolean()`函数可以将**任意类型**的值转为布尔值。

它的转换规则相对简单：除了以下五个值的转换结果为`false`，其他的值全部为`true`。

- `undefined`
- `null`
- `0`（包含`-0`和`+0`）
- `NaN`
- `''`（空字符串）

```
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean(''/""/``) // false
```

当然，`true`和`false`这两个布尔值不会发生变化。

```
Boolean(true) // true
Boolean(false) // false
```

注意，**所有对象（包括空对象）**的转换结果都是`true`，甚至连`false`对应的布尔对象`new Boolean(false)`也是`true`

```
Boolean({}) // true
Boolean([]) // true
Boolean(new Boolean(false)) // true
```

所有对象的布尔值都是`true`，这是因为 JavaScript 语言设计的时候，出于性能的考虑，如果对象需要计算才能得到布尔值，对于`obj1 && obj2`这样的场景，可能会需要较多的计算。为了保证性能，就统一规定，对象的布尔值为`true`。

## 自动转换（隐式转换）

下面介绍自动转换，它是以强制转换为基础的。

遇到以下三种情况时，JavaScript 会自动转换数据类型，即转换是自动完成的，用户不可见。

第一种情况，不同类型的数据互相运算。

```
123 + 'abc' // "123abc"
```

第二种情况，对非布尔值类型的数据求布尔值。

```
if ('abc') {
  console.log('hello')
}  // "hello"
```

第三种情况，对非数值类型的值使用一元运算符（即`+`和`-`）。

```
+ {foo: 'bar'} // NaN
- [1, 2, 3] // NaN
```

自动转换的规则是这样的：预期什么类型的值，就调用该类型的转换函数。比如，某个位置预期为字符串，就调用`String()`函数进行转换。如果该位置既可以是字符串，也可能是数值，那么默认转为数值。

由于自动转换具有不确定性，而且不易除错，建议在预期为布尔值、数值、字符串的地方，全部使用`Boolean()`、`Number()`和`String()`函数进行显式转换。

#### 自动转换为布尔值

JavaScript 遇到预期为布尔值的地方（比如`if`语句的条件部分），就会将非布尔值的参数自动转换为布尔值。系统内部会自动调用`Boolean()`函数。

因此除了以下五个值，其他都是自动转为`true`。

- `undefined`
- `null`
- `+0`或`-0`
- `NaN`
- `''`（空字符串）

下面这个例子中，条件部分的每个值都相当于`false`，使用否定运算符后，就变成了`true`。

```
if ( !undefined
  && !null
  && !0
  && !NaN
  && !''
) {
  console.log('true');
} // true
```

下面两种写法，有时也用于将一个表达式转为布尔值。它们内部调用的也是`Boolean()`函数。

```
// 写法一
expression ? true : false

// 写法二
!! expression
```

#### 自动转换为字符串

JavaScript 遇到预期为字符串的地方，就会将非字符串的值自动转为字符串。具体规则是，先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串。

字符串的自动转换，主要发生在字符串的加法运算时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。

```
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"
```

这种自动转换很容易出错。

```
var obj = {
  width: '100'
};

obj.width + 20 // "10020"
```

上面代码中，开发者可能期望返回`120`，但是由于自动转换，实际上返回了一个字符`10020`。

#### 自动转换为数值

JavaScript 遇到预期为数值的地方，就会将参数值自动转换为数值。系统内部会自动调用`Number()`函数。

除了加法运算符（`+`）有可能把运算子转为字符串，其他运算符都会把运算子自动转成数值。

```
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5' * []    // 0 
false / '5' // 0
'abc' - 1   // NaN
null + 1 // 1
undefined + 1 // NaN
```

上面代码中，运算符两侧的运算子，都被转成了数值。

注意：`null`转为数值时为`0`，而`undefined`转为数值时为`NaN`。

一元运算符也会把运算子转成数值。

```
+'abc' // NaN
-'abc' // NaN
+true  // 1
-false // 0
```

## 真题解答

- _JavaScript_ 中如何进行数据类型的转换？

参考答案：

类型转换可以分为两种，**隐性转换**和**显性转换**。

**1. 隐性转换**

当不同数据类型之间进行相互运算，或者当对非布尔类型的数据求布尔值的时候，会发生隐性转换。

预期为数字的时候：算术运算的时候，我们的结果和运算的数都是数字，数据会转换为数字来进行计算。

预期为字符串的时候：如果有一个操作数为字符串时，使用`+`符号做相加运算时，会自动转换为字符串。

预期为布尔的时候：前面在介绍布尔类型时所提到的 9 个值会转为 false，其余转为 true

**2. 显性转换**

所谓显性转换，就是只程序员强制将一种类型转换为另外一种类型。显性转换往往会使用到一些转换方法。常见的转换方法如下：

- 转换为数值类型：`Number()`，`parseInt()`，`parseFloat()`
- 转换为布尔类型：`Boolean()`
- 转换为字符串类型：`toString()`，`String()`

当然，除了使用上面的转换方法，我们也可以通过一些快捷方式来进行数据类型的显性转换，如下：

- 转换字符串：直接和一个空字符串拼接，例如：`a = "" + 数据`
- 转换布尔：!!数据类型，例如：`!!"Hello"`
- 转换数值：数据*1 或 /1，例如：`"Hello * 1"`

# 十五、手写flat 函数

```
function flat(arr, depth = 0) {
  const res = [];
  for (const item of arr) {
    if (Array.isArray(item) && depth > 0) {
      res.push(...flat(item, depth - 1)); // 每次flat返回的是一维数组
    } else {
      res.push(item);
    }
  }
  return res;
}

console.log(flat([1, 2, 3, [4, 5, [6, 7, [8, 9]]]]));
```

# 十六、手写 filter 函数

```
// thisArg：可选 用于普通函数调整 this 的指向，箭头函数就无所谓了
Array.prototype.MyFilter = function (callback, thisArg) {
  const res = [];
  const array = this;
  for (let i = 0; i < array.length; i++) {
    // 这里是处理稀疏数组 没有的下标直接跳过
    if (i in array) {
      const value = array[i];
      // value：当前数组的值 i：索引值 array：整个数组
      // 至于这里为什么要重新绑定this的指向，是因为兼容一些普通函数的特殊操作【自认为哈~】
      if (callback.call(thisArg, value, i, array)) {
        res.push(value);
      }
    }
  }
  return res;
};

const arr = [1, 1, 2, 3, 4, 5, 5];
const obj = { age: 2 };
console.log(
  arr.MyFilter(function (i) {
    return i > this.age; // 这里的this指向的就是obj，当然只对普通函数有作用
  }, obj)
);
```

# 十七、call、apply、bind 区别？

在 JavaScript 中，`call`、`apply` 和 `bind` 都是 Function **实例方法**，用于改变函数执行时的 `this` 上下文，但它们的使用方式和返回值有所不同

|   |   |   |   |
|---|---|---|---|
|**方法**|**立即执行？**|**参数传递方式**|**返回值**|
|`call`|✅ 是|**逐个**参数传入 如 `fn.call(thisArg, a, b)`|函数执行结果|
|`apply`|✅ 是|**数组**形式传入 如`fn.apply(thisArg, [a, b])`|函数执行结果|
|`bind`|❌ 否|类似 `call`，但返回**新函数** 如`const newFn = fn.bind(thisArg, a, b)`|绑定了 `this`<br><br>的新函数|

如果，绑定的 this 对象是基本数据类型，其实是绑定了它的包装对象。

比如，

```
function fn() {
  console.log(this);
}

fn.call(true);
// Boolean {true}
```

## 应用场景：

#### `**call**` **和** `**apply**`

- **立即调用函数**并指定 `this` 上下文。
- **借调方法**：复用其他对象的方法（如 `[].slice.call(arguments)` 将类数组转为数组）。
- **动态参数传递**：`apply` 适合参数数量不确定的场景（如 `Math.max.apply(null, [1, 2, 3])`）。

`**bind**`

- 常用于**事件处理或异步**回调中保持 `this` 上下文

# 十八、对象与 Map 的区别？

## 1、键的不同

对象的 key 只能用 String 和 Symbol 类型

```
const obj = {};
obj[123] = "数字键";
obj[Symbol("symbol")] = "456";

// 其他的类型都会被自动转化成字符串
obj[true] = "12";
obj[undefined] = "78";
obj[null] = "7845";
obj[function () {}] = "789";
obj[{ a: 1 }] = "52"; // 特别注意：这里会转化成"[Object, Object]"

console.log(obj);
```

但是 Map 可以用**任意类型**： 包括对象，函数，基本类型

```
const map = new Map();
map.set(123, "13");
map.set("123", "14");
map.set(true, "45");
map.set("true", "45");
map.set(undefined, "78");
map.set(function () {}, "78546");

console.log(map);
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750425518933-471c3f50-23cf-475d-8cae-a302e2d89a07.png)

## 2、是否迭代

对象是不可以使用 for...of...来遍历，可以使用 for...in...

```
// 但是可以用 entries辅助遍历 for of
const obj = { a: 1, b: 2 };
for (const [key, value] of Object.entries(obj)) {
  console.log(key, value);
}
```

map 可以被迭代的，Map 数据结构对象本身提供了 forEach、for...of 来遍历

map 竟然也可以被 forEach 遍历，crazy！我还以为只有数组才可以使用 forEach 遍历

```
map.forEach((item, map) => {
  console.log(item, map);
});
```

## 3、键值对的顺序

Object：属性是**无序**的（ES6 之后部分顺序有保证，但不建议依赖）

Map：键值对有**插入顺序**，遍历时顺序固定

## 4、统计属性的长度

Object：`Object.keys(obj).length`

Map：`map.size`

## 5、有无原型链

Object：有原型链，可能存在默认属性，不过可以通过`Object.create(null)`创建无原型的对象

```
const obj = {};
console.log(obj.toString); // 输出: ƒ toString() { [native code] }

const pureObj = Object.create(null);
console.log(pureObj.toString); // 输出: undefined
```

Map：无原型链

```
const map = new Map();
console.log(map.hasOwnProperty); // 输出: undefined
```

## 6、适用场景

Object：更加适用于结构**固定**的结构，比如：JSON 格式，或者需要固定属性的对象

Map：更加适用于**频繁**进行 CRUD 的“字典”或“哈希表”

# 十九、值和引用

JavaScript 中的值分为基本数据类型和引用数据类型。

基本数据类型：储存在栈中

```
number string boolean null undefined symbol bigint
```

引用数据类型：地址存在栈中，内容存在堆中

```
object
```

面试题：为什么 null 和 undefined 明明是两种数据类型，但是 undefined==null 返回 true 呢？

这个是因为，历史遗留问题，

1. 由于设计问题 typeof null => "object" 本身就是一个坑，会带来未知的错误
2. JS 是弱语言，在比较的时，会自动转类型。在 C 语言的传统中，null 会被转化成 0，在开发中很难找出是那个地方错了

因此，JS 的设计者基于上面的问题，就重新设计了 undefined 这一数据类型。

所以说，undefined 设计出来就是来填 null 的坑的。

# 二十、包装类型

这个其实跟 Java 很像，类似于自动装箱

其目的：可以让基本数据类型，方便的调用包装类型的属性和方法

常见的包装类型：String、Number、Boolean、RegExp、Function、Object、Array、Date

例如说，

```
let num = 3.14156; 
console.log(num.toFixed(2));
```

JS引擎一看，一个基本数据类型调用方法了，它在底层就会自动装箱

1. 创建一个对应类型的包装类型
2. 在包装类型上调方法，在传回去
3. 销毁包装方法

大致是下面的实现：

```
var _num = new Number(3.14156);
var num = _num.toFixed(2);
_num = null;
```

其他的基本数据类型，同理。

为什么基本数据类型不能给它加属性呢？

比如：

let i = 1;

i.test = "hello";

因为，在底层转包装类型时，最后将包装类型销毁了

```
var _i = new Number(1);
_i.test = "hello";
_i = null;
```

那么怎么给基本数据类型加属性呢？（虽然不推荐这么做）

在对应包装方法的原型上添加方法或属性

```
Number.prototype.test = "hello";
var i = 1;
console.log(i.test); // "hello"
```

# 二十一、执行栈和执行上下文

执行上下文，英文全称为 _Execution Context_，一句话概括就是“代码（全局代码、函数代码）执行前进行的准备工作”，也称之为“执行上下文环境”。

运行 _JavaScript_ 代码时，当代码执行进入一个环境时，就会为该环境创建一个执行上下文，它会在你运行代码前做一些准备工作，如确定作用域，创建局部变量对象等。

JavaScript 的执行环境：

1. 全局环境
2. 函数环境
3. eval 环境（webpack 中看到过，好像是打包后的结果放在 eval 环境中执行？）

_JavaScript_ 运行时首先会进入全局环境，对应会生成全局上下文。程序代码中基本都会存在函数，那么**调用函数**，就会进入函数执行环境，对应就会生成该函数的执行上下文。

由于代码中会声明多个函数，对应的函数执行上下文也会存在多个。在 _JavaScript_ 中，通过栈的存取方式来管理执行上下文，我们可称其为执行栈，或函数调用栈（_Call Stack_）。

## 执行栈

程序执行进入一个执行环境时，它的执行上下文就会被创建，并被推入执行栈中（入栈）；程序执行完成时，它的执行上下文就会被销毁，并从栈顶被推出（出栈），控制权交由下一个执行上下文。因为 _JavaScript_ 在执行代码时最先进入全局环境，所以**处于栈底的永远是全局环境的执行上下文**。而处于**栈顶的是当前正在执行函数的执行上下文**。

```
function foo () { 
  function bar () {        
    return 'I am bar';
  }
  return bar();
}
foo();
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751789587481-26809706-337c-4a5b-8929-3ea8547c830a.png)

## 执行上下文的生命周期

分为两个阶段，创建阶段以及执行阶段

**创建阶段**

1. 创建变量对象（VO）

2. 确定函数形参，并赋值
3. 确定 arguments 对象。并赋值
4. 确定普通字面量形式的函数声明，并赋值
5. 确定变量声明，未赋值

6. 确定 this 指向
7. 确定作用域

当处于执行上下文的建立阶段时，我们可以将整个上下文环境看作是一个对象。该对象拥有 _3_ 个属性，如下：

```
executionContextObj = {
  variableObject : {}, // 变量对象，里面包含 Arguments 对象，形式参数，函数和局部变量
  scopeChain : {},// 作用域链，包含内部上下文所有变量对象的列表
  this : {}// 上下文中 this 的指向对象
}
```

在函数的建立阶段，首先会建立 _Arguments_ 对象。然后确定形式参数，检查当前上下文中的函数声明，每找到一个函数声明，就在 _variableObject_ 下面用函数名建立一个属性，属性值就指向该函数在内存中的地址的一个引用。

如果上述函数名已经存在于 _variableObject_（简称 _VO_） 下面，那么对应的属性值会被新的引用给覆盖。

最后，是确定当前上下文中的局部变量，如果遇到和函数名同名的变量，则会忽略该变量。

**执行阶段**

1. 变量对象赋值

- 变量赋值
- 函数表达式赋值

2. 调用函数
3. 顺序执行其它代码

```
const foo = function (i) {
  console.log(a);
  console.log(b);
  console.log(c);
  var a = "Hello";
  var b = function privateB() {};
  function c() {}
};
foo(10);
```

这段代码的 VO 是：

```
vo ={
	i : 10,
	arguments: {i : 10 mlength: 1},
	c: function() {}
	a : undefined
	b : undefined
}
```

# 二十二、作用域和作用域链

## 经典真题

- 谈谈你对作用域和作用域链的理解？

## 作用域（_Scope_）

### 什么是作用域

作用域是在运行时代码中的某些特定部分中变量，函数和对象的可访问性。

换句话说，作用域决定了代码区块中变量和其他资源的可见性。

可能这两句话并不好理解，我们先来看个例子：

```
function outFun2() {
    var inVariable = "内层变量2";
}
outFun2();
console.log(inVariable); // Uncaught ReferenceError: inVariable is not defined
```

从上面的例子可以体会到作用域的概念，变量 _inVariable_ 在全局作用域没有声明，所以在全局作用域下取值会报错。

我们可以这样理解：**作用域就是一个独立的地盘，让变量不会外泄、暴露出去**。也就是说**作用域最大的用处就是隔离变量，不同作用域下同名变量不会有冲突。**

_**ES6**_ **之前** _**JavaScript**_ **没有块级作用域，只有全局作用域和函数作用域**。

_ES6_ 的到来，为我们提供了“块级作用域”，可通过新增命令 _let_ 和 _const_ 来体现。

### 全局作用域和函数作用域

**（1）全局作用域**

在代码中任何地方都能访问到的对象拥有全局作用域，一般来说以下几种情形拥有全局作用域：

- 最外层函数和在最外层函数外面定义的变量拥有全局作用域

```
var outVariable = "我是最外层变量"; //最外层变量
function outFun() { //最外层函数
    var inVariable = "内层变量";
    function innerFun() { //内层函数
        console.log(inVariable);
    }
    innerFun();
}
console.log(outVariable); // 我是最外层变量
outFun(); // 内层变量
console.log(inVariable); // inVariable is not defined
innerFun(); // innerFun is not defined
```

- 所有未定义直接赋值的变量自动声明为拥有全局作用域

```
function outFun2() {
    variable = "未定义直接赋值的变量";
    var inVariable2 = "内层变量2";
}
outFun2();//要先执行这个函数，否则根本不知道里面是啥
console.log(variable); //未定义直接赋值的变量
console.log(inVariable2); //inVariable2 is not defined
```

- 所有 _window_ 对象的属性拥有全局作用域

一般情况下，_window_ 对象的内置属性都拥有全局作用域，例如 _window.name、window.location、window.top_ 等等。

全局作用域有个弊端：如果我们写了很多行 _JS_ 代码，变量定义都没有用函数包括，那么它们就全部都在全局作用域中。这样就会污染全局命名空间， 容易引起命名冲突。

```
// 张三写的代码中
var data = {a: 100}

// 李四写的代码中
var data = {x: true}
```

这就是为何 _jQuery、Zepto_ 等库的源码，所有的代码都会放在 _(function(){....})( )_ 中。

因为放在里面的所有变量，都不会被外泄和暴露，不会污染到外面，不会对其他的库或者 _JS_ 脚本造成影响。这是函数作用域的一个体现。

**（2）函数作用域**

函数作用域，是指声明在函数内部的变量，和全局作用域相反，局部作用域一般只在固定的代码片段内可访问到，最常见的例如函数内部。

```
function doSomething(){
    var stuName="zhangsan";
    function innerSay(){
        console.log(stuName);
    }
    innerSay();
}
console.log(stuName); // 脚本错误
innerSay(); // 脚本错误
```

**作用域是分层的，内层作用域可以访问外层作用域的变量，反之则不行**。

我们看个例子，用泡泡来比喻作用域可能好理解一点：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752050421210-9901629a-8f05-4587-9cf3-300367c11352.png)

最后输出的结果为 _2、4、12_

- 泡泡 _1_ 是全局作用域，有标识符 _foo_；
- 泡泡 _2_ 是作用域 _foo_，有标识符 _a、bar、b_；
- 泡泡 _3_ 是作用域 _bar_，仅有标识符 _c_。

值得注意的是：**块语句（大括号“｛ ｝”中间的语句），如** _**if**_ **和** _**switch**_ **条件语句或** _**for**_ **和** _**while**_ **循环语句，不像函数，它们不会创建一个新的作用域**。在块语句中定义的变量将保留在它们已经存在的作用域中。

```
if (true) {
    // 'if' 条件语句块不会创建一个新的作用域
    var name = 'Hammad'; // name 依然在全局作用域中
}
console.log(name); // logs 'Hammad'
```

_JS_ 的初学者经常需要花点时间才能习惯变量提升，而如果不理解这种特有行为，就可能导致 _bug_ 。

正因为如此， _ES6_ 引入了块级作用域，让变量的生命周期更加可控。

### 块级作用域

块级作用域可通过新增命令 _let_ 和 _const_ 声明，所声明的变量在指定块的作用域外无法被访问。

块级作用域在如下情况被创建：

1. 在一个函数内部
2. 在一个代码块（由一对花括号包裹）内部

_let_ 声明的语法与 _var_ 的语法一致。你基本上可以用 _let_ 来代替 _var_ 进行变量声明，但会将变量的作用域限制在当前代码块中。块级作用域有以下几个特点：

## 作用域链

### 什么是自由变量

首先认识一下什么叫做**自由变量** 。

如下代码中，_console.log(a)_ 要得到 _a_ 变量，但是在当前的作用域中没有定义 _a_（可对比一下 _b_）。当前作用域没有定义的变量，这成为自由变量 。

自由变量的值如何得到 ？

需要向父级作用域寻找（注意：这种说法并不严谨，下文会重点解释）。

```
var a = 100
function fn() {
    var b = 200
    console.log(a) // 这里的 a 在这里就是一个自由变量
    console.log(b)
}
fn()
```

### 什么是作用域链

如果父级也没呢？

再一层一层向上寻找，直到找到全局作用域还是没找到，就宣布放弃。这种一层一层的关系，就是作用域链 。

```
var a = 100
function f1() {
    var b = 200
    function f2() {
        var c = 300
        console.log(a) // 100 自由变量，顺作用域链向父作用域找
        console.log(b) // 200 自由变量，顺作用域链向父作用域找
        console.log(c) // 300 本作用域的变量
    }
    f2()
}
f1()
```

### 关于自由变量的取值

关于自由变量的值，上文提到要到父作用域中取，其实有时候这种解释会产生歧义。

```
var x = 10
function fn() {
    console.log(x)
}
function show(f) {
    var x = 20;
    (function () {
        f() // 10，而不是 20
    })()
}
show(fn)
```

在 _fn_ 函数中，取自由变量 _x_ 的值时，要到哪个作用域中取 ？

要到创建 _fn_ 函数的那个作用域中取，**无论** _**fn**_ **函数将在哪里调用**。

所以，不要在用以上说法了。相比而言，用这句话描述会更加贴切：**要到创建这个函数的那个域”。作用域中取值，这里强调的是“创建”，而不是“调用”**，切记切记，其实这就是所谓的"静态作用域"。

再来看一个例子：

```
const food = "rice";
const eat = function () {
    console.log(`eat ${food}`);
};
(function () {
    const food = "noodle";
    eat(); // eat rice
})();
```

在本示例中，最终打印的结果为 _eat rice_。因为对于 _eat( )_ 函数来说，创建该函数时它的父级上下文为全局上下文，所以 _food_ 的值为 _rice_。

如果我们将代码稍作修改，改成如下：

```
const food = "rice";
(function () {
    const food = "noodle";
    const eat = function () {
        console.log(`eat ${food}`);
    };
    eat(); // eat noodle
})();
```

这个时候，打印出来的值就为 _eat noodle_。因为对于 _eat( )_ 函数来讲，创建它的时候父级上下文为 _IIFE_，所以 _food_ 的值为 _noodle_。

## 作用域与执行上下文

许多开发人员经常混淆作用域和执行上下文的概念，误认为它们是相同的概念，但事实并非如此。

我们知道 _JavaScript_ 属于解释型语言，_JavaScript_ 的执行分为：解释和执行两个阶段，这两个阶段所做的事并不一样。

**解释阶段**

- 词法分析
- 语法分析
- 作用域规则确定

**执行阶段**

- 创建执行上下文
- 执行函数代码
- 垃圾回收

_JavaScript_ 解释阶段便会确定作用域规则，因此作用域在函数定义时就已经确定了，而不是在函数调用时确定，但是执行上下文是函数执行之前创建的。

执行上下文最明显的就是 _this_ 的指向是执行时确定的。而作用域访问的变量是编写代码的结构确定的。

作用域和执行上下文之间最大的区别是：

**执行上下文在运行时确定，随时可能改变，作用域在定义时就确定，并且不会改变**。

## 真题解答

- 谈谈你对作用域和作用域链的理解？

参考答案：

**什么是作业域 ？**

_ES5_ 中只存在两种作用域：全局作用域和函数作用域。

在 _JavaScript_ 中，我们将作用域定义为一套规则，这套规则用来管理引擎如何在当前作用域以及嵌套子作用域中根据标识符名称进行变量（变量名或者函数名）查找。_ES6_ 新增了块级作用域。

**什么是作用域链 ？**

当访问一个变量时，编译器在执行这段代码时，会首先从当前的作用域中查找是否有这个标识符，如果没有找到，就会去父作用域查找，如果父作用域还没找到继续向上查找，直到全局作用域为止。

而作用域链，就是有当前作用域与上层作用域的一系列变量对象组成，它保证了当前执行的作用域对符合访问权限的变量和函数的有序访问。

作用域链有一个非常重要的特性，**那就是作用域中的值是在函数创建的时候，就已经被存储了，是静态的**。

所谓静态，就是说作用域中的值一旦被确定了，永远不会变。**函数可以永远不被调用，但是作用域中的值在函数创建的时候就已经被写入了，**并且存储在函数作用域链对象里面。

# 二十三、this 指向问题

_this_ 的指向规律有如下几条：

- 在函数体中，非显式或隐式地简单调用函数时，在严格模式下，函数内的 _this_ 会被绑定到 _undefined_ 上，在非严格模式下则会被绑定到全局对象 _window/global_ 上。
- 一般使用 _new_ 方法调用构造函数时，构造函数内的 _this_ 会被绑定到新创建的对象上。
- 一般通过 _call/apply/bind_ 方法显式调用函数时，函数体内的 _this_ 会被绑定到指定参数的对象上。但是注意，不能给箭头函数调这些方法，因为箭头函数中没有 this 指向
- 一般通过上下文对象调用函数时，函数体内的 _this_ 会被绑定到该对象上。
- 在箭头函数中，_this_ 的指向是由外层（函数或全局）作用域来决定的。

# 二十四、闭包

闭包呢，其实不是某种技术，而是一种现象。就是，在函数内部使用了函数外部的变量。

在 JS 中，作用域链实现了闭包。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752119051593-b1a577b1-93ca-4543-abdb-2b065c706cb7.png)

自动形成的闭包，调完就会被销毁。。。

那么，怎么手动创建一个闭包呢？

```
function a() {
  const food = "rice";
  return function () {
    console.log(food);
  };
}

const t = a();
t(); // a()调用完毕后，不会被销毁
// 只有等 t 这个变量，没有被引用了，food才会被销毁
```

本来，在函数内部声明的变量，在外部是不能被访问的。一个是作用域控制了，另一个是函数调用后，里面的变量会被 gc 销毁掉。

但是，通过上面的例子，我们可以看到，

闭包可以让外部的环境使用到函数内部的变量。

闭包是什么？闭包的应用场景有哪些？怎么销毁闭包？

1. 闭包是一个封闭的空间，里面存储了在其他地方会引用到的该作用域的值，在 _JavaScript_ 中是通过作用域链来实现的闭包。
2. 只要在函数中使用了外部的数据，就会自动创建闭包。
3. 我们还可以通过一些手段手动创建闭包，从而让外部环境访问到函数内部的局部变量，让局部变量持续保存下来，不随着它的上下文环境一起销毁。

使用闭包可以解决一个全局变量污染的问题。

如果是自动产生的闭包，我们无需操心闭包的销毁，而如果是手动创建的闭包，可以把被引用的变量设置为 _null_，即手动清除变量，这样下次 _JavaScript_ 垃圾回收器在进行垃圾回收时，发现此变量已经没有任何引用了，就会把设为 _null_ 的量给回收了。

# 二十五、Dom 的事件传播机制

简单来说，

标准的 dom 事件传播机制，分为三个阶段：

事件捕获阶段 -> 事件处理阶段 -> 事件冒泡阶段

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752134722663-120a5f1f-eaa5-4c4b-a956-6090ba00d7f9.png)

事件委托：其实是基于冒泡事件的

事件委托利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件

比如：

```
import React from "react";
function App() {
  function handleClick(e) {
    if (e.target.nodeName === "LI") {
      console.log("hello world");
    }
  }

  return (
    <>
      <ul onClick={(e) => handleClick(e)}>
        <li>1</li>
        <li>2</li>
        <li>3</li>
        <li>4</li>
        <li>5</li>
        <li>6</li>
        <li>7</li>
        <li>8</li>
        <li>9</li>
        <li>10</li>
      </ul>
    </>
  );
}
export default App;
```

虽然用了 _react_ 来写，但是都是一样的，

点击 li 的时候，因为冒泡的原因，会执行父元素 ul 的事件。实现这个功能，只需要给 ul 添加一个事件，如果按照常规的写法，给所有 li 添加事件的话，太消耗性能了。

# 二十六、阻止默认行为

1. `_**e.preventDefualt()**_`【最常用的】
2. 在 dom1 的写法中，`_**return false**_`
3. `_**e.returnValue = false;**_` 已经从标准中移除了，所以不要用

还有两个属性，来辅助，

1. `_**e.cancelable**_`：该属性返回一个布尔值，表示事件是否可以取消。
2. `_**e.defaultPrevented**_`_**：**_ 该属性表示默认行为是否被阻止，返回 _true_ 表示被阻止，返回 _false_ 表示未被阻止。

# 二十七、严格模式

1. 严格模式通过**抛出错误**来消除了一些原有**静默错误**。
2. 严格模式修复了一些导致 JavaScript 引擎难以执行优化的缺陷：有时候，相同的代码，严格模式可以比非严格模式下**运行得更快**。
3. 严格模式**禁用了**在 ECMAScript 的未来版本中可能会定义的一些语法。

**开启严格模式**

1. 脚本开启，

```
"use strict"
const i =1;
```

2. 在函数内部开启，

```
function test() {
  "use strict"
  //...
}
```

**区别**

1. 给未声明的变量赋值，报错

```
"use strict";
// let i;

// 假设全局变量 i 不存在，这行会抛出 ReferenceError
i = 17; // 在普通模式下，这个变量会被挂载到window对象上
```

2. 给不能赋值的属性(NaN，只读属性，不可扩展属性)赋值
3. 删除一个不可删除的属性

```
"use strict";
delete Object.prototype; // 抛出 TypeError 错误
```

4. 同名的形参
5. 禁止八进制语法（以 0 开头的） -> es6 中使用 `0o 开头`
6. `eval`会开辟一个新的作用域，不会影响到全局作用域
7. 不能使用保留字作为标识符

# 二十八、0.1 + 0.2 !== 0.3

这个问题不是只有 JS 才存在 ，是所有的编程语言都会存在。

为什么呢？

因为，处理小数用的都是`IEEE-754`的标准，在 JS 中使用的是双精度 64 位表示所有的数（无论整数还是小数）

我们只需要了解，一个数存在计算机中，会用二进制来描述。`IEEE-754`来描述 64 的浮点数，其组成如下图所示，

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752139111255-ea9e972f-bbb2-4114-99cc-53d427231a03.png)

小数，在计算机中，转化成二进制是通过“乘 2 取整”这样有些小数就可能出现取不尽的问题。计算机只能截取前 52 位二进制数，所以说转化成二进制的时候就有误差，计算的时候误差也会存在。

比方说，

_0.1：0.0001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1001 1100 ..._

_0.2：0.0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 0011 ..._

_sum：0.0100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 1100 ..._

最终我们只能得到和的近似值（按照 _IEEE 754_ 标准保留 _52_ 位，按 _0_ 舍 _1_ 入来取值），然后转换为十进制数变成：

sum ≈ 0.30000000000000004

值得注意的是，在 JS 中大的整数也会有精度的误差

因为，整数也是按照浮点数来存的，整数是按最大 54 位来算，也就是说，

- 最大( _2__53_ _- 1_，_Number.MAX_SAFE_INTEGER_、_9007199254740991_)
- 最小( _-(2__53_ _- 1)_，_Number.MIN_SAFE_INTEGER_、_-9007199254740991_)

所以只要超过这个范围，就会存在被舍去的精度问题。

# 二十九、柯里化函数

那么柯里化函数，它是**函数式编程**里面的概念，

**柯里化（Currying）是一种处理****多元函数****的方法。它产生一系列连锁函数，其中每个函数****固定****部分参数，并返回一个新函数，用于处理其它剩余参数**。

一个小例子，

```
// 正常的写法
function add(x, y) {
  return x + y;
}
console.log(add(1, 2)); // 3
console.log(add(5, 7)); // 12

// 柯里化函数：就是一次只接收一个函数，然后剩下的参数通过返回函数继续接收
function _add(x) {
  return function (y) {
    console.log(x + y);
  };
}
_add(1)(2); // 3
_add(5)(7); // 12
```

那么，看上去好像也没有什么区别呀，

所以，使用柯里化函数有什么好处呢？最大的好处就是参数复用！！

## 参数复用

意思就是，一个函数有很多的参数，但在调用的时候，发现前面几个参数都是固定的，这个时候使用柯里化函数的好处就显而易见了。

比如，

```
// 正常正则验证字符串 reg.test(txt)

// 函数封装后
function check(reg, txt) {
    return reg.test(txt)
}

// 即使是相同的正则表达式，也需要重新传递一次
console.log(check(/\d+/g, 'test1')); // true
console.log(check(/\d+/g, 'testtest')); // false
console.log(check(/[a-z]+/g, 'test')); // true

// Currying后
function curryingCheck(reg) {
    return function (txt) {
        return reg.test(txt)
    }
}

// 正则表达式通过闭包保存了起来
var hasNumber = curryingCheck(/\d+/g)
var hasLetter = curryingCheck(/[a-z]+/g)

console.log(hasNumber('test1')); // true
console.log(hasNumber('testtest'));  // false
console.log(hasLetter('21212')); // false
```

## 封装通用的柯里化函数

```
function curry() {
  const fn = arguments[0];
  const args = Array.from(arguments).slice(1);
  if (args.length === fn.length) {
    return fn.apply(this, args);
  }
  function _curry() {
    // 处理后续的参数
    args.push(...arguments);
    if (args.length === fn.length) {
      return fn.apply(this, args);
    }
    return _curry;
  }

  return _curry;
}
```

## 一道经典的面试题

```
// 实现add函数，使得下面的式子成立

add(1)(2)(3) = 6;
add(1, 2, 3)(4) = 10;
add(1)(2)(3)(4)(5) = 15;
```

通过柯里化函数解决，

```
function add() {
  const args = Array.from(arguments).slice(); // 收集一开始传经来的参数
  function _add() { // 处理后续参数
    args.push(...arguments);
    return _add;
  }

  _add.toString = function () {   // 如果调用了toString方法,那么后面一定没有参数了
    return args.reduce((a, b) => a + b);
  };
  return _add;
}

console.log(add(1)(2)(3).toString()); // 6
console.log(add(1, 2, 3)(4, 5).toString()); // 15
console.log(add(1)(2)(3)(4)(5).toString()); // 15
```

# 三十、尺寸相关

主要分为以下两部分：

- _DOM_ 对象相关尺寸和位置属性

- 只读属性

- _clientWidth_ 和 _clientHeight_ 属性
- _offsetWidth_ 和 _offsetHeight_ 属性
- _clientTop_ 和 _clientLeft_ 属性
- _offsetLeft_ 和 _offsetTop_ 属性
- _scrollHeight_ 和 _scrollWidth_ 属性

- 可读可写属性

- _scrollTop_ 和 _scrollLeft_ 属性
- _domObj.style.xxx_ 属性

- _event_ 事件对象相关尺寸和位置属性

- _clientX_ 和 _clientY_ 属性
- _screenX_ 和 _screenY_ 属性
- _offsetX_ 和 _offsetY_ 属性
- _pageX_ 和 _pageY_ 属性

## _DOM_ 对象相关尺寸和位置属性

在 _DOM_ 对象所提供的尺寸和位置相关属性中，可以分为**只读属性**和**可读可写属性**。

### 只读属性

所谓的只读属性指的是 _DOM_ 节点的固有属性，该属性只能通过 _JavaScript_ 去获取而不能通过 _JavaScript_ 去设置，**而且获取的值是只有数字并不带单位的（** **_px、em_** **等 ）**

常见的只读属性有：

- _clientWidth_ 和 _clientHeight_ 属性
- _offsetWidth_ 和 _offsetHeight_ 属性
- _clientTop_ 和 _clientLeft_ 属性
- _offsetLeft_ 和 _offsetTop_ 属性
- _scrollHeight_ 和 _scrollWidth_ 属性

下面我们来一组一组进行介绍。

#### _clientWidth_ 和 _clientHeight_ 属性

该属性指的是元素的可视部分宽度和高度，即 _padding + content_，例如：

```
<div id="container" class="container"></div>
```

```
.container{
  width: 200px;
  height: 200px;
  background-color: red;
  border: 1px solid;
  overflow: auto;
  padding: 10px;
  margin: 20px;
}
```

```
var container = document.getElementById("container");
console.log("clientWidth:",container.clientWidth); // 220
console.log("clientHeight:",container.clientHeight); // 220
```

#### _offsetWidth_ 和 _offsetHeight_ 属性

这一对属性指的是元素的 _border+padding+content_ 的宽度和高度。例如：

```
var container = document.getElementById("container");
console.log("offsetWidth:", container.offsetWidth); // 222
console.log("offsetWidth:", container.offsetWidth); // 222
```

可以看到该属性和 _clientWidth_ 以及 _clientHeight_ 相比，多了设定的边框 _border_ 的宽度和高度。

#### _clientTop_ 和 _clientLeft_ 属性

这一对属性是用来读取元素的 _border_ 的宽度和高度的。例如：

```
var container = document.getElementById("container");
console.log("clientTop:", container.clientTop); // 1
console.log("clientLeft:", container.clientLeft); // 1
```

#### _offsetLeft_ 和 _offsetTop_ 属性

首先需要介绍一下 _offsetParent_ 属性，该属性是获取当前元素的离自己最近的并且定了位的祖先元素，该祖先元素就是当前元素的 _offsetParent_。

如果从该元素向上寻找，找不到这样一个祖先元素，那么当前元素的 _offsetParent_ 就是 _body_ 元素。

_offsetLeft_ 和 _offsetTop_ 指的是当前元素，相对于其 _offsetParent_ 左边距离和上边距离，即当前元素的 _border_ 到包含它的 _offsetParent_ 的 _border_ 的距离。

下面我们来具体举例说明：

```
var container = document.getElementById("container");
console.log(container.offsetParent); // body
```

可以看到，我们直接访问 _container_ 盒子的 _offsetParent_ 属性，因为不存在定了位的祖先元素，所以显示出来的是 _body_ 元素。

接下来我们对上面的代码进行改造：

```
<div id="container" class="container">
  <div id="item" class="item"></div>
</div>
```

```
.container {
  width: 200px;
  height: 200px;
  background-color: red;
  border: 1px solid;
  overflow: auto;
  padding: 10px;
  margin: 20px;
  position: relative;
}
.item{
  width: 50px;
  height: 50px;
  background-color: blue;
  position: absolute;
  left: 100px;
  top: 100px;
}
```

当前效果如下：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752311820139-7ec201f0-aed5-4e20-a3a4-13b79e824554.png)

接下来我们来获取 _item_ 盒子的 _offsetLeft_ 以及 _offsetTop_ 属性值。

```
var container = document.getElementById("container");
var item = document.getElementById("item");
console.log(item.offsetParent); // container 盒子 dom 对象
console.log(item.offsetLeft); // 100
console.log(item.offsetTop); // 100
```

接下来我们不对 _item_ 子元素进行定位，而是使用 _margin_ 的方式来设置子盒子的位置，如下：

```
.item{
  width: 50px;
  height: 50px;
  background-color: blue;
  margin: 50px;
}
```

然后再次获取 _item_ 盒子的 _offsetLeft_ 以及 _offsetTop_ 属性值，如下：

```
var container = document.getElementById("container");
var item = document.getElementById("item");
console.log(item.offsetParent); // container 盒子 dom 对象
console.log(item.offsetLeft); // 60
console.log(item.offsetTop); // 60
```

可以看到，打印出来的是 _60_，因为我们设置的 _margin_ 的值为 _50_，但是其定了位的父元素还设置了 _10_ 像素的 _padding_，所以加起来就是 _60_。

#### _scrollHeight_ 和 _scrollWidth_ 属性

内容的真实宽高

顾名思义，这两个属性指的是当元素内部的内容超出其宽度和高度的时候，元素内部内容的实际宽度和高度。

如果当前元素的内容没有超过其高度或者宽度，那么返回的就是元素的可视部分宽度和高度，即和 _clientWidth_ 和 _clientHeight_ 属性值相同。

```
<div id="container" class="container">Lorem ipsum dolor sit, amet consectetur adipisicing elit. Nulla repellat porro atque culpa rem sunt sed! Voluptates vel incidunt accusamus reiciendis aut, adipisci ut. Hic, impedit officia.Quis, animi beatae. Facere dolorum quasi laborum, rem facilis illum necessitatibus sint doloribus beatae
exercitationem sapiente! Quod vel cupiditate quam libero, delectus natus.</div>
```

```
.container {
  width: 200px;
  height: 200px;
  background-color: red;
  border: 1px solid;
  overflow: auto;
  padding: 10px;
  margin: 20px;
}
```

上面的代码中，我们为 _container_ 盒子加入了一些文字，使其能够产生滚动效果，接下来访问 _scrollHeight_ 和 _scrollWidth_ 属性，如下：

```
var container = document.getElementById("container");
console.log("scrollWidth:", container.scrollWidth); // scrollWidth: 220
console.log("scrollHeight", container.scrollHeight); // scrollHeight 372
```

如果 _container_ 盒子不具备滚动的条件，那么返回的值和 _clientWidth_ 和 _clientHeight_ 属性值是相同的。

```
<div id="container" class="container"></div>
```

```
var container = document.getElementById("container");
console.log("scrollWidth:", container.scrollWidth); // scrollWidth: 220
console.log("scrollHeight", container.scrollHeight); // scrollHeight 220
```

### 可读可写属性

所谓的可读可写属性指的是不仅能通过 _JavaScript_ 获取该属性的值，还能够通过 _JavaScript_ 为该属性赋值。

#### _scrollTop_ 和 _scrollLeft_ 属性

这对属性是可读写的，指的是当元素其中的内容超出其宽高的时候，元素**被卷起的高度和宽度**。

```
<div id="container" class="container">Lorem ipsum dolor sit, amet consectetur adipisicing elit. Nulla repellat porro atque culpa rem sunt sed! Voluptates vel incidunt accusamus reiciendis aut, adipisci ut. Hic, impedit officia.Quis, animi beatae. Facere dolorum quasi laborum, rem facilis illum necessitatibus sint doloribus beatae
exercitationem sapiente! Quod vel cupiditate quam libero, delectus natus.</div>
```

```
.container {
  width: 200px;
  height: 200px;
  background-color: red;
  border: 1px solid;
  overflow: auto;
  padding: 10px;
  margin: 20px;
}
```

```
var container = document.getElementById("container");
container.onscroll = function () {
  console.log("scrollTop:", container.scrollTop);
  console.log("scrollLeft", container.scrollLeft);
}
```

在上面的代码中，我们的 _container_ 盒子内容超出了容器的高度，我们为该盒子绑定了 _scroll_ 事件，滚动的时候打印出 _scrollTop_ 和 _scrollLeft_，即元素被卷起的高度和宽度。

效果如下：

![](https://cdn.nlark.com/yuque/0/2025/gif/51701855/1752311819956-8321b0a8-93c1-4fdb-bfb4-492342a221bd.gif)

该属性因为是可读可写属性，所以可以通过赋值来让内容自动滚动到某个位置。例如很多网站右下角都有回到顶部的按钮，背后对应的 _JavaScript_ 代码就是通过该属性来实现的。

#### _domObj.style.xxx_ 属性

对于一个 _DOM_ 元素来讲，它的 _style_ 属性返回的也是一个对象，并且这个对象中的任意一个属性是可读写的。例如 _domObj.style.top、domObj.style.wdith_ 等，在读的时候，它们返回的值常常是带有单位的（如 _px_ ）。

但是，对于这种方式，**它只能够获取到该元素的行内样式，而并不能获取到该元素最终计算好的样式**。如果想要获取计算好的样式，需要使用 *obj.currentstyle（ IE ）*和 _getComputedStyle_（ _IE_ 之外的浏览器 ）

另一方面，由于 _domObj.style.xxx_ 属性能够被赋值，所以 _JavaScript_ 控制 _DOM_ 元素运动的原理就是通过不断修改这些属性的值而达到其位置改变的，需要注意的是，**给这些属性赋值的时候需要带单位的要带上单位，否则不生效**。

## _event_ 事件对象相关尺寸和位置属性

对于元素的运动的操作，通常还会涉及到事件里面的 _event_ 对象，而 _event_ 对象也存在很多位置属性，且由于浏览器兼容性问题会导致这些属性间相互混淆，这里也来进行一个总结。

#### _clientX_ 和 _clientY_ 属性

这对属性是当事件发生时，**鼠标点击位置相对于浏览器（可视区）的坐标**，即浏览器左上角坐标的（ _0 , 0_ ），该属性以浏览器左上角坐标为原点，计算鼠标点击位置距离其左上角的位置，

不管浏览器窗口大小如何变化，都不会影响点击位置的坐标。

```
<div id="container" class="container"></div>
```

```
*{
  margin: 0;
  padding: 0;
}
.container {
  width: 200px;
  height: 200px;
  background-color: red;
  border: 1px solid;
  overflow: auto;
  padding: 10px;
  margin: 20px;
}
```

```
var container = document.getElementById("container");
container.onclick = function (ev) {
  var evt = ev || event;
  console.log(evt.clientX + ":" + evt.clientY);
}
```

在上面的代码中，我们为 _container_ 盒子绑定了点击事件，获取该盒子的 _clientX_ 以及 _clientY_ 的值，接下来我们来进行点击测试：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752311820133-888ace5e-4391-4bdc-94b2-e836f91b972b.png)

我们分别点击该盒子的最左上角和最右下角，打印出来的值分别是（_20 , 20_）和 （_241 , 241_），可以看出这确实是以浏览器左上角坐标的（ _0 , 0_ ）为原点来进行计算的。

#### _screenX_ 和 _screenY_ 属性

_screenX_ 和 _screenY_ 是事件发生时鼠标相对于屏幕的坐标，以设备屏幕的左上角为原点，事件发生时鼠标点击的地方即为该点的 _screenX_ 和 _screenY_ 值。例如：

```
var container = document.getElementById("container");
container.onclick = function (ev) {
  var evt = ev || event;
  console.log(evt.screenX + ":" + evt.screenY);
}
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1752311820367-d5893916-3bb8-487d-a1dd-12d77405d016.png)

我们将浏览器缩小，同样是点击 _container_ 盒子的最左上角，打印出来的是（_368 , 365_），因为这是以屏幕最左上角为原点的。

#### _offsetX_ 和 _offsetY_ 属性

这一对属性是指当事件发生时，鼠标点击位置相对于该事件源的位置，即点击该 _DOM_ 元素，以该 _DOM_ 左上角为原点来计算鼠标点击位置的坐标。例如：

```
var container = document.getElementById("container");
container.onclick = function (ev) {
  var evt = ev || event;
  console.log("offsetX:", evt.offsetX);
  console.log("offsetY:", evt.offsetY);
}
```

同样点击 _container_ 元素的最左上角，打印出（ _0, 0_ ）

#### _pageX_ 和 _pageY_ 属性

顾名思义，该属性是事件发生时鼠标点击位置相对于页面的位置，通常浏览器窗口没有出现滚动条时，该属性和 _clientX_ 及 _clientY_ 是等价的。

例如：

```
var container = document.getElementById("container");
container.onclick = function (ev) {
  var evt = ev || event;
  console.log("pageX:", evt.pageX);
  console.log("pageY:", evt.pageY);
  console.log("clientX:", evt.clientX);
  console.log("clientY:", evt.clientY);
}
```

此时点击 _container_ 盒子，得到的 _pageX、pageY_ 的值和 _clientX、clientY_ 完全相同。

但是当浏览器出现**滚动条**的时候，_pageX_ 通常会大于 _clientX_，因为页面还存在被卷起来的部分的宽度和高度。

例如：

```
...
body{
  height: 5000px;
}
...
```

我们为 _body_ 添加一个 _height_ 为 _500px_，使其能够产生滚动效果，此时两组属性的区别就出来了。如下：

![](https://cdn.nlark.com/yuque/0/2025/gif/51701855/1752311819945-0a334740-27d7-45fe-afa4-7b9349ab0c64.gif)

---

# 三十一、map foreach for...of...区别

# 三十二、在 web 开发中，什么是内存泄露，如何定位内存泄露

**内存泄露**是指程序中不再需要使用的内存未能被浏览器的垃圾回收机制（GC）正确释放，导致内存占用持续增长的现象。

长期内存泄露，会导致页面性能下降（卡断、响应变慢）

内存泄露的常见场景：

1. 未被声明的变量（会被挂载到 window 对象上）成为全局变量，无法被回收
2. 未清楚的闭包引用
3. 未移出的事件监听器
4. 未清除的定时器/回调