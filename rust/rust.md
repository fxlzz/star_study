> 小白学 rust... 确实跟其他的编程语言都不一样呀，不亏是对标 C++ 的

# 搭环境
官方安装地址：[rust](www.rust-lang.org/tools/install)
装之前需要装编译工具：[Build Tools for Visual Studio 2022](https://visualstudio.microsoft.com/zh-hans/visual-cpp-build-tools/?spm=a2ty_o01.29997173.0.0.3c685171FLJ7TW)

下载完成后，可通过 `cargo --version` 和 `rustc --version` 进行检验
- `cargo`： *包管理*工具和*构建*工具，相当于`npm + Webpack + Babel`的结合体
- `rustc`： rust 是一门编译语言，rustc 就是它的编译器

开发环境：idea - rustRouter / vscode + 插件

升级rust： `rustup update`
卸载rust: `rustup self uninstall`

## 认识 cargo
上文说到，cargo 相当厉害

主要功能：
- 依赖管理 -> 自动下载、解析和管理项目依赖（rust 项目中依赖包被称之为“create”)
- 项目构建 -> 会调用 rustc 编译项目
- 创建项目 -> 类似于 `npm + webpack` 使用`cargo new [project-name]` 创建项目
	- 项目会自动创建 `.git`
	- `src`目录
	- `cargo.toml`这个文件，记录了当前rust项目中依赖的包，类似于`package.json`
- 测试与发布 -> `cargo test` 以及 `cargo publish` 发布到`cargo.io`上

常用命令合集：
```bash
cargo new my_project        # 创建新项目
cargo build                 # 编译项目
cargo run                   # 编译并运行
cargo test                  # 运行测试
cargo add serde             # 添加依赖（需安装 cargo-edit）
cargo update                # 更新依赖
```

注意：
`cargo update` ： 这个命令会忽略 `cargo.lock` 文件，计算出所有符合`cargo.toml`声明的最新版本

项目目录结构：
```bash
my_project/
├── Cargo.toml    # 项目配置与依赖声明
├── Cargo.lock    # 依赖版本锁定文件（自动生成）
|—— .gitignore    # git 工程 
└── src/
    └── main.rs   # 主程序入口（二进制项目）或 lib.rs（库项目）
```

使用 `rustc` 来编译`.rs` 的源文件，在windows上会生成 `.exe（可执行文件）` | `.pdb（可调式文件）` 

# 猜字游戏
> 来自 rust 官网， 确实与其他的语言不太一样

注意几点：
1. `mut` 定义可变变量
2. `Result<xxx>`  Result 类型是*枚举*，那么说明，前面的操作可能成功也可能失败，需要放一手，用`expect`来捕获错误
	1. `Result` 的成员 `OK` 和 `Err`
	2. `OK` -> 表示成功
	3. `Err` -> 表示失败
3. *枚举* 可以通过 `match` 一起使用，是一种条件语句
4. `xxx!` 表示这是一个宏定义变量
5. `println!("you guessed: {}", guest);` 变量输出
6. 导包： Rust 有少量标准库，这些项被称作 *prelude* ，如果不在标准库中，需要用 `use` 关键字来显式引入 `use std::io;`
7. `crate` 是一个Rust代码包，有*库crate*的，也有*二进制crate*

# 通用编程概念
## 变量和可变性
> rust 还是有些不一样的

### 定义变量
在 `rust` 中有三种方式来定义变量
+ `let` 默认*不可变* ，不可以重新赋值
	+ `let mut x = 5;` 设置 `mut` 可以赋值 
+ `const` 定义常量 & 必须显式声明类型 & 不可遮蔽
+ `static` 静态变量 & 存在于程序整个生命周期 & 有固定的内存地址 & 全局配置或底层开发

### 遮蔽 Shadowing
同一个作用域下，允许重新定义同名变量

*变量遮蔽* 最经典的应用场景： 数据转换 & 改变可变性
+ 数据转换
```rust
let spaces = " "; // spaces -> string
let spaces = spaces.len(); // spaces -> 现在变成数字了！
```
+ 改变可变性
```rust
let mut x = 5; // 可变的
x = 6;

let x = x; // let 默认的是不可变的，通过遮蔽，x 又变成不可变的了
// x = 7; // 这里会报错
```

注意：
+ *遮蔽* 是创建了一个‘新’变量，而`mut`是在修改‘旧’变量的值
+ *遮蔽* 是有作用域的

## 数据类型
### 标量类型
#### 整数

| 长度   | 有符号   | 无符号   |
| ---- | ----- | ----- |
| 8位   | i8    | u8    |
| 16位  | i16   | u16   |
| 32位  | i32   | u32   |
| 64位  | i64   | u64   |
| 128位 | i128  | u128  |
| arch | isize | usize |

`isize & usize` ： 根据操作系统的位数来确定`size`

整数相除 -> 会向下取整
`let floored : i32 = 4 / 3; -> result: 1`

#### 浮点
两种 `f32` & `f64` 浮点型都是*有符号*的
`f32` 单精度浮点型
`f64` 双精度浮点型

#### 布尔 bool
有两个值： `true` & `false`

注意哈： 跟 js 不同，会将 0 / "" 等自动转化为 false

#### 字符 char

### 复合类型
元组 & 数组

#### 元组
可以将*多种类型*的多个值组合到一个复合类型，还可以*解构*
```rust
fn main() {
	let tup: (i32, f64, i8) = (500, 0.64, -1);
	let (x, y, z) = tup;
}
```

```rust
fn main() {
	let nums = (10, 6.4, 1);
	let n1 = x.0;
	let n2 = x.1;
	let n3 = x.2;
}
```

#### 数组












