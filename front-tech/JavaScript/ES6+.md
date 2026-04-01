# 类
>  ES6 更加支持面向对象编程

传统的构造函数的问题：
1. 属性和原型方法定义分离，降低了可读性
2. 原型成员可以被枚举
3. 默认情况下，构造函数仍然可以被当作普通函数使用

## 类的特点
1. 类声明*不会被提升*，与 let 和 const 一样，存在暂时性死区
2. 类中的所有代码均在*严格模式*下执行，全局调的函数，this 指向 undefined
3. 类的<font color="#ff0000">所有方法</font>都是<font color="#ff0000">不可枚举</font>`enumerable: false`，*方法*都加到*类的原型*上了（因为方法不可枚举，所以直接 ` Person.prototype === {} ` 在控制台是没有值输出的）
4. 类的所有方法都无法被当作构造函数使用
5. 类的构造器必须使用 *new* 来调用

第 4 点和第 5 点，是因为 ES5 之前的函数都有两种作用，一个是当作*构造器*（一般大写首字母），另一个是当作*普通函数*，所以为了避免歧义，
ES6 就明确了所有的方法不能作为构造函数，同时 ES6 中也提供了箭头函数它的作用只能当作普通函数

ES6 写法：
```js
class Person {
  // 挂在 Person 的原型上
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // 挂在 Person 的原型上
  sayHello() {
    console.log("hello");
  }
}

const p = new Person("张三", 18);
console.log(p);
```

ES5 写法：
```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayHello() {
  console.log("hello")
}

const p = new Person("张三",18)
```

## `get` 和 `set`
就跟 java 中的 get 和 set 方法差不多，JS 中应该是用 `Object.defineProperty()` 来实现的，ES 6 中提供了一种简单的写法

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this._age = age;
  }

  get age() {
    return this._age;
  }

  set age(val) {
    this._age = val;
  }

  sayHello() {
    console.log("hello");
  }
}
const p = new Person("张三", 18);
console.log(p.age) 
// 就跟定义属性描述符一样的，
// 访问age就会调用get age这个方法
```

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this._age = age;
    // 对面的get和set就相当于下面的代码
    Object.defineProperty(this, "age", {
      get() {
        return this._age;
      },
      set(val) {
        this._age = val;
      },
    });
  }

  sayHello() {
    console.log("hello");
  }
}

const p = new Person("lisi", 18);
console.log(p.age);
```

## 私有属性
- 不暴露
- 不可反射（reflect）
- 不可动态访问 (defineProperty)
- 不可被代理（Proxy）拦截

使用 `#name` "#" 来标识
```js
class Person {
  #name = "张三";   // 私有属性
  #age = 18;

  get name() {
    return this.#name;
  }
  
  set name(val) {
  	this.#name = val;
  }
}

const p = new Person();

console.log(p.#name); //  报错
p.name = "tom";
console.log(p.name); // tom
```

ES 5 通过*闭包*实现：
```js
function Person() {
	let name = "张三";

	this.getName = function() {
		return name;
	}
}
const p = new Person(); // p 的实例上只挂载了 getName 函数，访问不到 name
console.log(p.getName()); 
```

---
闭包的作用之一： 实现*私有属性*

进阶：
私有属性挂在哪里？
—— 内部槽（internal slot）/ 私有字段存储空间

JS 每一个对象上都有一个隐藏结构： `[[PrivateFieldValues]]`
私有属性可以理解为：
```js
p = {
  // 普通属性
  // name: "xxx"

  // 内部私有槽
  [[PrivateFieldValues]]: {
    #name: "张三"
  }
}
```

## 静态方法

使用关键字：`static`

那么跟 Java 一样，静态方法都是加在**类上的**，对象是不可以访问的

```js
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  static a = 1; // 静态属性 
  static test() {  // 静态方法
    console.log("test方法");
  }

  sayHello() {
    console.log("hello");
  }
}
```

## 字段初始化器
做了一些简化，使用字段初始化器，就相当于在构造器中赋了值，很适合一些**属性有初始值**的情况
比如，
```js
class Person {
  // 这里是挂在实例对象上的 p
  name = "张三";
  age = 18;
  
  // 上面的代码相当于下面的代码
  // constructor(name, age) {
  //   this.name = name;
  //   this.age = age;
  // }
}

const p = new Person();
console.log(p); // Perosn {name: "张三", age: 18}
```

【注意】：
1. 如果 字段初始化器 与 constructor 同时存在的话，会去找*构造器*的值
2. 没有使用static的字段初始化器，添加的成员位于**对象上**
3. 箭头函数在字段初始化器位置上，指向当前对象

第 3 点是什么意思呢？
举个例子，
```js
class Person {
  name = "张三";
  age = 18;

  // 这种是字段初始化器的写法，这里的this绑定的是当前对象，因为这种写法，其实是写在构造器中的
  print = () => {
    console.log(this.name + this.age);
  };
  
  constructor(name, age) {
    this.name = name;
    this.age = age;
    // 这里的this指向类的实例
    this.print = () => {
      console.log(this.name + this.age);
    }
  }

  // 普通成员方法的写法 -- 挂载原型上
  print() {
    console.log(this.name);
  }
}

const p = new Person();
const t = p.print; 
t();
// 如果是普通的写法，这个地方会报错的，全局调用，那么this的指向undefined
// 要手动绑定this的指向  const t = p.print.bind(p); 在调用 t() 就不会报错了
```

但是，这种写法有点问题，

使用字段初始化器的属性或方法，都是加在对象上的，普通方法是加在类的原型上的
所以，如果用这种字段初始化器的方式，*每一个对象都会创建一个新的 print 函数*，会造成内存的一定浪费。最好不使用字段初始器方法

