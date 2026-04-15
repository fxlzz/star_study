## 运行 TS

当然一般框架在脚手架中，就全部配置好了，不需要单独配置

1、全局下载 TS

```bash
npm i typescript -g
```

2、初始化项目

```bash
tsc --init
```

3、使用 tsc 命令  
自动运行 .ts 的脚本

```bash
tsc --watch
```

## 基本类型

JS 的基本类型：8个基本类型  
number string null undefined boolean object bigint symbol 当然object还包括Array String Function...

TS 的基本类型：
1. 包含上述 JS 的基本类型
2. 六个新类型：  
    any unknown never void tuple enum
3. 两个自定义类型：  
    type interface

## any
见名知意，就是什么类型都可以
注意：any类型变量，可以赋给任意类型
```ts
// 明确的表示a的类型是 any —— 【显式的any】
let a: any
// 以下对a的赋值，均无警告
a = 100
a = '你好'
a = false
// 没有明确的表示b的类型是any，但TS主动推断出来b是any —— 【隐式的any】
let b
//以下对b的赋值，均无警告
b = 100
b = '你好'
b = false
```

## unknown

意思就是，不知道的，常用于一开始不知道是什么类型，后期才会知道（**使用属性必须加断言**）  
可以认为是安全的any类型

```ts
// 设置a的类型为unknown
let a: unknown
//以下对a的赋值，均符合规范
a = 100
a = false
a = '你好'
```

【注意】：不能将未处理的unknown类型赋给其他已知的类型

```ts
// 设置x的数据类型为string
let x: string
x = a //警告：不能将类型“unknown”分配给类型“string”
```

解决方法：加断言（两种写法）

```ts
x = a as string // 或 x = <string>a
```

## never

never 的含义是：任何值都不是，即：不能有值，例如undefined 、nul1、、0都不行！

【注意】：几乎不用never 去直接限制变量，因为没有意义

如下面写的，没有意义：

```ts
/* 指定a的类型为never，那就意味着a以后不能存任何的数据了 */
let a: never
// 以下对a的所有赋值都会有警告
a = 1
a = true
a = undefined
a = null
```

never 类型，一般都是由 ts 自己推断出来的

【注意】：never 类型可以去限制 函数的返回值 一般用于**抛出异常**

```ts
// 限制throwError函数不需要有任何返回值，任何值都不行，像undeifned、null都不行
function throwError(str: string): never {
    throw new Error('程序异常退出:' + str)
    //... 下面的语句不会执行
}
```

## void

void的含义是空，即：函数不返回任何值，调用者也**不应依赖**其返回值**进行任何操作**！  
【注意】：**undefined 是 void 可以接受的一种“空”**  
下面的写法都是对的：在 JS 中，如果函数没有显示返回某个东西，就会隐式返回undefined

```ts
function logMessage(msg:string):void{
    console.log(msg)
}
// 无警告
function logMessage(msg:string):void{
    console.log(msg)
    return;
}
// 无警告
function logMessage(msg:string):void{
    console.log(msg)
    return undefined
}
```

## 声明对象类型

这里提一嘴 object 和 Object，因为限制返回太大了，所以一般不用  
object： 限制非原始类型，像数组、函数、对象啥的  
Object： 除了 undefined 和 null 都可以限制

在开发中，限制对象，一般使用以下的形式：

### 方法一：直接定义

```ts
// age?: number 可选参数
let person : {name: string, age?: number}
person = {name: "张三", age: 18}
person = {name: "李四"}

person = {name: "王五", gander: "男"} // 非法 因为没有对 gender 进行限制
```

### 方法二：索引签名

使用**索引签名**：可以动态的规定对象的属性，**键**和**类型**都是可以变化的  
例如：

```ts
let person: {
    name: string,
    age?: number,
    [key: string]: any // 索引签名
}

person = {name: "王五", gander: "M"}
```

## 声明数组类型

### 方法一：类型[] 或 泛型

数组里面的类型是一个，不能有其他的数据类型

```ts
let arr1: string[]
let arr2: Array<string> // 泛型
arr1 = ['a','b','c']
arr2 = ['hello','world']
```

### 方法二：使用 tuple 类型

元组(Tuple )是一种特殊的数组类型，可以存储固定数量的元素，并且每个元素的类型是已知的且可以不同。元组用于精确描述一组值的类型.

```ts
let arr1 : [string, number]
arr1 = ["hello", 12] // 不能多也不能少，必须一一对应
let arr2 : [boolean, ...string[], number]
arr2 = [true, "hello", "world", 18]
let arr3 : [string, boolean?]
arr3 = ["nihao"]
```

## enum

### 数字枚举

