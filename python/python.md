>  复习一下 python 的语法，使用 JS 类比学习，使用苏格拉底提问法向 AI 提问

# 基础数据类型
| 类型  | JS                   | PY                                                 |
| --- | -------------------- | -------------------------------------------------- |
| 布尔  | `true / fale`        | `True / False`                                     |
| 空值  | `null / undefined`   | `None`                                             |
| 列表  | `const arr = [1, 2]` | `list = [1, 2]`                                    |
| 字典  | `const obj = {a: 1}` | `dict = {"a": 1}` 注意 python 字典 key 值只能是 string 类型的 |
| 元组  |                      | `tuple = (1, 2)`                                   |

PY 中的模板字符串 --- 加 f
`print(f"hello, my name is {name}")`

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

### 列表推导式
```python
response = {
    "model": "GPT-4o",
    "usage": (150, 200),  
    "tags": ["ai", "chat", "generative", "fullstack"]
}
tags = response.get("tags")
# 在标签中，挑选长度大于2的值
# JS
const res = tags.filter(t => t.length > 2)

# python
res = [t for t in tags if len(t) > 2]
```

## *元组*操作
元组可看作是*不可变列表*，一旦创建，不能修改、添加、删除其中元素
```python
point = (10, 20)

# 解构
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

for key, value in dict.items():
```