## 继承

### ES5 的写法
```js
function Animal(type, name, age) {
  this.type = type;
  this.name = name;
  this.age = age;
}

Animal.prototype.print = function () {
  console.log("【类型】：" + this.type);
  console.log("【名称】：" + this.name);
  console.log("【年龄】：" + this.age);
};

function Dog(type, name, age, loves) {
  Animal.call(this, type, name, age); // 调用父类的构造函数
  this.loves = loves;
}

// 将 Dog的原型的隐式原型 指向 Animal的原型，之前是指向Object的原型
Object.setPrototypeOf(Dog.prototype, Animal.prototype);

// 重写父类方法
Dog.prototype.print = function () {
  Animal.prototype.print();
  console.log("【爱好】：" + this.loves);
};

const dog = new Dog("犬类", "小白", 5, "变棉花");
console.log(dog);
dog.print();

console.log(Dog.prototype);
```

重点就是两句话，
1. 调用父类构造函数
```js
function Dog(type, name, age, loves) {
  Animal.call(this, type, name, age); // 调用父类的构造函数
  this.loves = loves;
}
```

2. 将 子类原型的隐式原型 指向 父类的原型
```js
// 将 Dog的原型的隐式原型 指向 Animal的原型，之前是指向Object的原型
Object.setPrototypeOf(Dog.prototype, Animal.prototype);
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751108688953-0e68db39-f56f-447e-a373-bfa4cf2a1d58.png)

### ES6 的写法
```js
class Animal {
  constructor(type, name, age) {
    this.type = type;
    this.name = name;
    this.age = age;
  }

  print() {
    console.log("【类型】：" + this.type);
    console.log("【名称】：" + this.name);
    console.log("【年龄】：" + this.age);
  }
}

class Dog extends Animal {
  constructor(type, name, age, loves) {
    super(type, name, age);
    this.loves = loves;
  }

  // 重写父类的方法
  print() {
    super.print();
    console.log("【爱好】：" + this.loves);
  }
}

const dog = new Dog("犬类", "小白", 5, "变棉花");
console.log(dog);
dog.print();
```

ES6 中新增了`extends、super`关键字，来解决类之间的**继承**关系。ES5 中的手动处理原型链的问题，在 ES6 中自动给我们解决好了。

# 函数

## 参数默认值
这个发生在传递参数的过程中出现，如果没有传实参，那么形参就用默认值，
就是要注意，默认参数要放到参数列表的最后面

```js
function sum(a, b = 10) {
  return a + b;
}

sum(10);
```

## 剩余参数
用于一个函数中，不确定有多少个形参列表的情况

```js
function sum(...args) {
  return args.reduce((a, b) => a + b, 0);
}

console.log(sum(10, 20));
console.log(sum(10, 20, 30));
console.log(sum(10, 20, 30, 40));
```

## 展开运算符

这个形式上跟剩余参数，差不多，也是三个点，这个东西就非常灵活了

可以用来展开*可迭代对象*（存在知名符号` [Symbol.iterator]`）

这里我就不多写了，展开数组求最大值、最小值什么的，进行浅拷贝，合并数组、对象什么的，只有你想不到没有它做不到

## 判断调用函数是否是用 new 关键字创建的
```js
function Person(name, age) {
  // 之前判断 this
  if (this instanceof Person) {
    throw new Error("没有用new关键字创建函数");
  }
  // ES6 之后出了一个 new.target
  // console.log(new.target); // 如果用new创建的，那么就会指向Person对象，如果不是为undefined
  if (new.target === undefined) {
    throw new Error("没有用new关键字创建函数");
  }
  this.name = name;
  this.age = age;
}

const p1 = new Person("张三", 18);
console.log(p1);

const p2 = Person("李四", 20);
console.log(p2);
```

# Promise
>  专门用来处理异步场景（网络请求、监听事件、定时器等），之前都是用回调函数来处理异步的
>  有了 Promise 就有了统一处理异步的方法

## Promise A+规范
这个是在 ES6 还没出 Promise 之前，前端社区出的一个规范

这个规范认为呢，每一个异步的场景，都应该是一个 Promise 对象。

每一个 Promise 对象都应该有两个阶段，三种状态。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750927034868-4c83b66f-8d02-4b93-9809-abd522adbf8d.png)

【注意】：
- 一旦进入已决阶段，那么这个 promise 的状态就不能修改
- 不能说从 fullfilled 的阶段又回到 pending 阶段，这个是不可能的

从未决到已决阶段：通过两个方法【resolve(data)从 pending 到 fulfilled ，同理 reject(resaon)从 pending 到 rejected】

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750927398972-2e5bad86-2cc9-4555-92e4-324b7974eca5.png)

对【已完成】状态后续的处理，称之为 onfulfilled(data)，对【拒绝】状态后续的处理，称之为 onrejected(err)

【注意】：
onfulfilled 和 onrejected 都是进入微队列的

## ES6 实现
```js
// ES6 提供的 Promise对象 就是实现了上面所说的 Promise A+ 规范中说的几点
 const pro = new Promise((resolve, reject) => {
    console.log("立即执行");
    if (Math.random() < 0.5) {
      resolve("成功！");
    } else {
      reject("失败");
    }
  });