这是 TS 枚举中，最常见的

```ts
enum Direction { Up, Down, Left, Right }
console.log(Direction) // 打印Direction会看到如下内容
/*
{
  0:'Up',
  1:'Down',
  2:'Left',
  3:'Right',
  Up:0,
  Down:1,
  Left:2,
  Right:3
}
*/
// 反向映射
console.log(Direction.Up) // 0
console.log(Direction[0]) // "Up"
// 此行代码报错，枚举中的属性是只读的
Direction.Up = 'shang'
```

### 字符枚举

值是字符串，其他的没啥

```ts
enum Direction {
  Up = "up",
  Down = "down",
  Left = "left",
  Right = "right",
}
let dir: Direction = Direction.Up;
console.log(dir); // 输出: "up"
```

### 常量枚举

官方描述：常量枚举是一种特殊枚举类型，它使用const 关键字定义，在编译时会被内联，避免生成一些额外的代码。

何为编译时内联？  
所谓“内联”其实就是TypeScript 在编译时，会将枚举成员引用替换为它们的实际值，而不是生成额外的枚举对象。这可以**减少**生成的JavaScript 代码量，并**提高**运行时**性能**。

```ts
const enum Directions {
  Up,
  Down,
  Left,
  Right,
}
let x = Directions.Down;
```

转化后的 JS 代码：比数字和字符枚举转化的 JS 代码都要少，这就是常量枚举的好处

```ts
"use strict";
let x = 1 /* Directions.Down */;
```

## 声明函数类型

### 方法一：使用 =>

这里的 => 跟 JS 中的箭头函数，不一样哈~ 这个后面加的是返回类型， JS代码中返回的是一个函数

```ts
let count: (a: number, b: number) => number
count = function (x, y) {
    return x + y
}
```

### 方法二：使用type

```ts
type myAdd = (x: number, y: number) => number;
const add: myAdd = (x, y) => {
  return x + y;
};
```

### 方法三：使用接口

```ts
interface IAdd { 
  (x: number, y: number): number
};
const add: IAdd = (x, y) => {
  return x + y;
};
```

  
## type 联合、交叉类型

### 基本用法

这个东西，就是提供一种**自定义**类型的感觉

```ts
type num = number;

let a: num = 18;
console.log(typeof a); // number
```

### 联合类型

联合类型是一种高级类型，它表示一个值可以是几种不同类型之一。

```ts
type Gender = "男" | "女";
// 这个还能扩展，但是如果使用 enum 枚举的话，扩展性没有那么强
type ExpandGendar = Gender | "未知"

let person: { name: string; age?: number; gender: Gender };

person = { name: "ww", gender: "男" };
```

### 交叉类型

允许将**多个类型合并为一个类型**，合并后的类型将拥有所有被合并类型的成员，交叉类型通常用于**对象类型**。

```ts
type person = {
  name: string;
  age?: number;
  gender: "男" | "女";
};

type student = {
  grade: string;
};

let person: person & student = { name: "ww", gender: "男", grade: "高三" };
```

## TS 对类的扩展

```ts
// JS 代码，描述一个类
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    speak() {
        console.log(`我是${this.name}，年龄是${this.age}`);
    }
}
```

TS 代码

```ts
class Person {
  constructor(public name: string, public age: number) {}

  public speak() {
    console.log(`我是${this.name}，年龄是${this.age}`);
  }
}
```

### 权限修饰符

不难看出，增加了一些**权限修饰符**，并且简写了构造方法

|   |   |   |
|---|---|---|
|修饰符|含义|具体规则|
|public|公开的|可以被：类内部、子类、类外部访问|
|protected|受保护的|可以被：类内部、子类访问|
|private|私有的|可以被：类内部访问|
|readonly|只读属性|属性无法修改|
  

## 抽象类

**关键字：abstract**

**抽象类不能被实例化，可以拥有抽象方法 和 普通方法**  
当然 TS 这个抽象类没有 JAVA 功能那么多，JAVA多了一个 final关键字 和 内部类（通过内部类实现实例化）

```ts
// 抽象类
abstract class Package {
  constructor(public weight: number) {} 
  public abstract calculate(price: number): void; // 抽象方法
}

class standPackage extends Package {
  // 子类需要重写抽象方法
  public calculate(price: number): void {
    console.log("标准的运费是：" + this.weight * price);
  }
}

class quickPackage extends Package {
  constructor(weight: number, private unitPrice: number) {
    super(weight);
  }
  public calculate(price: number): void {
    let total = 0;
    if (this.weight > 10) {
      total = (this.weight - 10) * this.unitPrice + 10 * price;
      console.log("快速的运费是：" + total);
    } else {
      total = this.weight * price;
      console.log("快速的运费是：" + total);
    }
  }
}
```

