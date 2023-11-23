### git 基础

#### 获取 git 仓库

##### 初始化本地目录为 git 仓库

`git init` 指令会在项目目录下创建一个名为 `.git` 的子目录，这个子目录含有你初始化的 git 仓库中所有必须的文件

初始提交

```cmd
$ git add *.c
$ git add LICENSE
$ git commit -m 'initial project version'
```

##### 克隆现有的仓库

`git clone https://github.com/repositoryName`
git 支持多种数据传输协议，以上使用的是 https 协议，也可以使用 `git://` 协议或 ssh 传输协议

#### 记录每次更新到仓库

工作目录下的文件不外乎两种状态：已跟踪或未跟踪
已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照有它们的记录
除去已跟踪的文件外，其他文件都属于未跟踪文件，它既不存在于上次快照的记录中，也没有被放入暂存区

检查当前文件状态：`git status`
状态简览：`git status -s` (新添加的未追踪文件: ??, 新添加到暂存区: A, 修改过的文件: M)