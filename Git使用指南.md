## 基本概念
* 工作区（当前的工作目录）
* 暂存区（Add等操作将修改保存至暂存区）
* 本地版本库（Commit操作将缓存区的修改提交到本地版本库）
* 远程版本库（本地版本push到远程版本库）

## 常用操作

### Git 常用命令使用
命令行里git的命令列表以及解释：
```shell
git clone <address>：复制代码库到本地。
git add <file> ...：添加文件到代码库中。
git rm <file> ...：删除代码库的文件。
git commit -m <message>：提交更改，在修改了文件以后，使用这个命令提交修改。
git pull：从远程同步代码库到本地。
git push：推送代码到远程代码库。
git branch：查看当前分支。带*是当前分支。
git branch <branch-name>：新建一个分支。
git branch -d <branch-name>：删除一个分支。
git checkout <branch-name>：切换到指定分支。
git log：查看提交记录（即历史的 commit 记录）。
git status：当前修改的状态，是否修改了还没提交，或者那些文件未使用。
git reset <log>：恢复到历史版本。
git push origin HEAD --force：使本地和远程的内容都回退到log对应的状态
```
代码库应该很好理解，就是存放代码的地方，而在 git clone 里，代码库一般指的是远程的代码库，即 github 给出的链接。而分支则是开发的一个阶段或者一个旁系版本，至于怎么定则取决于使用者了。例如，有一个分支叫做stable，代表里面的代码是经过测试的、稳定的；另一个分支叫dev，则是保存开发中的代码，不一定经过足够测试。

### 一般流程

- 编辑文件，更新代码。
- 添加代码到当前待提交的更改列表中：git add <修改的文件>。
- 提交当前修改作为一个记录：git commit -m '修改了<修改的文件>，原因是：……'。
- 更新代码：git push。
- 不要用git pull，用git fetch和git merge代替它，参考：http://longair.net/blog/2009/04/16/git-fetch-and-merge/。

### Git 分支操作
```shell
# 创建分支
git branch feeder

# 查看当前分支
git branch

# git切换分支
git checkout feeder

# 创建并切换分支
git checkout -b feeder

# 合并分支
git merge feeder

# 删除分支
git branch -d feeder
```
参考：
* [分支的新建与合并](https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)

### git 拉去代码
```shell
git clone <远程仓库地址>

# 从远程拉取数据
git pull <远程主机名> <远程分支名>:<本地分支名>

# 将远程主机更新取回本地
git fetch <远程仓库地址>
```

### git 提交代码

```shell
# 将工作区的修改提交到暂存区
git add . # 提交当前目录及所有子文件
git add <file>

# 将暂存区内容提交到版本库
git commit -m "..."

# 将本地分支推送到远程主机
git push <远程主机名> <本地分支名>:<远程分支名>
```

### git 回退操作
三种模式
```shell
# 此为默认方式，不带任何参数的git reset，即时这种方式，它回退到某个版本，只保留源码，回退commit和index信息
git reset --mixed

# 回退到某个版本，只回退了commit的信息，不会恢复到index file一级。如果还要提交，直接commit即可
git reset --soft

# 彻底回退到某个版本，本地的源码也会变为上一个版本的内容
git reset --hard
```

示例：
```shell
# 回退所有内容到上一个版本
git reset HEAD^
# 回退a.py这个文件的版本到上一个版本
git reset HEAD^ a.py
# 向前回退到第3个版本
git reset --soft HEAD~3
# 将本地的状态回退到和远程的一样
git reset --hard origin/master
# 回退到某个版本 
git reset 057d
# 回退到上一次提交的状态，按照某一次的commit完全反向的进行一次commit
git revert HEAD
```

### 合并分支 merge

比如，如果要将开发中的分支（develop），合并到稳定分支（master），
```shell
# 切换master分支
git checkout master
# 执行合并操作
git merge <branchName>
# 如果有冲突，会提示你，调用git status查看冲突文件，解决冲突，然后调用git add或git rm将解决后的文件暂存。所有冲突解决后提交更改
git commit
# 合并后删除分支信息
git branch -d <branchName>
# 查看所有分支信息
git branch -a
```

## 更改提交用户名/邮箱
```shell
git config user.name "XXX"
git config user.email "XXX"
git config --global user.name "XXX"
git config --global user.email "XXX"
```

参考：
* https://alvinalexander.com/git/git-show-change-username-email-address

## 删除文件夹
```shell
git checkout master
# 本地文件也删除
git rm -r folder-name
# 本地文件不删除，只删除git记录
git rm --cached file1.txt
git commit -m "Remove duplicated directory"
git push origin branch_name
```

### 删除远程仓库脏文件
```shell
# 预览将要删除的文件
git rm -r -n --cached 文件/文件夹名称
# 删除
git rm -r --cached 文件/文件夹名称
# 提交
git commit -m "提交说明"
git push origin master
```
注： 修改本地`.gitignore`会更好

## 外部资料
* [Git 简单使用](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
* [Git Pro2](https://progit.bootcss.com/)
* [Github使用指南](https://github.com/NeuOL/neuola-legacy/wiki/github%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97)
* [git - 简明指南](http://rogerdudler.github.io/git-guide/index.zh.html)
* [Git版本控制与工作流](http://blog.jobbole.com/87410/)
* [ProGit](https://git-scm.com/book/zh/v2)
* [git-flow备忘清单](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)