## 接口

interface 是一种定义结构的方式，主要作用是为：类、对象、函数等规定一种契约，这样可以确保代码的一致性和类型安全，但要注意interface 只能定义格式，不能包含任何实现！

### 定义类的结构

```ts
interface IPerson {
  name: string;
  age?: number;
  speak(n: number): void;
}

class Person implements IPerson {
  constructor(public name: string, public age: number) {}
  speak(n: number): void {
    console.log(n);
  }
}

const p = new Person("张三", 18);
p.speak(3);
```

### 定义对象结构

```ts
interface IStudents {
  name: string;
  age: number;
  gander: "男" | "女";
  garde?: string;
  run(): void;
}

const stu: IStudents = {
  name: "张三",
  age: 18,
  gander: "男",
  run() {
    console.log(`我是${this.name}, 年龄是${this.age}, 性别是${this.gander}`);
  },
};
stu.run();
```

### 定义函数结构

```ts
interface IAdd {
  (x: number, y: number): number;
}
const add: IAdd = (x, y) => {
  return x + y;
};
```

### 接口间的继承

```ts
interface IA {
  name: string;
  speak(): void;
}

interface IB extends IA {
  age: number;
}

// C类实现了 IB 接口，就必须实现 IA 和 IB 接口中所有规定的东西
class C implements IB {
  constructor(public name: string, public age: number) {}
  speak(): void {
    console.log("nihao");
  }
}
```

### 接口间的合并（可重复定义）

可以扩展同一个接口的内容，可以重复定义接口

```ts
interface A {
  name: string;
}

interface A {
  age: string;
}

class C implements A {
  constructor(public name: string, public age: string) {}
}

// 或者 一个接口跟某个类名一样 那么那个接口也可以扩展那个类
interface Person { // 这样Person类上面就加上了 hello 这个方法
  hello() : void
}

class Person {
  constructor(public name: string, public age: string) {}
  speak() {
    console.log(`我的名字是${this.name}, 年龄是${this.age}`)
  }
}

const p = new Person("张三", 18)
p.hello() // 这样就可以调了
```

总结：何时使用接口？

1. 定义对象的格式： 描述数据模型、API 响应格式、配置对象........等等，是开发中用的最多的场景。
2. 类的契约：规定一个类需要实现哪些属性和方法。
3. 扩展已有接口：一般用于扩展第三方库的类型， 这种特性在大型项目中可能会用到



## 泛型

泛型可以让我们**动态**的**传递类型**，当传递参数的时候，才能知道他的类型

### 泛型函数

```ts
function logger<T>(data: T) {
  console.log(data);
}

logger<string>("你好");
logger<number>(5);
```

### 泛型可以有多个

```ts
function logData<T, U>(data1: T, data2: U): T | U {
  console.log(data1,data2)
  return Date.now() % 2 ? data1 : data2
}
logData<number, string>(100, 'hello')
logData<string, boolean>('ok', false)
```

### 泛型接口

```ts
interface PersonInterface<T> {
  name: string,
  age: number,
  extraInfo: T
}
let p1: PersonInterface<string>
let p2: PersonInterface<number>
p1 = { name: '张三', age: 18, extraInfo: '一个好人' }
p2 = { name: '李四', age: 18, extraInfo: 250 }
```

### 泛型约束

```ts
interface LengthInterface {
  length: number
}
// 约束规则是：传入的类型T必须具有 length 属性
function logPerson<T extends LengthInterface>(data: T): void {
  console.log(data.length)
}
logPerson<string>('hello')
// 报错：因为number不具备length属性
// logPerson<number>(100)
```

### 泛型类

```ts
class Person<T> {
  constructor(
    public name: string,
    public age: number,
    public extraInfo: T
  ) { }
  speak() {
    console.log(`我叫${this.name}今年${this.age}岁了`)
    console.log(this.extraInfo)
  }
}
// 测试代码1
const p1 = new Person<number>("tom", 30, 250);
// 测试代码2
type JobInfo = {
  title: string;
  company: string;
}
const p2 = new Person<JobInfo>("tom", 30, { title: '研发总监', company: '发发发
  科技公司' });
```

## 类型文件说明

如果想在 TS 文件中，导入 JS 文件，那么就需要在目录下面写入一个跟 JS 文件名一模一样的 .d.ts 文件

类型声明文件是 TypeScript 中的一种特殊文件，通常以 `.d.ts` 作为扩展名。它的主要作用是为现有的JavaScript 代码提供类型信息，使得 TypeScript 能够在使用这些 JavaScript 库或模块时进行类型检查和提示。

