# javaScript

## 浏览器

![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240905122619592.png "null")![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739261494848-6eabe989-5563-46b4-a931-2e89435b5f4c.png)

## 数据类型

（栈）原始值：`Number String Boolean undefined null Symbol`

（堆）引用值：`Object Arrary Function Date...`

他两的区别就是一个按值传递，一个传递地址

这里就写点，我不知道的内容（或者比较模糊的东西）

### typeof()

判断是那个数据类型

【注意】

1. 这个判断之后，之后返回这几个类型`number string boolean object undefined function`值得注意的是：`typeof(null) 返回的是 object 的类型, typeof(NaN) --> number`
2. 返回的不管是表示什么类型，但是返回值一定是**String类型**的
3. 按常量，没有定义的变量会报错，但是`typeof(a);`里面的a可以是没有声明（这是唯一特殊的）

### 数据类型转化

这个是 js 特有的，下面有分为 **显示**类型转化 和 **隐式**类型转化

但是 其实 隐式类型转化也是调用了显示类型的函数

【注意】

### 显示类型转化：

1. `Number()` 将其他类型的值， 转化成数字

```js
let num = "10";
console.log(Number(num)); // 10

let num = "abc";
console.log(Number(num)); // NaN

let num = true / false;
console.log(Number(num)); // 1 / 0

let num = undefined;
console.log(Number(num)); // NaN 这个比较特殊

let num = null;
console.log(Number(num)); // 0
```

2. `ParseInt(String/Number, radix)`这个方法虽然也能转化成数字，但是只能接收 String、Number 类型的

```js
let num = 123.8;
console.log(ParseInt(num)); // 123 不会四舍五入 直接砍掉

let num = "123abc";
console.log(ParseInt(num)); // 123 返回非数字位前面的所有数字

let num = "123.58abc";
console.log(ParseInt(num)); // 123 也是返回非数字位前面的所有数字

let num = "a";
console.log(ParseInt(num, 16)); // 10 这表示 将num看成16进制的数，转成10进制的数输出
// 将某个十进制的数 转化成 规定的进制数
```

3. `ParseFloat(String/Number)` 这个转成浮点数
4. `toString(radix)` 将其他类型转化成字符串类型

```js
let num = 6;
console.log(num.toString(2)); // 110 将目标值看成10进制表示的数，转化成2进制数输出
// 将 10 进制的数，转化成想要的进制数

所以结合 parseInt 方法，就可以实现 2 ---> 10 ---> 16
比如：
let num = 111;
console.log(parseInt(num, 2).toString(16)); // 7
```

5. `String()` 转化成字符串类型
6. `Boolean()` 转化成布尔类型

```js
let boolean = "";
console.log(Boolean(boolean)); // false

let boolean = 0;
console.log(Boolean(boolean)); // false

let boolean = 1;
console.log(Boolean(boolean)); // true

let boolean = 1124 / "124";
console.log(Boolean(boolean)); // true

let boolean = undefined;
console.log(Boolean(boolean)); // false

let boolean = null;
console.log(Boolean(boolean)); // false
```

### 隐式类型转化

![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240905132255836.png "null")![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739261946850-dd5a02b7-d923-4b68-8c71-4c99b454a5dd.png)

1. `isNaN()` 判断传进来的数是不是**NaN**在这个方法里面其实就是调用了 Number() —> 在跟NaN比较 ，相等则返回 true 否则返回false![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240905131500790.png "null")

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739261962834-8b886766-1d6f-4318-8d11-a629dff25fbc.png)

2. 算术运算，报告 + - * % / > < >= <= ![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240905131511849.png "null")

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739261974866-c7e19172-fd2d-4bdf-bc4c-a87d0ebb94a1.png)

### 特殊的

```js
undefined  == null // true
NaN == NaN         // false 
```

说一下 == 和 === 的区别：

1. 首先，== 会进行隐式类型转化，只判断`值`是否相等`console.log(1 == "1"); true`
2. === 就不会进行隐式转化，等号两边长的不一样，就返回false 例如： `"1" === 1 false "1" == 1 true`

### 关于隐式赋值的题目

```js
var str = false + 1;
console.log(str); // 1
var demo = false == 1;
console.log(demo); // false
if (typeof (a) && -true + (+undefined) + "") {
    console.log(11);
}
if (11 + "11" * 2 == 33) {
    console.log(22);
}
!!" " + !!"" - !!false || console.log(33);
```

## 预编译

这个东西就解释了，变量提升和函数提升，等一系列诡异的东西（当然也只会出现在 var 定义的变量中）

1. 函数声明整体提升
2. 变量 声明提升

但，这两句话不是预编译的全部

现在，让我来解释一下，js中最最最炸裂的东西

首先，预编译 会分为 全局预编译 和 局部预编译

### 全局预编译

1. 创建 GO 对象，这里多说一点，就是 GO 可以认为是 window 对象
2. 扫描全局，将变量声明，作为 GO 的属性值，值为undefined
3. 找函数声明，值为函数体

上述过程中，如果有重复的变量或函数名，在 GO 对象中只会有最后执行的值

这里还要知道：`imply global 暗示全局变量`

也就是任何变量，如果变量未经声明就赋值，此变量就为全局对象所有（也就是 GO 相当于 window）。

### 局部预编译

与全局预编译大体相同，只不过多了一个步骤

1. 创建 AO 对象
2. 找形参和变量声明，将变量名和形参名作为 AO 对象的属性名，值为undefined
3. 将实参值和形参统一
4. 在函数体里面找函数声明，值为函数体

