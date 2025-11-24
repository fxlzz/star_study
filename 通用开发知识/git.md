# git 
> git 也是团队协作中，必不可少的一部分，太牛逼了，我只能说

## 基本概念
git 中的三个仓库
+ 本地仓库
+ 暂存区
+ 远程仓库
  
<img width="559" height="151" alt="image" src="https://github.com/user-attachments/assets/29ffcf53-869f-44b2-933e-00d69030b928" />

**本地仓库**： 其实可以理解为，在编译器中 未提交 的代码
**暂存区**：见名知意
**远程仓库**：云端开源社区，比如，gitee / github...

## 快速上手
```bash
git clone xxx # 克隆项目
git pull # 获取最新的代码
# 在本地打开项目，修改代码，git 状态改变
git add . # 将所有状态改变的代码加入暂存区
git commit -m "说明信息" # 加入本地仓库
git push  # 将本地仓库更新到远程仓库 
```
但这里需要注意的是，首次 push 会有报错
```bash
# 将远程仓库地址关联到本地，并命名为 origin
git remote add origin <remote-repo-url>
```
使用 -u 或 --set-upstream 参数，建立本地分支和远程分支的追踪关系
```bash
git push -u origin main # 本地仓库提交到远程的主分支名称 main
```
---
恭喜你，git 的 helloworld 你会了

## 切换分支
使用场景：什么时候会切换分支呢？
- 不想破坏主分支代码
- 搞点小实验
- 等

**创建并切换到新分支**
`git switch -c <branch-name>` 也可用 checkout 相关指令

**删除分支**
`git switch -d <branch-name>` -D 强制删除

**本地分支与远程分支的对应关系**
`git branch -vv` 

**查看本地分支**
`git branch -v`

## 查看状态
`git status` 被git追踪的状态[修改、删除、添加、加入暂存区等] 
`git log` 查看git的提交日志
`git log --pretty=oneline --graph --abbrev-commit <branch-name>` 更友好的方式查看日志

## 暂存代码
使用场景：当你A需求写到一半，来了一个紧急的需求（另一个分支），需要你马上去支持。

```bash
# 保存当前工作区的代码
git stash save "暂存说明"

# 查看暂存的代码
git stash list

# 取出暂存代码【存是以栈的方式】
git stash pop # 取第一个
git stash pop stash{n} # 指定第几个

# 删除
git stash drop stash{n}
```
注意：stash只能保存被git追踪的代码，如果你新增了一个文件，这个文件不会被git追踪
所以，
```bash
git add .
git stash save "xxx"
```

## 合并冲突
一般，合并冲突会发生在，更改了同一个文件 / 很久没有 pull 代码 / merge / push等
处理起来，其实很简单，在编译器中都有图形化展示，有冲突的代码
