pro.then((data) => console.log(data)).catch((err) => console.log(err));
```

【注意】：
1. `new Promise((resolve, reject) => {})`这个会立即执行，不会加入到微队列，加入微队列的是后续的处理函数（then、catch）
2. 如果整个 Promise 没有调用 resolve 或 reject，promise 的状态是 pending
3. 调用 resolve ，那么 promise 状态是 fulfilled
4. 调用 reject，那么 promise 状态时 rejected

## Promise 链式调用

那么这个才是真繁琐，有下面几条规则

1. then 方法必定会返回一个新的 Promise，可以认为是一个新任务
2. 新任务的状态取决于后续处理：

3. 若**没有**相关后续处理，那么新任务的状态和前任务一直，数据为前任务的数据
4. 若**有**后续处理但**还未执行**，新任务挂起
5. 若**有**后续处理并且**执行了**，那么根据后续处理的情况确定新任务的状态

6. 后续处理执行**没有报错**，那么新任务状态为**完成**，数据为后续处理的返回值
7. 后续处理执行**有报错**，那么状态为**失败**，数据为异常对象
8. 后续处理要是返回了一个**新的任务**，那么新任务的状态和数据与返回的任务对象一致

### 情况一：没有后续处理
```js
const pro1 = new Promise((resolve, reject) => {
  console.log("学习");
  resolve(12);
});

console.log(pro1); // Promise { 12 } fulfilled
console.log("-----------------------");

const pro2 = pro1.catch(() => {
  console.log("考试");
});     // 没有后续处理，前任务是resolve，后任务是catch，状态与前任务保持一致

setTimeout(() => {
  console.log(pro2); // Promise { 12 }
}, 1000);
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750937122270-5bc1ed61-4d63-473b-a600-692abdace14f.png)

### 情况二：有后续处理，但是前任务没有执行完
```js
const pro1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log("学习");
    resolve(12);
  }, 2000);
});

console.log("pro1", pro1);
console.log("-----------------------");

const pro2 = pro1.then(() => {
  console.log("考试");
});  // 有后续处理，但是前任务没有执行完成，pending

setTimeout(() => {
  console.log("pro2", pro2);
}, 1000);
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750937519115-cc26ca10-85b4-43a9-a9e4-9b6998e87fb7.png)

### 情况三：有相关处理，执行没有报错
```js
const pro1 = new Promise((resolve, reject) => {
  console.log("学习");
  resolve(12);
});

console.log(pro1); // Promise { 12 } -> fulfilled
console.log("-----------------------");

const pro2 = pro1.then(() => {
  console.log("考试");
  return 456;
});   // 有相关处理，执行过程中没有报错，状态为fulfilled，数据是返回的数据

setTimeout(() => {
  console.log(pro2); // Promise { 456 } -> fulfilled
}, 1000);
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750937565220-1cd72777-526a-47a5-9339-119d827f3aeb.png)

### 情况四：有后续处理，但是执行过程中报错了
```js
const pro1 = new Promise((resolve, reject) => {
  console.log("学习");
  resolve(12);
});

console.log("pro1", pro1); // Promise { 12 }  
console.log("-----------------------");

const pro2 = pro1.then(() => {
  console.log("考试");
  throw new Error("考试睡着了~"); 
}); // 有后续处理，但是处理过程中报错了，rejected

setTimeout(() => {
  console.log("pro2", pro2);
}, 1000);
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750937748430-33676166-e56e-4e0f-abec-c6d30d45501f.png)

### 情况五：有后续处理，返回一个新任务
```js
const pro1 = new Promise((resolve, reject) => {
  console.log("学习");
  resolve(12);
});

console.log("pro1", pro1); // Promise { 12 }
console.log("-----------------------");

const pro2 = pro1.then(() => {
  return new Promise((resolve, reject) => {}); // 返回一个新任务，pro2的状态跟随新任务状态
});

setTimeout(() => {
  console.log("pro2", pro2); // Promise <pending>
}, 1000);
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1750937865318-56b10b28-4c45-4214-a9c4-b0f6314b3c38.png)

## Promise 静态方法

用于处理**多个** Promise 对象，需要**汇总**的情况

|   |   |
|---|---|
|**方法名**|**含义**|
|Promise.resolve(data)|直接返回一个完成状态的任务|
|Promise.reject(reason)|直接返回一个拒绝状态的任务|
|Promise.all(任务数组)【要求：一个都不能少】|返回一个任务  <br>任务数组全部成功则成功  <br>任何一个失败则失败|
|Promise.any(任务数组)|返回一个任务  <br>任务数组任一成功则成功  <br>任务全部失败则失败|
|Promise.allSettled(任务数组)<br><br>【这个方法只有两种状态：pending 和 fulfilled】|返回一个任务  <br>任务数组**全部已决**则成功  <br>该任务不会失败|
|Promise.race(任务数组)|返回一个任务  <br>任务数组任一已决则已决，状态和其一致|

【注意】：使用技巧
1. 如果需要结果一个都不能少，全部都要成功，那么 all
2. 如果需要得到第一个成功的信息，那么 any
3. 如果需要知道每个任务的状态，那么 allSettled, Promise.allSettled 的返回值是一个 对象数组，每个对象有一个 status 来描述那个任务的状态
4. 如果需要第一个有结果的数据（不管成功还是失败），那么 race

## async 和 await
其实就是 Promise then 和 catch 的语法糖，Promise 解决了异步场景中地狱回调的问题，但是没有解决回调的问题。那么 async 和 await 就可以解决回调的问题。

【注意】：在 ESM 环境下，不能在顶层直接使用 await，必须放在 async 函数下（是为了更好的静态模块分析）；在 CJM 环境下，可以直接使用 await .

**async**

async 一定是标记在函数前面的，一旦标记了 async 关键字，那么该函数**返回 Promise 对象**。