### 关于预编译题目

![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240906164836967.png "null")![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739262113201-9e6d0359-58e8-4969-97a6-98d1c92d6add.png)

![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240906164840456.png "null")![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739262122798-f0420579-8ad8-4b94-a007-e7fa919107a5.png)

![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240906164844643.png "null")![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739262132405-212a1683-b7bc-4d6e-9fe3-94cd1dcb862b.png)

![](C:\Users\晓兰\AppData\Roaming\Typora\typora-user-images\image-20240906164848212.png "null")![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1739262140046-64a05d3b-541c-4cbf-a3c3-26e58650bf7b.png)

# 函数

怎么说，js的一切都是对象（当然笼统的说是这样）

## 函数声明

```
function 函数名(){
    // 函数体
}
```

函数体的代码不会直接运行，必须要手动调用函数，才能运行其中的代码。

## 函数提升

其实最最最本质的原因，是因为js引擎会进行**预编译**

1. 通过字面量声明的函数，会提升到脚本块的顶部。
2. 通过字面量声明的函数，会成为全局对象的属性。  
    函数字面量就是：`` `function test()` ``这样声明的函数，会在window身上，也就是 window.test === test 为true

## 其他特点

通过typeof 函数名，得到的结果是"function"

函数内部声明的变量：

1. 如果不使用var声明，和全局变量一致，表示给全局对象添加属性
2. 如果使用var声明，变量提升到所在函数的顶部，函数外部不可以使用该变量  
    **函数中声明的变量，仅能在函数中使用，在外部无效**

```js
function test() {
    a = 10; // 没有用var声明，最终会加到 window 对象上
    console.log(b); // 变量提升，其实是预编译
    var b = 20;
}
test();
console.log(b); // 报错，其实是作用域
```

## 参数

如果实参没有传递，则对应的形参为 undefined

## 返回值

return 后面如果不跟任何数据，返回undefined

如果函数中没有书写return，则该函数会在末尾自动return undefined。

## 作用域和闭包

### 作用域

JS中，有两种作用域：总的就是一句话，外部的不能访问里面的，里面的可以访问外部环境的变量

1. 全局作用域(GO)  
    直接在脚本中书写的代码

在全局作用域中声明的变量，会被提升到脚本块的顶部，并且会成为全局对象的属性。

2. 函数作用域(AO)  
    函数中的代码

在函数作用域中声明的变量，会被提升到函数的顶部，并且不会成为全局对象的属性。  
**因此，函数中声明的变量不会导致全局对象的污染**

## 立即执行函数

当函数成为一个表达式时，它既不会提升，也不会污染全局对象。  
将函数变为一个函数表达式的方式之一，将函数用小括号括起来。

```js
// 函数表达式
var test = function() { 
    console.log(123);
} //匿名函数

(function test() {
    console.log(12);
})(); // 立即执行函数 IIFE
```

### 闭包

闭包（closure），是一种现象，内部函数，可以使用外部函数环境中的变量。

## 函数表达式和this

### 函数表达式

JS中，函数也是一个数据，语法上，函数可以用于任何需要数据的地方  
【注意】  
那么基于这个，函数的参数也可以是函数（回调函数就出现了）

```js
function test(a, callback) {
    console.log(a);
    callback();
}
test(1, function () { console.log(12); })
// 箭头函数引入之后，就可以简化为
test(1, () => console.log(12))
```

函数是一个引用类型，将其赋值给某个变量时，变量中保存的是函数的地址

### this关键字

this无法赋值, 也就是 this = {} false

在全局作用域中，this关键字固定指向全局对象。  
在函数作用域中，取决于函数是如何被调用的

- 函数直接调用，this指向全局对象
- 通过一个对象的属性调用，格式为对象.属性()或对象"属性"，this指向对象

```js
console.log(this); //指向 window 对象
function test() {
    console.log(this);
    // 函数被正常调用，那么this也指window
}
test(); // 类似于window.test();
var obj = {
    name: '张三',
    age: 18,
    sayHello: function () {
        console.log(this);
    }
}
obj.sayHello(); // 也可以 ob['sayHello']()
```

## 构造函数

在JS中，构造函数可以认为就是来便于创建对象的（当然是ES5之前），其实就认为它是类也可以

1. 函数名使用大驼峰命名法，例如：Number()、String()
2. 构造函数内部，会自动创建一个新对象，this指向新创建的对象，并且自动返回新对象
3. 构造函数中如果出现返回值，如果返回的是原始类型，则直接忽略；如果返回的是引用类型，则使用返回的结果
4. 所有的对象，最终都是通过构造函数创建的，例如：数组 new Array([1,2,3])、对象 new Object({name: '张三'})、函数 new Function()

```js
// 普通函数创建对象
function createUser(name, age) {
    return {
        name,
        age,
        sayHello() {
            console.log(`我是${this.name}, 年龄是${this.age}`);
        }
    }
}
createUser('张三', 18).sayHello();

// 构造函数创建对象
function User(name, age) {
    // 构造函数底层，会自动创建一个新对象，this指向新对象，并且自动返回对象
    // console.log(this); // User {}
    this.name = name; // 为新对象添加属性, 其实是 User {name: '张三'}
    this.age = age;
    this.sayHello = function () {
        console.log(12);
    }
    // return {}; 构造函数本身就返回了一个对象，如果手动返回一个引用类型会覆盖那个新对象，如果手动返回一个原始值，那就无所谓
}
var user = new User('张三', 18);
// var user = User('张三', 18); // 当作普通函数来调用，所有的this都指向 window，user 为 undefined，因为普通函数中没有写return语句的话，js会自动加上 return undefined
user.sayHello()
```

