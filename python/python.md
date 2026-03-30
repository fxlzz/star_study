>  复习一下 python 的语法，使用 JS 类比学习，使用苏格拉底提问法向 AI 提问

# 基础数据类型
| 类型  | JS                   | PY                                                 |
| --- | -------------------- | -------------------------------------------------- |
| 布尔  | `true / fale`        | `True / False`                                     |
| 空值  | `null / undefined`   | `None`                                             |
| 列表  | `const arr = [1, 2]` | `list = [1, 2]`                                    |
| 字典  | `const obj = {a: 1}` | `dict = {"a": 1}` 注意 python 字典 key 值只能是 string 类型的 |
| 元组  |                      | `tuple = (1, 2)`                                   |

## *String* 操作
PY 中的模板字符串 --- 加 f
+  `print(f"hello, my name is {name}")`
+ `strip() ---> trim()`
+ `split(sep)`
+ `join(list)` <font color="#ff0000">注意! </font>Python 中 `"-".join(["a", "b"])` 在 JS 中是 `["a", "b"].join("-")
+ `replace(old, new)` 返回新串
+ `lower() / upper()` 返回新串
+ `startswith() / endswith()`

## *列表*操作
### 基础操作
+  `[start:end: step] ---> slice(start, end) 左闭右开` 
	1. 省略参数
```python
numbers = [10,20,30,40]
print(numbers[:3]) # [10, 20, 30] -- slice(0, 4)
print(numbers[2:]) # [30, 40] -- slice(2)
print(numbers[:]) # 浅拷贝 类似于 JS 扩展运算符[...numbers]
```
	2. 步长
```python
print(numbers[0:4:2]) # 隔一个取一个 [10, 30]
```
	3. 负数
```python
print(numbers[-1]) # 40 -- slice(-1)
print(numbers[::-1]) # 步长为 -1 代表翻转列表 [40, 30, 20, 10] -- numbers.rervse()
```
 
+ `list.append("1") ---> arr.push("1")`
+ `list.extend(list2) ---> arr.concat(arr2) 或 [...arr1, ...arr2]`
+ `list.insert(index, x) ---> arr.splice(index, 0, x)`
+ `list.remove(x) 删除值为 x`
+ `list.pop(index) ---> arr.pop()`
+ `list.index("xxx") ---> arr.indexOf("xxx") 返回第一个索引`
+ `list.sort() ---> arr.sort((a, b) => a - b)`
	+ `list.sort(reverse=True)`
	+ `list.sort(key=lambda x: x["score"], reverse=True) # 按分数降序`
---
### 函数式编程 
注意：python 返回的是一个*迭代器*，如果想看到具体的列表结果，需要用 `list()` 包裹一下
+  `map(func, iterable) --- arr.map(func)`
+ `fliter(func, iterable) --- arr.filter(func)`
+ `reduce(func, iterable) --- arr.reduce((pre, cur) => {})`

```python
nums = [1, 2, 3, 4]

doubled = list(map(lambda x: x * 2, nums)) # [2, 4, 6, 8]

evens = list(filter(lambda x: x % 2 == 0, nums)) # [2, 4]
```

不过 python 中更加推崇*列表推导式*，更容易阅读

---
上面的换成列表推导式的写法：
```python
nums = [1, 2, 3, 4]

doubled = [x * 2 for x in nums]

evens = [x for x in nums if x % 2 == 0]
```

### Lambda 表达式
语法公式：
`lambda 参数1, 参数2: 返回值表达式`

- **JS**: `(a, b) => a + b`
- **Python**: `lambda a, b: a + b`

注意：
+ `lambda` 只能写*一行*表达式，不能写多行，也不能写 return

### 列表推导式
语法公式：
`[ <结果表达式> for <变量> in <可迭代对象> if <过滤条件> ]`

```python
nums = [1, 2, 3, 4, 5]

# 循环
res = []
for i in nums:
	if i > 2:
		res.append(i * 2)

# 列表推导式 
# 将大于 2 的数 * 2 返回出来
res = [x * 2 for x in nums if x > 2]
print(res)
```

## *元组*操作
元组可看作是*不可变列表*，一旦创建，不能修改、添加、删除其中元素
```python
point = (10, 20)

# 解构可用于任何 "可迭代对象"
x, y = point
print(x, y)
```


## *字典*操作
```python
user = {
	"name" : "tom",
	"age" : 18,
}
user["addr"] = "hunan" # 添加或修改

# 读取
print(user["name"])
email = user.get("email") # 不存在返回 None，不会报错

```

#  可迭代对象
与 JS `Symbol.iterator` 类似，只要满足可迭代协议，就可视为*可迭代对象* 

只要对象实现了 `__iter__` (返回一个迭代器 `{next(), done: false}`)

常见的可迭代对象：
+ list、tuple、str、range
+ dict、set
+ 文件对象 ` for line in open ("file.txt")`
+ 生成器 generator

# 流程控制与循环
## 流程控制
```js
// JavaScript
if (a) {
} else if (b) {
} else {
}
```

```python
# python
if a:
    print("a")
elif b:
    print("b")
else:
    print("c")
```

## 循环
```js
// JavaScript
for (let i = 0; i < 10; i++) {}
while(a) {}
for (const o in obj) {}
for (const a of arr) {}
arr.forEach(item => {}) // 等一系列数组方法

Object.keys(obj).forEach(key => {})
Object.values(obj).forEach(val => {})
Object.entries(obj).forEach(([key, val]) => {})
Reflect.ownKeys(obj).forEach(key => {})
``` 

```python
# python
for tool in tools: # 类似于 js 的 for..of 循环，只能遍历可迭代对象

for i in range(5): # 0 1 2 3 4 不包含5 
# range(start, stop, step)
 
for index, item in enumerate(numbers): # 同时获取索引和值 --- 类似于 forEach((item, index) => {})

names= ["tom", "marry"]
ages = [18, 19]
# 配对，zip接收多对列表
for name, age in zip(names, ages):
	print(f"{name} 的年龄是: {age}")

while a > 0: 

for key in dict 或 dict.keys():
for value in dict.values():
for key, value in dict.items():
```