比如，
```js
// 情况一：直接return一个值
async function test() {
  return 1; // 这个地方就相当于 return Promise.resolve(1);
}

console.log(test()); // 这个函数返回的是一个 Promise 对象，数据是return后的数据
// Promise { 1 } -> fulfilled

// 情况二：返回一个新的Promise
async function test() {
  return new Promise((resolve) => {
    resolve(123);
  }); // 状态吸收
}

setTimeout(() => {
  const pro = test();
  console.log(pro); // Promise<pending>
  pro.then(() => {
    console.log("Promise状态：", pro); // Promise { 123 }
  });
}, 1000);


// 情况三：async 里面报错
async function test() {
  const data = null;
  data.toString();
  return 123;
}

setTimeout(() => {
  console.log(test()); // <rejected> xxx null toString...
}, 1000);
```

**await**

await 的作用就是：等待一个 promise，如果这个*promise 状态是已完成*，那么将后续代码加入到微队列，如果没有后续代码，那么将 async 返回的 promise 的状态从 pending 转到 fulfilled 的这个过程加入到微队列。

【注意】：await 后面跟的可以不是一个 Promise 对象，但 await 内部会将其转化成一个 Promise 对象

比如：

```js
await 1 <==> await Promise.resolve(1);
```

用法：
```js
function delay(duration) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, duration);
  });
}

await delay(1000);
console.log("1秒之后输出这句话");

// 上面两句 就相当于下面的写法
delay(1000).then(() => console.log("1秒之后输出这句话"))
```

await 表示等待某个 Promise 完成，
```js
async function method(){
  const n = await 1;
  console.log(n); // 1
} // const n 和 console.log(n) 实际上是在then方法里面的，会加入到微队列

// 上面的函数等同于
function method(){
  return new Promise((resolve, reject)=>{
    Promise.resolve(1).then(n=>{
      console.log(n);
      resolve(1)
    })
  })
}
```

如果需要针对失败的任务进行处理，可以使用`try-catch`语法
```js
async function method(){
  try{
    const n = await Promise.reject(123); // 这句代码将抛出异常
    console.log('成功', n)
  }
  catch(err){
    console.log('失败', err)
  }
}

method(); // 输出： 失败 123
```

### 练习题
```js
async function asy1() {
  console.log(1);
  await asy2();
  console.log(2);
}

const asy2 = async () => {
  await setTimeout(() => {
    Promise.resolve().then(() => {
      console.log(3);
    });
    console.log(4);
  }, 0);
  // 这里要认识两个函数有什么作用：
  // 1. setTimeout 这个函数呢 ---> 通知计时线程0秒后，将回调函数加入到延时队列中
  // 2，await 等待一个promise，如果这个promise是已完成的状态，那么将后续代码加入到微队列，如果后续没有代码，那么将async返回的promise从pending转到fulfilled状态
};

const asy3 = async () => {
  Promise.resolve().then(() => {
    console.log(6);
  });
};

asy1();
console.log(7);
asy3().then(() => {
  console.log("object");
});
// 输出结果： 1 7 6 2 4 3
```

## 状态吸收

一个 Promise 函数(p1)中返回了另一个 Promise(p2)，那么这个 p1 promise 需要吸收 p2 的状态，具体过程分为两步，都是放在**微队列**中执行的。一步是“准备”，另一步是“吸收”要等微队列中“吸收”这个任务执行完之后，p1 的状态才会变成 fulfilled，否则为 pending。

那么这里有几个例子，

```js
// 例子一：
const p1 = Promise.resolve();
const p2 = new Promise((resolve) => {
  resolve(p1); // 在一个promise中调用另一个promise
});

console.log(p1); // promise{undefined} -> fulfilled
console.log(p2); // promise{pending}

// 例子二：
async function async1() {
  console.log(1);
  await async2(); // 当微队列中完成“吸收”这个任务后，await的这个promise就到了fulfilled状态
  console.log("AAA");
}

async function async2() {
  return Promise.resolve(2); 
  // 这里被async关键字标记了，会返回一个新的promise，
  // 这个promise需要吸收Promise.resolve(2)它的状态
}

async1();

Promise.resolve()
  .then(() => {
    console.log(3);
  })
  .then(() => {
    console.log(4);
  })
  .then(() => {
    console.log(5);
  });
// 具体过程是这样的，
// 微队列：准备、输出3、吸收、输出4、输出AAA、输出5
// 输出：1 3 4 AAA 5

// 例子三：
Promise.resolve()
  .then(() => {
    console.log(0);
    return Promise.resolve(4);
  })
  .then((res) => {
    console.log(res);
  });

Promise.resolve()
  .then(() => {
    console.log(1);
  })
  .then(() => {
    console.log(2);
  })
  .then(() => {
    console.log(3);
  })
  .then(() => {
    console.log(5);
  });
// 输出 0 1 2 3 4 5
```

## 【手写】xxx

# 迭代器

其最核心的东西，就是可以动态的取数据，不用管数据有多少，使用迭代器的人也不需要知道数据是什么

## 形式

迭代器返回一个对象，对象中需要包含一个 *next 函数*，函数返回一个对象，对象中包含 value 和 done 两个属性
- value：表示下一个值
- done：表示后面是否还有值

举个例子：
```js
const arr = [1, 2, 3, 4, 5];

const obj = {
  i: 0,
  next() {
    return {
      value: arr[this.i++],
      done: this.i > arr.length - 1,
    };
  },
};

console.log(obj.next()); // 1
console.log(obj.next()); // 2
```

迭代器**获得数据**是随时可中断的，