【注意】：

1. 只有new出来的，才是对象  
    我们写的代码，都是语法糖. var obj = {} <=> var obj = new Object(); 其他的都类似(都是经过装箱的)，特别注意：function test() {} <=> new Function() {}，所以函数也是new出来的，test是Function的对象
2. 所有函数都可以用 new 关键字调用，返回的值是其本身(当然是指没有return的时候)  
    举个例子：

```js
function test() {
    console.log(12)
}
var obj = test(); // 这个obj就不是test()的对象
console.log(test()); // test() f

// 如果有返回值的话，如下所示：
function test() {
    return {};
}
console.log(test()); // {}
```

## 函数的本质

函数的本质就是对象。我算知道了，JS中真是一切都是对象啊！！！！

Function  
所有的函数，都是通过new Function()创建。

由于函数本身就是对象，因此函数中，可以拥有各种属性。

```js
function User() {
    console.log(12);
}
User.a = 10; // 其实普通函数也是一样的
User.hello = function() {
    console.log('hello')
}
```

【注意】  
JS开始玩抽象了，其实在ES5之前，可以认为构造函数就是类，加上函数本质是对象，那么静态方法（类方法）、静态属性、成员方法什么的就出现了  
比如：

```js
var n = 3.1415; --> 其实这里进行了自动装箱（当然这个是JAVA中的概念）
// 上面的语句就类似于 var n = new Number(3.1415);
// 也就是说 n 可以被理解为 对象，Number 可以被理解为类
console.log(n.toFixed(2)); // toFixed 成员方法
console.log(Number.isNaN(NaN)); // isNaN 静态方法 
console.log(isNaN(n)); 
// 那么为什么直接用也可以呢？ 因为JS为了方便开发者使用，将一些函数、属性放在了 window 身上
```

### 包装类

JS为了增强原始类型的功能，为boolean、string、number分别创建了一个构造函数：

1. Boolean
2. String
3. Number

自动装箱  
如果语法上，将原始类型当作对象使用时（一般是在使用属性时），JS会自动在该位置利用对应的构造函数，创建对象来访问原始类型的属性。

# 标准库（标准API）

## Object

### 静态成员

1. keys(某个对象)，得到某个对象的所有属性名**数组**
2. values(某个对象)，得到某个对象的所有属性值**数组**
3. entries(某个对象)，得到某个对象的所有属性名和属性值的**数组**

### 实例成员

实例成员可以被重写

所有对象，都拥有Object的所有实例成员(类似于Java中的顶级类)

- toString方法：得到某个**对象**的字符串格式  
    默认情况下，该方法返回"[object Object]";
- valueOf方法：得到某个对象的值  
    默认情况下，返回该对象本身  
    【注意】如果转化的是原始值，那么就会自动装箱

在JS中，当自动的进行**类型转换**时，如果要对一个对象进行转换，实际上是先调用对象的valueOf方法，然后调用**返回结果**的toString方法，将得到的结果进行进一步转换。

【注意】  
如果调用了valueOf已经得到了原始类型，则不再调用toString  
例如：

```js
 var obj = {
        x: 13,
        y: 34534,
        valueOf() {
            return 123;
        }
    }
console.log(obj + 1); // 124
// 不会进行下面的操作
// console.log(obj.valueOf().toString() + 1);
```

## Function

**所有函数都具有Function中的实例成员**

语法：arguments：在函数中使用，获取该函数调用时，传递的所有参数

arguments是一个类数组（也称为伪数组：没有通过Array构造函数创建的类似于数组结构的对象），伪数组会缺少大量的数组实例方法

arguments数组中的值，会与对应的**形参映射**  
例如：

```js
function sum(a, b) {
    console.log(arguments);
    // arguments 跟形参会有映射关系
    // 如果有有一个参数没有传参，那么就没有射关系，但是如果传的是undefined那么也会有映射
    arguments[0] = 'abs'
    arguments[1] = 12
    return a + b;
}
console.log(sum(undefined, 1)); // abs12
```

### 实例成员

1. length属性，得到函数形参数量
2. apply方法：调用函数，同时指定函数中的this指向，参数以**数组**传递(this指向使用一次)
3. call方法：调用函数，同时指定函数中的this指向，参数以**列表**传递(this指向使用一次)
4. bind方法：得到一个新函数，该函数中的this始终指向指定的值(永久使用)

`` `A.call(B) 这句话的意思就是，将A中的this指向修改为B` ``

```js
var obj = {
    name: '张三',
    age: 18
}
function sum(a, b) {
    // 没有指定this指向时，在全局调用的函数的this指向window
    console.log(this); // 指向obj
    return a + b;
}
console.log(sum.apply(obj, [1, 3]));
console.log(sum.call(obj, 1, 3));
```

## 伪数组->真数组

### 方法一：

**通常，可以利用apply、call方法，将某个伪数组转换伪真数组**

```js
function test() {
    console.log(arguments);
    var newArr = [].slice.apply(arguments)
    // 更好的方法是
    var newArr = Array.prototype.slice.call(arguments).slice(1)
    console.log(newArr);
}
test(1, 2, 3, 4)
/*
    [].slice.apply(arguments)  解释说明

    1. [].slice 创建了一个slice函数的引用。
    2. apply 方法将slice函数调用时this的值设置为arguments对象，并将arguments对象作为参数数组传递给slice函数。
    3. slice函数从arguments对象中提取所有参数，并返回一个新的数组。
 * /
```

