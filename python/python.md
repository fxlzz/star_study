>  复习一下 python 的语法，使用 JS 类比学习，使用苏格拉底提问法向 AI 提问

# 基础数据类型
| 类型  | JS                   | PY                                                 |
| --- | -------------------- | -------------------------------------------------- |
| 布尔  | `true / fale`        | `True / False`                                     |
| 空值  | `null / undefined`   | `None`                                             |
| 列表  | `const arr = [1, 2]` | `list = [1, 2]`                                    |
| 字典  | `const obj = {a: 1}` | `dict = {"a": 1}` 注意 python 字典 key 值只能是 string 类型的 |
| 元组  |                      | `tuple = (1, 2)`                                   |

PY 中的模板字符串 -- 加 f
`print(f"hello, my name is {name}")`

列表操作：
`append("1") ---> push("1")`

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

while a > 0: 
```