```ts
export function add(a, b) {
  return a + b;
}
export function mul(a, b) {
  return a * b;
}
```

```ts
declare function add(a: number, b: number): number;
declare function mul(a: number, b: number): number;
export { add, mul };
```

```ts
import { add, mul } from "./demo.js";
const x = add(2, 3); // x 类型为 number
const y = mul(4, 5); // y 类型为 number
console.log(x,y)
```

  

## 装饰器

装饰器本质是⼀种特殊的**函数**，它可以对：类、属性、⽅法、参数进⾏扩展，同时能让代码更 简洁。

### 类装饰器

#### 基本使用

```ts
function demo(target: Function) {
  console.log(target) // class A{} 就是A类
}
class A {
}
```

```ts
// 定义⼀个装饰器，实现执⾏结果。Person 实例调⽤toString 时返回JSON.stringify 的应⽤举例
function demo(target: Function) {
  target.prototype.toString = function () {
    return JSON.stringify(this);
  };
}

@demo
class Person {
  constructor(public name: string, public age: number) {}
}

const p = new Person("张三", 18);
console.log(p.toString());
```

#### 怎么在 TS 中限制它为构造函数呢？

```ts
/**
  ○ new     表示：该类型是可以⽤new操作符调⽤。
  ○ ...args 表示：构造器可以接受【任意数量】的参数。
  ○ any[]   表示：构造器可以接受【任意类型】的参数。
  ○ {}      表示：返回类型是对象(⾮null、⾮undefined的对象)
 */
type Constructor = new (...args: any[]) => {};
```

```ts
// 定义⼀个构造类型，且包含⼀个静态属性wife
type Constructor = {
  new(...args: any[]): {}; // 构造签名
  wife: string; // wife属性
};
function test(fn:Constructor){}
class Person {
  static wife = 'asd'
}
test(Person)
```

#### 关于返回值

类装饰器**有**返回值：若类装饰器**返回⼀个新的类**，那这个新类将**替换**掉被装饰的类。

类装饰器**⽆**返回值：若类装饰器⽆返回值或返回 undefined ，那被装饰的类不会被替换。

一个例子：

```ts
// 需求：设计⼀个LogTime 装饰器，可以给实例添加⼀个属性
// ⽤于记录实例对象的创建时间，再添加⼀个⽅法⽤于读取创建时间。
type Constructor = new (...args: any[]) => {};
interface Person { // 防止Person类实例在调用的时候，报错（类型“Person”上不存在属性“getTime”）
  getTime(): Date; 
}

function LogTime<T extends Constructor>(target: T) {
  return class extends target {
    createdTime: string;
    constructor(...args: any[]) {
      super(...args);
      this.createdTime = new Date().toLocaleString();
    }
    getTime() {
      return `这个对象的创建时间是${this.createdTime}`;
    }
  };
}

@LogTime
class Person {
  constructor(public name: string, public age: number) {}
  speak() {
    console.log(`我是${this.name}, 年龄${this.age}`);
  }
}

const p = new Person("张三", 18);
console.log(p); // 这个地方输出的是被替换的那个类
p.speak();
console.log(p.getTime());
```

#### 类装饰器工厂

其实就是装饰器中在**返回一个函数**，这个函数才是真正的装饰器

```ts
// 需求：定义⼀个 LogInfo 类装饰器⼯⼚，实现 Person 实例可以调⽤到 introduce ⽅法，
// 且 introduce 中输出内容的次数， LogInfo 接收的参数决定。
type Constructor = new (...args: any[]) => {};
interface Person {
  introduce(): void;
}

function LogInfo(n: number) {
  // 真正的装饰器
  return function <T extends Constructor>(target: T) {
    target.prototype.introduce = function () {
      for (let i = 0; i < n; i++) {
        console.log(`我是${this.name}, 年龄是${this.age}`);
      }
    };
  };
}

@LogInfo(5)
class Person {
  constructor(public name: string, public age: number) {}
}
const p = new Person("张三", 18);
p.introduce();
```

#### 装饰器组合

先【从上到下】执行所有的装饰器工厂，得到装饰器之后，在【从下往上】执行所有的装饰器

```ts
function test1(target: Function) {
  console.log("test1");
}
function test2() {
  console.log("test2工厂");
  return function (target: Function) {
    console.log("test2");
  };
}
function test3(target: Function) {
  console.log("test3");
}
function test4() {
  console.log("test4工厂");
  return function (target: Function) {
    console.log("test4");
  };
}

@test1
@test2()
@test3
@test4()
class person {}

// 输出
// test2工厂
// test4工厂
// test4
// test3
// test2
// test1
```