如果有大量数据需要读取出来，遍历的话，需要先将数据放到某个容器（数组）中，消耗内存不说，而且遇到无限多的数据就无计可施了

迭代器可以获取**无限大量**数据，
```js
function createFibonacciIterator() {
  let pre1 = 1,
    pre2 = 1,
    n = 1;
  return {
    next() {
      let value;
      if (n <= 2) {
        value = 1;
      } else {
        value = pre1 + pre2;
        pre1 = pre2;
        pre2 = value;
      }
      n++;
      return { value, done: false };
    },
  };
}

const fib = createFibonacciIterator();
console.log(fib.next());
console.log(fib.next());
```

## 可迭代协议

ES6规定，如果一个对象具有知名符号属性`Symbol.iterator`，并且属性值是一个**迭代器创建函数(iterator creater)**，则该对象是可迭代的（iterable）

比如说，
```js
const arr = [1, 2, 3, 4, 6];
const iterator = arr[Symbol.iterator]();  // Symbol.iterator 返回的是一个迭代器函数
let res = iterator.next(); // res是迭代器返回的对象{value: 1, done: false}
while (!res.done) {
  console.log(res.value);
  res = iterator.next();
}
```

利用知名符号，可以自定义可迭代变量
```js
const obj = {
  a: 1,
  b: 2,
  [Symbol.iterator]() {
    const keys = Object.keys(obj);
    const values = Object.values(obj);
    let i = 0;
    return {
      next() {
        return {
          value: { keyName: keys[i], keyValue: values[i] },
          done: i++ >= keys.length,
        };
      },
    };
  },
};

// const iterator = obj[Symbol.iterator]();
// let res = iterator.next();
// while (!res.done) {
//   console.log(res.value);
//   res = iterator.next();
// }

// for...of 其实就是上面这一段代码的语法糖，可以用来遍历可迭代对象
// item 是迭代对象返回的value值
for (const item of obj) {
  console.log(item);
}
```

# 生成器

那么生成器，它诞生的目的呢，就是为了简化迭代器的每次 next 函数中返回的数据

**生成器是什么呢？**

生成器只能由 Generator 函数创建，但是不能直接通过 new 关键字创建，生成器函数会返回一个生成器，生成器中包含了 next 函数和知名符号 Symbol.iterator，所以说它返回了一个迭代器也是可以的

**怎么创建生成器函数呢？**

```js
function *test() {}
// 或
function* test() {}
```

## 简单认识生成器
```js
function* test() {
  console.log("第一次运行");
  yield 1; // yield 后面的就类似于 迭代器 next 函数中 value 的值
           // 只要生成器函数还有语句，那么done为 false
  console.log("第二次运行");
  return 10; // 给最后一个value赋值，如果没有显示return那么就是undefined
}

// 注意：这里并不会执行test函数中的语句
const generator = test(); // 调用test()生成器函数 返回一个生成器

// 每次迭代 -> 运行到下一个 yield 关键字停止
console.log(generator.next()); // {value: 1, done: false}
console.log(generator.next()); // {value: 10, done: true}
```

【注意】：
1. 调 `next()`可以传递参数，传递的参数会交给 yield 表达式的返回值
2. 生成器函数中使用`return 10`会给最后一个 value 赋值
3. 在生成器函数内部，可以调用其他生成器函数，但是要注意加上*号

## 举个例子

利用生成器来迭代数组
```js
const arr = [1, 2, 3, 4];
function* test(arr) {
  for (const item of arr) {
    yield item;
  }
}

const iterator = test(arr);
for (const item of iterator) {
  console.log(item);
}
```

next() 传递参数
```js
function* test() {
  console.log("第一次运行");
  const a = yield 1;
  console.log("----------");
  console.log("第二次运行");
  console.log(a);
}

const it = test();
console.log(it.next()); // 这行代码会停在 yield 1 ，但是此时 a 还没有赋值
console.log(it.next(112)); // 第二次运行的时候，此时 a 才会被赋值，值为 112
```

在生成器中调用其他的生成器
```js
function* t1() {
  yield "a";
  yield "b";
}

function* test() {
  yield* t1(); // 类似于 yield "a"; yield "b";
  yield 1;
  yield 2;
}

const it = test();
console.log(it.next()); // { value: 'a', done: false }
console.log(it.next()); // { value: 'b', done: false }
console.log(it.next()); // { value: '1', done: false }
console.log(it.next()); // { value: '2', done: false }
console.log(it.next()); // { value: undefined, done: false }
```

## 【应用】处理异步场景
```js
function* task() {
  const a = yield 1;
  console.log(a); // 同步代码，这里经过run处理后，应该输出 1

  const res = yield fetch("https://uapis.cn/api/say").then((resp) => resp.text());
  console.log(res); // 处理异步代码，这里经过run处理后，应该是服务器返回的结果
}

// 接收一个生成器函数，到达类似于 async 和 await
function run(generatorFunc) {
  const generator = generatorFunc();
  // 手动执行生成器函数中的语句，直到第一个yield关键字
  let res = generator.next();
  handleResult();
  function handleResult() {
    if (res.done) {
      return;
    }

    // 处理res的value的类型 -> 1.promise类型 2.其他类型
    if (typeof res.value.then === "function") {
      // primise类型
      res.value.then((data) => {
        res = generator.next(data);
        handleResult();
      });
    } else {
      // 其他类型
      res = generator.next(res.value);
      handleResult();
    }
  }
}

run(task);
```

# Set 集合

Set 集合：用于存放不重复的数据，是一个可迭代对象，没有下标