### 方法二：

调用数组的静态方法 from  
Array.from(arguments);

## Array

**凡是通过Array构造函数创建的对象，都是数组**

### 静态成员

- from方法：可以将一个伪数组转换为真数组
- isArray方法：判断一个给定的数据，是否为一个**真数组**
- of方法：类似于中括号创建数组，依次赋予数组每一项的值

### 实例成员

- fill方法：用某个数据填充数组
- pop: 删除最后一个元素并返回
- push：在数组末尾加入一个元素
- reverse：将当前数组颠倒顺序
- shift：移除第一个元素并返回
- sort：对数组进行排序
- splice：添加、删除、替换元素
- unshift：在数组前面添加元素

**纯函数、无副作用函数：不会导致当前对象发生改变**

- concat：连接一个或多个数组
- includes: 数组中是否包含满足条件的元素
- join：将数组转化为字符串
- slice：切片(包头不包尾)
- indexOf：返回查找指定元素的下标（从前往后找）
- lastIndexOf：从后往前找
- forEach: 遍历数组
- every：是否所有元素都满足条件(返回布尔值)
- some：是否至少有一个元素满足条件(返回布尔值)
- filter：过滤，得到满足条件的元素组成的新数组
- find: 查找第一个满足条件的元素，返回元素本身，如果没有找到，返回undefined
- findIndex: 查找第一个满足条件的元素，返回元素的下标
- map：映射，将数组的每一项映射称为另外一项
- reduce：统计，累计

## 内置对象(包装类)

【注意】

1. new 包装器(值)：返回的是一个对象
2. 包装器(值)：返回的是一个原始类型

### Number

**静态方法**

- isNaN：判断是不是NaN
- isFinite：判断一个值是不是一个有限数（只有判断Infinity是false）
- isInteger：判断一个数据是否是整数
- parseFloat: 将一个数据转换为小数
- parseInt：将以一个数据转换为整数，直接舍去小数部分

parseInt、parseFloat要求参数是一个**字符串**，如果不是字符串，则会先转换为**字符串**。 从字符串开始位置进行查找，找到第一个有效的数字进行转换，**如果没有找到，则返回NaN，左右空白字符会忽略**

parseInt，可以传入第二个参数，表示将给定的字符串，识别为多少进制，返回十进制数字。  
例如：  
parseInt(101, 2) -> 5  
parseInt('a', 16) -> 10  
parseInt(103, 2) -> 2 注意，转化的值是有效的值，如果没有有效值，返回的也是NaN

**实例方法**

- toFixed方法：会有四舍五入
- toPrecision：以指定的精度返回一个数字字符串

### Boolean

会对参数进行布尔判定  
除了下面这些会判定为false，其他的会判定为true

1. 空
2. ""
3. null
4. undefined
5. 0
6. false  
    【注意】  
    new Boolean([]) new Boolean({}) 都判定为true

### String

**静态成员**  
fromCharCode：通过unicode编码创建字符串

**实例成员**  
length：字符串长度  
字符串是一个**伪数组**

- charAt：得到指定位置的字符，如果超出字符串范围，返回空串''
- charCodeAt：得到指定位置字符的编码，如果超出字符串范围，返回NaN
- concat：连接一个或多个字符串
- includes：判断某个指定条件的元素是否在字符串中
- endsWith：判断字符串是否由某个元素结尾
- startsWith：判断字符串是否由某个元素开头
- indexOf：查询某个元素在字符串中的位置（从左往右）
- lastIndexOf：从右往左
- padStart：按指定的字符来填充字符（在字符串开头填充）例如：padStart(2,'0') 表示 如果字符串长度没有2，那么在该字符串前面用 '0' 来填充
- padEnd：在字符串末尾填充
- repeat：让字符串重复几次
- slice：从某个位置取到**某个位置**；位置可以是负数；
- substring：从某个位置取到某个位置；不可以是负数；参数位置是可调换的
- toLowerCase：转小写
- toUpperCase：转大写
- split：分割字符串
---
关于查找子串：查找的子串只要有一部分在模版串中
例如，
`"mongo-713d3062-configsvr0-0-0".includes("7") -> true`
- includes("") 返回 true，否则返回 false
- indexOf("") 返回下标，否则返回 -1
---
如果需要精确匹配，则需要正则
- `"mongo-713d3062-configsvr0-0-0".serach(/mongo-713d3062-configsvr0/)` 返回子串的起始下标，否则返回 -1

## Math对象

基本上都是一些数学方法

### 实例方法

- random方法: 产生一个0~1之间的随机数，注意，不能获得1，只能无限接近

```js
  // 随机生成 max - min之间的整数
  function myRandom(min, max) {
      return Math.floor(Math.random() * (max - min) + min)
  }
```

- abs方法：求绝对值
- floor方法：对一个数向下取整，特别留意一下负数的情况，Math.floor(-3.1) --> -4
- ceil方法：对一个数向上取整，Math.ceil(-3.1) --> -3
- max方法：得到一组数字的最大值；如果无参，得到-Infinity，  
    【特别注意】：一组数字 1,2,3 !== 数字数组 [1,2,3]
