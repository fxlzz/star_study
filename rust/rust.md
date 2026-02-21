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

# 变量和可变性