**创建一个 Set 集合**
```js
// 无参构造
new Set();
// 有参构造 -> 传一个可迭代对象
可以通过一维数组来初始化set集合
new Set(iterator); -> new Set([1,2,4,5])
```

**一些 API**
```js
add(iterator)
delete(item) -> boolean: true -> 表示删除成功 false -> 删除失败
has(item) -> boolean: true -> 集合中存在item
size
for of
forEach((item, index(item), set) => {}) 
// 注意：set集合中的forEach与数组中的forEach有些许差入，
// 集合中的callback的第二个参数是item本身，数组中是index下标
```

**集合与数组之间的转化**
使用展开运算符即可，展开运算符可以展开**可迭代对象**

```js
const s = new Set([2,4,5]);
console.log([...s])
```

## 【应用】并集、交集、差集

虽然说 Set 集合中有对应的 API 可以实现功能
```js
const arr1 = [11, 22, 33, 44, 33, 66, 11];
const arr2 = [98, 77, 23, 22, 25, 57, 12];

// 并集
console.log([...new Set(arr1.concat(arr2))]);

// 交集
console.log([...new Set(arr1)].filter((item) => arr2.includes(item)));

// 差集
console.log(arr1.filter((item) => !new Set(arr2).has(item)));
console.log(arr2.filter((item) => !new Set(arr1).has(item)));
```

## 【扩展】手写 Set
```js
class MySet {
  _datas;
  constructor(iterator = []) {
    // 判断传进来的参数是不是 可迭代对象
    if (!iterator[Symbol.iterator]) {
      throw new Error(`${iterator}不是可迭代对象`);
    }

    this._datas = [];
    for (const item of iterator) {
      this.add(item);
    }
  }

  add(value) {
    if (!this.has(value)) {
      this._datas.push(value);
    }
  }

  has(value) {
    for (const item of this._datas) {
      if (this.isEquals(item, value)) {
        return true;
      }
    }
    return false;
  }

  delete(value) {
    for (let i = 0; i < this._datas.length; i++) {
      const element = this._datas[i];
      if (this.isEquals(element, value)) {
        this._datas.splice(i, 1);
        return true;
      }
    }
    return false;
  }

  // 只读属性 
  get size() {
    return this._datas.length;
  }

  clear() {
    this._datas.length = 0;
  }
  isEquals(val1, val2) {
    if (val1 === 0 && val2 === 0) {
      return true;
    }
    return Object.is(val1, val2);
  }

  // set 创建出来的函数是可迭代对象
  *[Symbol.iterator]() {
    for (const item of this._datas) {
      yield item;
    }
  }
}

const set = new MySet();
set.add(1);
set.add(12);
set.add(12);
set.add("1");

for (const element of set) {
  console.log(element);
}

console.log(set);
```

# Map

存储键值对的数据结构，有 key 和 value 组成，键是不能重复的

与对象不同的是：

1. 对象的键 只能是**字符串或 Symbol**（其他类型会转成字符串），而 Map 的键可以**随意**
2. 对象是**不可迭代**的，而 Map 是*可迭代的*
3. 对象不易获取长度（Object.keys(obj).length），而 Map 通过 size 属性就可以获得

一般，结构固定的数据结构使用 **对象**，需要频繁的增删改查使用 **Map**

**创建 Map**
```js
new Map();
new Map(itertable); // 注意，第一层是可迭代对象，第二层也必须是迭代对象，
// 会将第二层的第一位作为 key ，第二层的第二位作为 value

例如：可以用二维数组来初始化Map
new Map([["a", 2], ["b", 3], ["c", 4]]) => "a"是key，2是value（以此类推）
```

注意： Map 对于 key 的判断，采用的是 SameValueZero 的算法，也就是 0 与 -0 不相等，但是 NaN 等于 NaN。其他的值都遵循 === 的判断结果.

**Map 的 API**

- `set(key, value)` -> 如果 key 不存在，那么新增一个，如果存在，修改值
- `get(key)` -> 根据 key 获得对应的 value
- `has(key) `-> 判断 key 在不在 Map
- `delete(key)` -> 删除 key
- `size` -> 长度
- `clear()` -> 清空 Map
- `forEach((value, key, map) => {})` -> value：值，key：键，map：Map 本身
--- 
返回的是可迭代对象，可用 for...of 来遍历
- `keys()` -> 获取所有的 key 值
- `values()` -> 获取所有的 value 值
- `entries()` -> 获取 key 、value 值

```js
const map = new Map();
map.set("0", "foo");
map.set(1, "bar");

const iterator = map.keys();
console.log(iterator.next().value); // "0"
```

## 【扩展】手写 Map
```js
class MyMap {
  _datas;
  constructor(itertable = []) {
    if (!itertable[Symbol.iterator]) {
      throw new Error(`${itertable}不是可迭代对象`);
    }
    this._datas = [];
    for (const item of itertable) {
      if (!item[Symbol.iterator]) {
        throw new Error(`${item}不是可迭代对象`);
      }

      /* iter 是迭代器对象
       *    {
	            next(){
		            return {
			            value: ["a", 1],
			            done: xxx
		            }
	            }
            }
       */
      const iter = item[Symbol.iterator]();
      const key = iter.next().value[0];
      const value = iter.next().value[1];
      this.set(key, value);
    }
  }

  set(key, value) {
    // 如果key在Map中不存在，那么直接增加，如果存在，修改对应的value
    if (this.has(key)) {
      const obj = this._getObj(key);
      obj.value = value;
    } else {
      this._datas.push({
        key,
        value,
      });
    }
  }

  get(key) {
    return this._getObj(key).value;
  }

  // 查找 key 返回对应的 item
  _getObj(key) {
    for (const item of this._datas) {
      if (this.isEquals(key, item.key)) {
        return item;
      }
    }
  }

  *[Symbol.iterator]() {
    for (const item of this._datas) {
      yield [item.key, item.value];
    }
  }

  has(key) {
    return this._getObj(key) !== undefined;
  }

  isEquals(val1, val2) {
    if (val1 === 0 && val2 === 0) {
      return true;
    }
    return Object.is(val1, val2);
  }
}

const map = new MyMap();
map.set("a", 1);
map.set("a", 123);
map.set("b", 123);
map.set("v", 123);
for (const [key, value] of map) {
  console.log("key " + key, "value " + value);
}

console.log(map);
```

