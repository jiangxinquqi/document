# 最佳实践

## 配置ssh免密登陆

```shell
# Step 1: 生成秘钥,一直回车
ssh-keygen -t rsa -C <comment>
# Step 2: 将HOME/.ssh/id_rsa.pub 的内容复制到仓库的秘钥管理中
```

## 中文转码问题

```shell
git config --global core.quotepath false
git config --global gui.encoding utf-8
git config --global i18n.commit.encoding utf-8
git config --global i18n.logoutputencoding utf-8
# 设置环境变量
set LESSCHARSET=utf-8
```

## git squash合并commit

```shell
# dev分支合并到master分支,dev分支多个commit合并
# step 1 : 同步local/master 和 origin/master
# step 2 : git checkout dev;git rebase -i master; (此处分支也可以为commit-id)
# step 3 : 进入编辑页面，保持需要保留的commit为pick,其他commit修改为s（squash缩写）
# step 4 : git push -f
```

## 修改本次提交信息

```shell
git commit --amend # 修改最近一次提交的提交信息
```

## A分支的commit-id提交合并到B分支

```shell
# 在B分支执行
git cherry-pick <commit-id>
```

## 版本回退

### 1）未使用git add缓存代码时

```bash
git checkout --file_path_name # 新建的文件需要自己删除。
git checkout . # 放弃所有本次修改
```

### 2）已经使用git add缓存代码

```bash
git reset HEAD file_name
git reset HEAD . # 放弃所有缓存
```

### 3）已经使用git commit提交

```bash
git reset --hard [version] # 回退到指定版本
git commit --amend  # 修改注释
```

### 4）清除untracked files和目录

```bash
git clean -df # 删除untracked files和目录
```

# 常用指令

## git config

```bash
# 列出所有的配置
git config --global -l

# 常用配置
git config --global user.name xiaoxiao
git config --global user.email xiaoxiao@example.com

# git提交注释控制台输出编码改为utf-8
git config --global core.quotepath false
git config --global i18n.commit.encoding utf-8
git config --global i18n.logoutputencoding utf-8
```

## git branch

```bash
查看本地分支：git branch
查看远程分支：git branch -r
查看远程分支：git branch -a //显示远程分支
删除远程分支：git branch -d [branch_name] // -d只能删除已经合并的分支。-D强制删除
```

## git stash

```bash
git stash //把所有本地修改都放到暂存区
git stash pop //恢复stash@{0}工作现场到当前
git stash list //查看stash队列
git stash pop stash@{num} //恢复指定的工作现场
git stash clear //清除stash队列
```

## git remote

```bash
git remote -v  # 查看远程仓库
git remote add [alias] [url]  # 添加远程仓库
git remote rename [old-alias] [new-alias] # 重命名
git remote set-url [alias] [url] # 修改关联的远程仓库
```

## git add

```bash
git add -u # -u 表示只添加已经被git索引中的文件，不添加新文件
git add -p # -p 表示可选择文件的部分提交
```

## git commit

```bash
git commit -a -m "xxxx" # git add + git commit -m组合
git commit --amend # 修改最近一次提交的提交信息
git commit -c <commit> # 使用某一次提交的提交信息
```

## git pull

```bash
git pull origin dev # 当前分支覆盖远程分支
```

## git checkout

```bash
git checkout -b dev（本地分支名） origin/dev（远程分支名）
git checkout <commit> # 切换到某一个提交
git checkout <commit> --file.xml # 检出某一个提交的某一个文件
```

## git reset

```bash
git reset --hard HEAD^1 # 重置到上一个提交
git reset --soft HEAD^1 # 重置到上一个提交，当前提交内容放到暂存区
```

## git fetch

```bash
git fetch -p  # 清除git远程跟踪
```

## git cherry-pick

```sh
git cherry-pick <commit>
# 不直接提交，将摘取内容放入暂存区，可修改后再提交
git cherry-pick --no-commit <commit>
# 摘取一个区间的提交，左开右闭，不包含<commit1>
git cherry-pick <commit1>..<commit2>
# 摘取一个区间的提交，包含<commit1>
git cherry-pick <commit1>^..<commit2>
```



## git tag



```bash
git tag  # 查看版本
git tag [name] # 给当前提交打标签
git tag -d [name] # 删除标签
git tag -r # 查看远程标签
# 给HEAD对应提交打标签，标签可通过 git push origin <tag> 推送到远程仓库
git tag <tag>
git tag <tag> <commit> # 给某个提交打标签
git tag # 列出所有标签
git tag -d <tag> # 删除标签
```



## 其他

| 命令         | 说明                                       | 例子                     |
| ------------ | ------------------------------------------ | ------------------------ |
| `git revert` | 回滚某一提交，会生成新的提交               | `git revert <commit>`    |
| `git blame`  | 查看文件每一行是由谁提交的                 | `git blame bin/start.sh` |
| `git rm`     | 删除已跟踪的文件，需要运行`git commit`提交 | `git rm bin/start.sh`    |
| `git clean`  | 删除未跟踪的文件                           | `git clean bin/start.sh` |
| `git log`    | 查看提交历史                               |                          |
| `git diff`   | 比较提交之间的差别                         |                          |
| `git stash`  |                                            |                          |



# 实践建议



## 一个提交只做一件事情



例如，开发新功能和修改配置不要放在一个提交，修改不同的Bug应拆成多个提交。通过减小提交粒度，便于进行代码检视，某个提交发现问题时能快速回滚（`git revert`），查看提交历史也能清楚地知道每个提交的目



开发一个大功能也可视情况拆分成多个提交，例如实体类定义、业务逻辑实现分开提交。



## 提交信息写清晰



反例：“修复Bug”，修复了什么Bug？提交信息应简要说明该提交的目的，更进一步，可以关联issue、需求编号、问题编号，也可以添加多行更详细的说明。



## 不需要每天下班提交代码来保存工作



有的规范要求：“每天下班提交手头的代码。”不需要，不要提交未完成的工作**并push**，每个提交应该是完整的。



## 不要提交多余的文件、代码、注释、临时代码



例如，工程文件、`target`目录等添加到`.gitignore`。提交前执行`git status`检查一下要提交的文件，执行`tig`或者`git diff`浏览要提交的内容。



## 自己开发的分支尽量基于最新的主干分支



checkout前运行`git pull`更新主干分支，开发过程中运行`git fetch`获取各分支最新内容。



在自己的分支上开发的过程中，发现主干分支更新了，且**该分支没有push过**，可执行`git rebase`到主干分支（工作区的内容需要先`git stash`才能`git rebase`）；若已经push过了，则禁止rebase。



这样能尽量减少图形上分支的交叉，便于管理员观察、分析开发情况。



## 及时删除过期分支



提PR时勾选“合并后删除分支”，同时删除本地已经合并了的分支。