- min方法：得到一组数字的最小值；如果无参，得到Infinity
- pow方法：求一个数字的幂次方，平方可以用 `**2` 来表示，比如，3的平方 --> `3**2`或 `Math.pow(2,3)`
- round方法：得到一个四舍五入的整数，也要注意负数的情况，当小数部分的值大于0.5那么就往小的方法约  
    例如：Math.round(-1.5) --> -1 Math.round(-1.6) --> -2

## Date对象

### 创建时间对象

- 直接调用函数（不适用new），忽略所有参数，直接返回当前时间的字符串。
- new Date(): 创建日期对象

1. 无参，当前时间
2. 1个参数，参数为数字，表示传入的是时间戳
3. 两个参数以上，分别表示：年、月、日、时、分、秒、毫秒  
    【注意】：月份的数字从0开始计算。

如果缺失参数，日期部分默认为1，时分秒毫秒默认为0。

月、日、时、分、秒、毫秒，均可以传递负数，如果传递负数，会根据指定日期进行计算。

### 部分实例方法

都是配套的方法，重写了valueOf()方法，可以进行数学计算

- getDate方法：得到日期部分，就是几号
- getDay方法：得到星期几，0表示星期天
- getFullYear方法：得到年份
- getHours方法：得到小时部分
- getMinutes方法：得到分钟部分
- getTime方法：得到时间戳
- setTime方法：重新设置时间戳
- toDateString方法：将日期部分转换为可读的字符串。
- toLocaleDateString方法：根据当前系统的地区设置，将日期部分转换为可读的字符串（'2024/11/3）
- toLocaleString方法：根据当前系统的地区设置，将整个日期对象转换为可读的字符串（'2024/11/3 21:13:57'）
- toLocaleTimeString方法：根据当前系统的地区设置，将时间部分转换为可读的字符串（'21:13:57'）

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748509569861-e7801401-57d5-4b0a-bd3a-c04936accd4f.png)

## 正则表达式

## 异常机制

JS中的错误分为：

1. 语法错误：会导致整个脚本块无法执行。
2. 运行错误  
    运行报错：会导致当前脚本块后续代码无法执行  
    运行结果不符合预期

### 调试错误

1. 控制台打印 用console.log
2. 断点调试 vsCode中的调式工具或浏览器F12中的source

### 抛出错误

错误在js中本质上是一个对象，抛出错误的语法为：

```
throw 错误对象;
错误对象的构造函数为Error
```

### 捕获错误

```js
try{
    //代码块1
}
catch(错误对象){
    //代码块2
}
finally{
    //代码块3
}
```

当运行代码1的时候，如果发生错误，立即停止代码1的执行，转而执行代码2，错误对象为抛出的错误对象。无论代码1和代码2是否，最终都将执行代码3

# web api

标准库：ECMAScript中的对象和函数  
Web Api：浏览器宿主环境中的对象和函数

Web Api：

- BOM：Browser Object Model，浏览器对象模型
- DOM：Document Object Model，文档对象模型  
    BOM：控制浏览器本身 DOM：控制HTML文档

ES 由 ECMAScript 规定的 WebApi 由 W3C（万维网联盟） 制定

## DOM是什么

DOM的核心理念，是将一个HTML或XML文档，用对象模型表示，每个对象称之为dom对象

dom对象又称之为节点Node

节点的类型：

- DocumentType，文档类型节点
- Document，文档节点，表示整个文档
- Comment，注释节点
- Element，元素节点
- Text，文本节点
- Attribute，属性节点
- DocumentFragment，文档片段节点  
    dom树：文档中不同的节点形成的树形结构

## 获得DOM节点

获取dom对象

全局对象 window 中有属性document，代表的是整个文档节点

document.body：获取body元素节点  
document.documentElement: 获取根元素 -> html

### 新的获取元素节点的方式

**通过方法获取**

- document.getElementById：通过id获取对应id的元素
- document.getElementsByTagName: 通过元素名称获取元素
- document.getElementsByClassName：通过元素的类样式获取元素，IE9以下无效
- document.querySelector：通过CSS选择器获取元素，得到匹配的第一个，IE8以下无效
- document.querySelectorAll：通过CSS选择器获取元素，得到所有匹配的结果，IE8以下无效

【注意】

1. 在所有的得到类数组的方法中，除了querySelectorAll，其他的方法都是实时更新的。
2. getElementById 得到元素执行效率最高。
3. 书写了id的元素，会自动成为window对象的属性。它是一个实时的单对象。不推荐使用。
4. querySelector、querySelectorAll，可以作为其他元素节点对象的方法使用。  
    意思就是，

```js
   var ul = document.querySelector('ul')
   var lis = ul.querySelectorAll('li')
```

### 根据节点关系获取节点

这个会获得那些文本节点、注释节点，一般不用

这些都是属性，不是方法

- **parentNode**：获取父节点（元素、文档）
- previousSibling：获取上一个兄弟节点
- nextSibling：获取下一个兄弟节点
- childNodes：获取所有的子节点（实时的）
- firstChild：获取第一个子节点
- lastChild：获取最后一个子节点
- attributes: 获取某个元素的属性节点

### 获取元素节点

一般用这个来获取元素

这些都是属性，不是方法

- parentElement：获取父元素
- previousElementSibling：获取上一个兄弟元素
- nextElementSibling：获取下一个兄弟元素
- children：获取子元素
- firstElementChild：获取第一个子元素
- lastElementChild：获取最后一个子元素

### 获取节点信息(了解)

- nodeName：获取节点名称
- nodeValue：获取节点的值
- nodeType：节点类型，是一个数字

## dom元素操作