# WeakSet 和 WeakMap

## WeakSet

使用该集合，可以实现和set一样的功能，不同的是：
1. **不会阻止对象进行垃圾回收**
2. 只能添加**对象**
3. 不能遍历（不是可迭代的对象）、没有size属性、没有forEach方法

## WeakMap

类似于map的集合，不同的是：

1. **不会阻止对象进行垃圾回收**
```js
const John = { name: 'John' };
const map = new Map();
map.set(John, "studuent");

const weakMap = new WeakMap();
weakMap.set(John, 'student');

// 在Map中,因为map.set引用了John这个对象,所以它不会被GC回收
// 但是,在WeakMap中,不会阻止John这个对象被GC回收,没有那么严格
```

2. 它的键只能是**对象**
3. 不能遍历（不是可迭代的对象）、没有size属性、没有forEach方法

# 代理与反射

## 属性描述符
每一个属性除了 name 和 value 之外，还有一些其他的属性值

**怎么获取属性描述符？**
+  `Object.getOwnPropertyDescriptor(obj, property)` 获取那个对象的那个属性的属性描述符
+  `Object.getOwnPropertyDescriptors(obj)` 获取那个对象的所有属性的属性描述符

**那么有哪些内容呢？**
```js
{ value: 1, writable: true, enumerable: true, configurable: true }
```

- value：属性值
- writable：是否可写（可以重新赋值吗）
- enumerable：是可枚举的吗（可以遍历出来吗）
- configurable：属性描述符能不能修改

**怎么修改属性描述符呢？**
注意：
如果某个属性变成了*存储器*变量，那么，该属性是没有内存空间的
因此，如果配置了 get 或 set 那么就不能配置 *value 和 writable*

```js
Object.defineProperty(obj, property, {}); // 修改那个对象的那个属性的属性描述符，内容是什么 
Object.defineProperties(obj, {}); // 一起修改
```

比如，
```js
const obj = {
  a: 1,
  b: 2,
};

// 修改了"a"的属性描述符
Object.defineProperty(obj, "a", {
  value: 3,
  writable: false,
});

obj.a = 10; // 报错，不可写的

console.log(Object.getOwnPropertyDescriptor(obj, "a"));
```

上面的内容，看起来也没有什么很牛的东西

最最最最牛逼的是，defineProperty 中提供了 get 和 set 方法，如果*使用了 get/set 方法*，那么该属性就变成了*存储器*属性

读取该属性需要执行 get()
设置属性需要执行 set()

比如，
```js
const obj = {
  name: "张三",
  age: undefined,
};

Object.defineProperty(obj, "age", {
  get() {
    return obj._age; // 这个地方千万不能写 obj.age 不然就会死循环
  },
  set(val) {
    if (typeof val !== "number") {
      throw new TypeError("年龄必须是一个数字");
    }
    if (val < 0) {
      val = 0;
    } else if (val > 200) {
      val = 200;
    }
    obj._age = val;
  },
});

obj.age = "13"; // 报错
console.log(obj.age);
```

可以对外界进行控制，要访问某个值或设置某个值都可以被 get 和 set 函数监控到

一个应用——**数据控制视图**：
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <p>
      <span>姓名：</span>
      <span id="spanName"></span>
    </p>
    <p>
      <span>年龄：</span>
      <span id="spanAge"></span>
    </p>

    <script>
      const spanName = document.getElementById("spanName");
      const spanAge = document.getElementById("spanAge");

      Object.defineProperties(this, {
        spanName: {
          get() {
            return spanName.innerHTML;
          },
          set(val) {
            spanName.innerHTML = val;
          },
        },
        spanAge: {
          get() {
            return spanAge.innerHTML;
          },
          set(val) {
            spanAge.innerHTML = val;
          },
        },
      });
    </script>
  </body>
</html>
```

## 反射 Reflect

简单来说呢，就是将一些底层的东西，变成 API 实现。为了适配**函数式编程**，减少魔法。

什么是魔法呢？就是不能这个动作不能被修改，比方说，= 这是赋值符号，看到这个动作就是要赋值了，但是赋值的底层不能被开发者修改，将那个值赋值个那个变量。

就是说，反射将一些常见的*魔法（=，new，in...）都做成 API*了，开发者可以通过反射干预底层实现

- `Reflect.set(target, propertyKey, value)`: 设置对象target的属性propertyKey的值为value，等同于给对象的属性赋值
- `Reflect.get(target, propertyKey)`: 读取对象target的属性propertyKey，等同于读取对象的属性值
- `Reflect.apply(target, thisArgument, argumentsList)`：调用一个指定的函数，并绑定this和参数列表。等同于函数调用
- `Reflect.deleteProperty(target, propertyKey)`：删除一个对象的属性
- `Reflect.defineProperty(target, propertyKey, attributes)`：类似于Object.defineProperty，不同的是如果配置出现问题，返回false而不是报错
- `Reflect.construct(target, argumentsList)`：用构造函数的方式创建一个对象
- `Reflect.has(target, propertyKey)`: 判断一个对象是否拥有一个属性
- 其他API：

[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)

```js
const obj = {
  a: 1,
  b: 2,
};

