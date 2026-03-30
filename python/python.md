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

# 类型转换
与 JS 一样，python 作为弱语言类型，也会自动进行类型转换
## Bool 
一下都会被视为 False
+ `None, False`
+ `0, 0.0`
+ `"", [], (), {}, set()` 这里与 JS 不一样，JS 会将 `[], {}...` 视为 true

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

# 函数
## 定义与调用
Python 中支持，默认参数传递以及关键字参数
 
 ```python
 def ask_ai(prompt, model="gpt-4o", temperature=0.7):
 	return "AI 回复"
 
 # 正常调用
 ask_ai("你好", "claude-3", 0.5)
 
 # 关键字参数 -- 指定参数名
 ask_ai(temperature=0.9, prompt="写个诗", model="gpt-4")
 ```

其实与 JS 没什么区别，只不过 python 底层自己支持
```js
const func = ({ prompt, model="gpt-4o" }) => {}

func({ prompt: "hi", model: "qwen3-max"})
```

## 坑！默认参数
>  在 python 中，如果给默认参数，那么说明这个参数是*可选参数*

永远不要使用"*可变类型（列表、字典）*"作为函数的*默认参数*

```python
# python 中错误的写法
def fn(msg, list = []):
    list.append(msg)
    return list
    
print(fn("hello")) # ["hello"]
print(fn("world")) # ["hello", "world"]

# 正确写法：默认给 None
def fn(msg, list = None):
	if list is None: 
		list = []
	list.append(msg)
	return list
```

Python 的默认参数在*函数定义时* 执行，而不是每次调用时执行。因此，定义时 list 参数会变成"静态变量" 存放在内存中

每次调用，使用的都是同一块空间

而 JS 不同，每次调用，都是重新创建新的空间
```js
function fn(msg, arr = []) {
  arr.push(msg);
  return arr;
}
console.log(fn("hello")); // ["hello"]
console.log(fn("world")); // ["world"]
```

## 万能参数 `*args` 与 `**kwargs`
+ `*args` 将*多余参数*打包进一个*元组*
+ `**kwargs` 将*多余键值对*参数打包进一个*字典*

```python
def fn(task_name, *tags, **configs):
    print(f"开始任务：{task_name}")
    print(f"标签分类：{tags}")
    print(f"模型配置：{configs}")

fn("翻译", "NLP", "紧急", model="gpt-4", retry=3, stream=True)


# 输出结果：
# 开始任务：翻译 
# 标签分类：('NLP', '紧急') 
# 模型配置：{'model': 'gpt-4', 'retry': 3, 'stream': True}
```

## 闭包与装饰器
装饰器： 横向抽离代码，常用于*函数调用耗时、重试机制、身份验证等*

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print("函数开始运行")
        res = func(*args, **kwargs)
        print(f"运行结果： {res}")
        print("函数运行结束")
        return res
    return wrapper

@logger
def fn(a, b):
    return a + b

fn(1, 4)
```

## 异常处理
一般语言中都有异常处理机制
+ 捕获异常
+ 抛出异常 `raise Exception("xxx") --- throw new Error("xxx")`
+ finally 执行

`try-except-finally --- try-catch-finally`

```python
try: 
	res = 10 / 0
except ZeroDivisionError as e:
	# 指定错误类型
	print("不能除以0")
except Exception as e:
	# 捕获其他错误 类似于 catch(e)
	print(f"其他错误：{e}")
else: 
	# JS 没有这个！如果 try 块没有发生异常，就会执行这里
	print("没有错误")
finally: 
	# 无论是否报错都会执行（常用于关闭数据库连接、释放资源）
	print("清理工作完成")
```

# 模块化
与 JS 类似，每一个 `.py` 文件都会视为一个模块，可供其他文件导入

## 导入
```python
import os # 导入整个模块
import numpy as np # 起别名
from module import funcA, funcB <---> import {funcA, funcB} from "module"; 
```

## 导出
在 Python 中，一个 .py 文件里的*所有全局变量、函数、类*，默认都是可以被导出的

控制范围：若不想让使用者将私有函数导出，可加*下划线（私有）* 或在为文件顶部定义 `__all__` 列表

```python
# my_module.py
__all__ = ['public_func'] # 只有在这里面的才会被 import * 导出

def public_func(): pass
def _private_func(): pass # 下划线开头默认也是私有的
```