### 初识元素事件

元素事件：某个元素发生一件事（被点击 click）

事件处理程序：是一个函数，发生了一件事，应该做什么事情

注册事件：将事件处理程序与某个事件关联

**this关键字在事件处理程序中指代当前发生的事件元素**

### 获取和设置元素属性
- 获取所有的属性：`attributes` 配合 `for.. of` 来遍历所有的属性值
- 通用方式：`getAttribute、setAttribute`  
    一般*设置*自定义属性 `lis[0].setAttribute('data-abx', 123)`  
    *获取*属性 `lis[0].getAttribute('data-abx') // 123`  
    **可识别属性**  
    正常的HTML属性
- dom对象.属性名：推荐  
    例如：`lis[0].innerHTML` 或 `lis[0].style`  
    【注意】

1. 正常的属性即使没有赋值，也有默认值
2. 布尔属性在dom对象中，得到的是boolean  
    例如：checked、selected等  
    `document.querySelector('input[type=checkbox]').checked   `
3. 某些表单元素可以获取到某些不存在的属性  
    比如：文本域本来是没有 value 值的，但是为了好获得文本域中的内容，就给它设置了value值
4. 某些属性与标识符冲突，此时，需要更换属性名  
    比如，class 属性就改成了 className

**自定义属性**  
HTML5 建议自定义属性使用data-作为前缀

如果遵从HTML5 自定义属性规范，可以使用dom对象.dataset.属性名控制属性（输出就是一个对象）

删除自定义属性，但是不能删除可识别属性，说白了就只能删除自定义属性

- removeAttribute("属性名");
- delete dom.dataset.属性名（推荐）

### 获取和设置元素内容

- innerHTML：获取和设置元素的内部HTML文本
- innerText：获取和设置元素内部的纯文本，仅得到元素内部**显示**出来的文本，如果没有在浏览器上面显示，那么就不能获得
- textContent：获取和设置元素内部的纯文本，textContent得到的是内部**源代码**中的文本

### 元素结构重构

- 父元素.appendChild(元素)：在某个元素末尾加入一个子元素
- 父元素.insertBefore(待插入的元素, 哪个元素之前)
- 父元素.replaceChild(替换的元素, 被替换的元素)

【注意】  
更改元素结构效率较低，尽量少用。

### 创建和删除元素

#### 创建元素

- document.createElement("元素名")：创建一个元素对象

- document.createTextNode("文本")
- document.createDocumentFragment(): 创建文档片段，可以将一串创建好的元素对象，放到这个片段中，然后在一起加入到父元素中的某个位置

- dom对象.cloneNode(是否深度克隆)：复制一个新的dom对象并返回  
    例如：uls.getElementsByTagName('li')[1].cloneNode(true) true表示深度克隆，会将被克隆对象中的内容(源代码内容)也克隆下来

#### 删除元素

- 父元素.removeChild(子元素dom对象) -> "他杀"
- 子元素.remove() -> "自杀"

## dom元素样式

### 控制dom元素的类样式

- tagName: 获得元素大写名称
- className： 获取或设置元素的类名
- classList： dom4的新属性，是一个用于控制元素类名的对象

1. add：用于添加一个类名
2. remove：用于移除一个类名
3. contains：用于判断一个类名是否存在
4. toggle：用于添加/移除一个类名

### 获取样式

**CSS的短横线命名，需要转换为小驼峰命名**  
例如：background-color -> backgroundColor

- dom.style：得到**行内样式**对象(得到的是未计算的值，源代码是怎样的就是怎样的)
- window.getComputedStyle(dom元素)：得到某个元素最终计算的样式(得到的都是就计算后的值，都是绝对单位)

可以有第二个参数，用于得到某个元素的某个伪元素样式

### 设置样式

`dom.style.样式名 = 值`

设置的是行内样式

# dom对象

## 事件流

**事件冒泡**：先触发最里层的元素，然后再依次触发外层元素  
**事件捕获**：先触发外层的元素，然后再依次触发里面元素

默认情况下，事件是冒泡的方式触发

事件源、事件目标：事件目标阶段的元素

## 事件注册

事件绑定

### dom0

将事件名称前面加上on，作为dom的属性名，给该属性赋值为一个函数，即为事件注册。

```js
// 一个元素只能注册一个事件
btn.onclick = function() {
    console.log('dom0')
}

// 移除事件
btn.onclick = null;
```

移除：重新给事件属性赋值，通常赋值为null和undefined

### dom2

dom对象.addEventListener：注册事件

```js
btn.addEventListener('click', () => {
    console.log('btn');
}, {
    capture: true,
    once: true
})
```

与dom0的区别

1. dom2可以为某个元素的同一个事件，添加多个处理程序，按照注册的先后顺序运行
2. dom2允许开发者控制事件处理的阶段，使用第三个参数，表示是否在捕获阶段触发

如果元素是目标元素（事件源），第三个参数无效  
事件的移除：dom对象.removeEventListener(事件名, 处理函数);

dom2中如果要移除事件，不能使用匿名函数

【注意】  
添加和移除事件时，可以将第三个参数写为一个对象，进行相关配置

## 事件对象

事件对象封装了事件的相关信息

### 获取事件对象

- 通过事件处理函数的参数获取
- 旧版本的IE浏览器通过window.event获取

### 事件对象的通用成员

- target & srcElement  
    事件目标（事件源）

事件委托：通过给祖先元素注册事件，在程序处理程序中判断事件源进行不同的处理。`e.target.tagName === 'BUTTON'`