obj.a = 10; // 这句话与下面这句话是一样的
Reflect.set(obj, "a", 10);

console.log(obj.a); // 这句话与下面这句话是一样的
console.log(Reflect.get(obj, "a"));

// 功能都是相同的，使用Reflect可以一眼看出是在操作底层
```

## 代理 Proxy

代理呢，就是在 target 对象上套了一层，外界要获取 target 的相关信息，其实是在问代理要东西。

而且，*代理是不占内容空间的*，

就好像，target 是当事人，proxy 是代理律师。法院还是对方当事人找的都是代理律师，而不是直接找当事人

**创建一个代理对象**
```js
// 代理一个目标对象
// target：目标对象
// handler：是一个普通对象，其中可以重写底层实现
// 返回一个代理对象
new Proxy(target, handler)
```

**举个例子：**
```js
const obj = {
  a: 1,
  b: 2,
};

const proxy = new Proxy(obj, {
  set(target, propertyKey, value) {
    return Reflect.set(target, propertyKey, value);
    // target[propertyKey] = value;
    // return true; // 必须返回boolean不然会报错
  },

  get(target, propertyKey) {
    console.log("访问target，其实要过一遍代理对象");
    return Reflect.get(target, propertyKey);
    // return target[propertyKey];
  },
});

proxy.a = 10;

console.log(proxy.a);
```

## 【应用】观察者模式

Object.defineProperty() 方式实现，但是还是有问题，如果手动修改 target，ob 并不会检测到 target的修改，从而对页面进行修改。而且，这种方式在内部维护了一个新的对象 ob，会造成内容空间的浪费。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="container"></div>
    <button id="add">+1</button>
    <script>
      // 观察者模式
      function observer(target) {
        const app = document.getElementById("container");
        const ob = {};
        const keys = Object.keys(target);
        for (const key of keys) {
          Object.defineProperty(ob, key, {
            get() {
              return target[key];
            },
            set(val) {
              target[key] = val;
              render();
            },
            enumerable: true,
          });
        }

        render();

        function render() {
          let html = "";
          for (const key of Object.keys(ob)) {
            html += `
          <p><span>${key}: </span><span>${ob[key]}</span></p>
        `;
          }

          app.innerHTML = html;
        }

        return ob;
      }

      const target = {
        a: 1,
        b: 2,
      };
      const ob = observer(target);
      const btn = document.getElementById("add");
      btn.addEventListener("click", () => {
        ob.a++;
      });

      console.log(ob);
    </script>
  </body>
</html>
```

Proxy 代理方式实现
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="container"></div>
    <button id="add">+1</button>
    <script>
      // 观察者模式
      function observer(target) {
        const app = document.getElementById("container");
        const proxy = new Proxy(target, {
          get(target, propertyKey) {
            return Reflect.get(target, propertyKey);
          },
          set(target, propertyKey, value) {
            Reflect.set(target, propertyKey, value);
            render();
            return true;
          },
        });

        render();

        function render() {
          let html = "";
          for (const key of Object.keys(target)) {
            html += `
          <p><span>${key}: </span><span>${target[key]}</span></p>
        `;
          }

          app.innerHTML = html;
        }

        return proxy;
      }

      const target = {
        a: 1,
        b: 2,
      };
      const ob = observer(target);
      const btn = document.getElementById("add");
      btn.addEventListener("click", () => {
        ob.a++;
      });

      console.log(ob);
    </script>
  </body>
</html>
```

那么 Proxy 是给 target 直接套了一层，而 defineProperty 是给 target 的每一个属性加了 get/set 方法。

自然外层的 target 新增属性什么的，defineProperty 是检测不到的，好吧，好像 proxy 也检测不到。但是 proxy 可以检测 ob 的改变，进行页面的重新渲染。defineProperty 检测不到 ob 新增属性的修改。

# 类型化数组

## 数字存储的前置知识

1. 计算机必须使用固定的位数来存储数字，无论存储的数字是大是小，在内存中占用的空间是固定的。
2. n位的无符号整数能表示的数字是2^n个，取值范围是：0 ~ 2^n - 1
3. n位的有符号整数能表示的数字是2^n个，取值范围是：-2^(n-1) ~ 2^(n-1) - 1
4. 浮点数表示法可以用于表示整数和小数，目前分为两种标准：

5. 32位浮点数：又称为单精度浮点数，它用1位表示符号，8位表示阶码，23位表示尾数
6. 64位浮点数：又称为双精度浮点数，它用1位表示符号，11位表示阶码，52位表示尾数

7. JS中的所有数字，均使用双精度浮点数保存

## 类型化数组

类型化数组：用于优化多个数字的存储

具体分为：

- Int8Array： 8位有符号整数（-128 ~ 127）
- Uint8Array： 8位无符号整数（0 ~ 255）
- Int16Array: ...
- Uint16Array: ...
- Int32Array: ...
- Uint32Array: ...
- Float32Array:
- Float64Array

如何创建数组

```js
new 数组构造函数(长度)

数组构造函数.of(元素...)

数组构造函数.from(可迭代对象)

new 数组构造函数(其他类型化数组)
```

得到长度

```js
数组.length   //得到元素数量
数组.byteLength //得到占用的字节数
```

其他的用法跟普通数组一致，但是：
- 不能增加和删除数据，类型化数组的长度固定
- 一些返回数组的方法，返回的数组是同类型化的新数组
