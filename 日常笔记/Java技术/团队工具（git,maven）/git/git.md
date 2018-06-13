# git

[git 用法](http://www.bootcss.com/p/git-guide/)

## 创建新仓库

新建一个git仓库指令：
```git init``` 

## 检出仓库

### 创建一个本地仓库的克隆版

```git clone $PATH```

## 添加和提交

### 添加

改动计划，将其提交到缓冲区：
```
git add 
```

### 提交

提交改动
```
git commit -m "代码提交信息"
```
此时，代码改动已经提交到HEAD，但还没到远程仓库。

## 推送改动

改动已经在本地commit后，提交到远程仓库使用以下指令：
```
git push origin <branch>
```

如果还没有克隆现有仓库，并欲将仓库连接到某个远程服务器，使用以下指令：
```
git remote add origin <server>
```

## 分支

### 创建分支
```
git checkout -b <branch_name>
```

### 切换分支

```
git checkout <branch_name>
```

### 删除分支
```
git branch -d <branch_name>
```

### 分支推送到远程仓库
```
git push origin <branch_name>
```

## 更新和合并

### 更新本地仓库到最新改动
```
git pull
```

### 在本地获取并合并远端改动
```
git merge <branch_name>
```

### 手动合并
自动合并并非次次都能成功，并可能导致冲突。需要在本地修改这些文件解决冲突。修改完后，需要执行：
```
git add <filename>
```

### 查看改动

```
git diff <source_branch> <target_branch>
```
#### 汇总
```
$ git merge --squash # 合并代码
$ git rebase --interactive # 交互变基
$ git push origin --delete <branch> # 删除远程分支
$ git branch --set-upstream-to=<uri> # 设置远程分支
$ git add remote <name> <uri> # 设置源
$ git rm --cached <file> # 删除暂存
$ git remote add <repository> <uri> # 增加源
$ git add -u # 暂存已跟踪文件
$ git add -A # 暂存所有文件
$ git tag <tagname> # 为当前版本打tag
$ git tag -a <tagname> <hash> -m <comment> # 为特定版本打tag
$ git push --tags # 推送tag到源
$ git reset --hard <hash> # 强制回滚
$ git reset --hard HEAD~3 # 强制回滚三个版本
$ git submodule add <url> <dir> # 增加子模块
$ git submodule update --init --recursive # 递归更新子模块
$ git filter-branch --tree-filter 'rm -f <filename>' # 过滤历史提交
```