通常，事件委托用于**动态**生成元素的区域。

- currentTarget  
    当前目标：获取绑定事件的元素，等效于this
- type  
    字符串，得到事件的类型
- preventDefault & returnValue  
    preventDefault方法  
    阻止浏览器默认行为  
    dom0的方式：在事件处理程序中返回false

```js
 <div class="box">
    <a id="a" href="http://www.baidu.com" onclick="return false">百度</a>
    </div>
<script>
    a.onclick = function (e) {
        e.preventDefault();
    }
</script>
```

- stopPropagation方法  
    阻止事件冒泡
- eventPhase  
    得到事件所处的阶段

1： 事件捕获 2： 事件目标 3： 事件冒泡

## 鼠标事件

### 事件类型

- click：用户单击主鼠标按钮（一般是左键）或者按下在聚焦时按下回车键时触发
- dblclick：用户双击主鼠标按键触发（频率取决于系统配置）
- mousedown：用户按下鼠标任意按键时触发
- mouseup：用户抬起鼠标任意按键时触发
- mousemove：鼠标在元素上移动时触发
- mouseover：鼠标进入元素时触发
- mouseout：鼠标离开元素时触发
- mouseenter：鼠标进入元素时触发，该事件不会冒泡
- mouseleave：鼠标离开元素时触发，该事件不会冒泡  
    区别：

1. over和out，不考虑子元素，从父元素移动到子元素，对于父元素而言，仍然算作离开
2. enter和leave，考虑子元素，子元素仍然是父元素的一部分
3. mouseenter和mouseleave不会冒泡

### 事件对象

所有的鼠标事件，事件处理程序中的事件对象，都为 MouseEvent

- altKey：触发事件时，是否按下了键盘的alt键
- ctrlKey：触发事件时，是否按下了键盘的ctrl键
- shiftKey：触发事件时，是否按下了键盘的shift键
- button：触发事件时，鼠标按键类型

- 0：左键
- 1：中键
- 2：右键

### 位置：

1. page：pageX、pageY，当前鼠标距离**页面**的横纵坐标
2. client: clientX、clientY，鼠标相对于**视口**的坐标  
    【注意】：page 与 client 的区别：如果没有滚动条的话，两个值是一样的，client是固定的
3. offset：offsetX、offsetY，鼠标相对于事件源的内边距的坐标(建议不用)
4. screen: screenX、screenY，鼠标相对于**电脑屏幕**
5. x、y，等同于clientX、clientY
6. movement：movementX、movementY，只在鼠标移动事件中有效，相对于上一次鼠标位置，偏移的距离

## 键盘事件

### 事件类型

- keydown：按下键盘上任意键触发，如果按住不放，会重复触发此事件
- keypress：按下键盘上一个字符键时触发
- keyup：抬起键盘上任意键触发
- keydown、keypress 如果阻止了事件默认行为，文本不会显示。

### 事件对象

KeyboardEvent

- code：得到按键字符串，适配键盘布局
- key：得到按键字符串，不适配键盘布局。能得到打印字符

## 表单事件

- focus：元素聚焦的时候触发（能与用户发生交互的元素，都可以聚焦），该事件不会冒泡
- blur：元素失去焦点时触发，该事件不会冒泡。
- submit：提交表单事件，仅在form元素有效。
- change：文本改变事件
- input: 文本改变事件，即时触发

## 其他事件

window全局对象

- load、DOMContentLoaded、readystatechange  
    window的load：页面中所有资源全部加载完毕的事件 图片的load：图片资源加载完毕的事件

浏览器渲染页面的过程：

1. 得到页面源代码
2. 创建document节点
3. 从上到下，将元素依次添加到dom树中，每添加一个元素，进行预渲染
4. 按照结构，依次渲染子节点  
    document的DOMContentLoaded: dom树构建完成后发生

readystate: loading(等待阶段)、interactive、complete

interactive(交互阶段)：触发DOMContentLoaded事件

complete(完成阶段)：触发window的load事件

**js代码应该尽量写到页面底部**

- css应该写到页面顶部：避免出现闪烁（如果放到页面底部，会导致元素先没有样式，使用丑陋的默认样式，然后当读到css文件后，重新改变样式）
- JS应该写到页面底部：避免阻塞后续的渲染，也避免运行JS时，得不到页面中的元素
- unload、beforeunload(了解即可)

beforeunload: window的事件，关闭窗口时运行，可以阻止关闭窗口 unload：window的事件，关闭窗口时运行

- scroll  
    窗口发生滚动时运行的事件

通过scrollTop和scrollLeft，可以获取和设置滚动距离  
scrollHeight / scrollWidth：整个内容的高度，包括隐藏的内容和滚动条

```js
document.querySelector('p').onscroll = function() {
    // 注意，这两个属性不是 e 事件对象中的属性
    console.log(this.scrollTop, this.scrollLeft);
}
```

- resize  
    窗口尺寸发生改变时运行的事件，监听的是视口尺寸
- contextmenu  
    右键菜单事件，相当于规定了 e.butto=n = 2

```js
document.querySelector('div').oncontextmenu = function (e) {
    // 阻止默认行为，右键不能出现菜单
    e.preventDefault();
}
```

- paste  
    粘贴事件

```js
document.querySelector('textarea').onpaste = function (e) {
    // 阻止默认行为，区域不能粘贴东西
     e.preventDefault();
}
```

- copy  
    复制事件

