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