```js
document.querySelector('textarea').oncopy = function (e) {
    // 阻止默认行为，区域不能复制东西
    e.preventDefault();
}
```

- cut  
    剪切事件

```js
document.querySelector('textarea').oncut = function (e) {
    // 阻止默认行为，区域不能剪切东西
    e.preventDefault();
}
```

# 计时器

计时器是异步的，当时机成熟之后才会执行

计时器会返回一个数字，该数字表示计时器的编号

- setTimeout方法：指定时间到达后运行某个函数
- clearTimeout方法：清除计时器
- setInterval方法：指定间隔时间到达后运行某个函数
- clearInterval方法：清除计时器

# window对象

## 自身方法

这些方法了解即可，都不用了

- open：打开一个新窗口 open("页面路径", "打开目标", "配置")
- alert
- confirm
- prompt

## 对象属性

- location：地址栏对象  
    href属性：得到目前地址 其他属性参考location.jpg  
    reload方法：强制刷新当前页面
- navigator：大部分的方法，浏览器都不放正确的内容，所以页没用
- history：历史记录  
    go方法 back方法 forword方法
- console  
    log方法：打印对象的valueOf的返回值  
    dir方法：打印对象结构  
    tiem方法和timeEnd方法：用于计时(一般成对出现)

# 原型和原型链

前提知识：

- 所有对象都是通过new 函数创建
- 所有的函数是对象
- 函数中可以有属性(因为函数是对象嘛~)
- 所有对象都是引用类型

## 原型 prototype

所有函数都有一个属性：prototype，称之为**函数原型**，也可以称为**显示原型**

默认情况下，prototype是一个普通的Object对象

默认情况下，prototype中有一个属性，constructor，它也是一个对象，它指向**构造函数本身**。  
即：`` `test.prototype.constructor === test` `` 成立

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751032905757-5a8aa3fa-fea5-485f-be61-1e7e6ad2722f.png)

## 隐式原型 __proto__

所有的对象都有一个属性：__**proto__**，称之为**隐式原型**

默认情况下，隐式原型指向**创建该对象的函数的原型**  
比如说：

```js
var obj = {} // <=> var obj = new Object()
console.log(obj.__proto__ === Object.prototype); // true
```

猴子补丁：在函数原型中加入成员，以增强起对象的功能，猴子补丁会导致原型污染，使用需谨慎。  
例如：

```js
// 这样操作之后，所有的数组对象都可以用 sum 函数
Array.prototype.sum = function(a, b) {
    return a + b;
}
```

## 原型链

当访问一个对象的属性或方法时，JS 的查找方式：

1. 先看对象本身有没有，有 -> 直接用
2. 没有 -> 沿着对象的隐式原型 (__proto__)向上查找，直到找到 Object 的原型，

如果还没有找到那么返回 undefined

特殊点：

1. Function的__proto__指向自身的prototype
2. Object的prototype的__proto__指向null
3. 所有函数原型的隐式原型 Function.prototype.__proto__ 都指向 Object.prototype

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751032719330-3fa64344-8060-4c1a-b04d-e16a2071dc45.png)

# 原型链的应用

## 基础方法

1. W3C不推荐直接使用系统成员__proto__，获得某个对象的隐式原型

Object.getPrototypeOf(对象)

```js
function test() {
}
var o = new test();
// 得到o的隐式原型，类似于 __proto__
console.log(Object.getPrototypeOf(o)); // {}
```

2. 判断当前对象(this)是否在指定对象的原型链上

Object.prototype.isPrototypeOf(对象)

```js
console.log(Object.prototype.isPrototypeOf(o)); // true
```

3. 判断函数的原型是否在对象的原型链上

对象 instanceof 函数

```js
o instanceof Object // true  Object的原型是不是在o的原型链上
o instanceof Array // false
```

4. 创建一个**新对象**，其隐式原型指向指定的对象

Object.create(对象)

```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayHello = () => {
  console.log("hello");
};

const p = new Person("张三", 18);
p.sayHello(); 
const p1 = Object.create(p);
p1.sayHello();
console.log(Object.getPrototypeOf(p1) === p); // true
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751034824256-82103d6a-120c-45a6-a051-4e7000cd8b1e.png)

5. 判断一个对象自身是否拥有某个属性

Object.prototype.hasOwnProperty(属性名)

```js
var p = {
    x: 1,
    y: 23
}
var obj = Object.create(p); // obj的隐式原型指向 p
obj.abx = '123';
console.log(obj.hasOwnProperty("abx")); // true

for (var item in obj) {
    // 如果不进行处理的话，就将原型上的属性也打印出来
    // 加上判断，输出的值都是 obj 自己拥有的属性
    if (obj.hasOwnProperty(item)) {
        console.log(item);
    }
}
```

## 应用

1. 类数组转换为真数组

Array.prototype.slice.call(类数组);

```js
function sum() {
    // 表示，slice中的this指向都指向 arguments
    var arr = Array.prototype.slice.call(arguments)
    console.log(arr);
}
sum(1, 2, 3, 4)
```

2. 实现继承

默认情况下，所有**构造函数**的父类都是Object

圣杯模式

```js
this.inherit = (function () {
    var Temp = function () { };
    return function (son, father) {
        Temp.prototype = father.prototype;
        son.prototype = new Temp();
        son.prototype.constructor = son;
        son.prototype.uber = father;
    }
}());


function inheritPrototype(son, father) {
    var temp = Object.create(father.prototype);
    temp.constructor = son;
    son.prototype = temp;
}
```