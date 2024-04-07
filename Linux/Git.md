# Git基本操作

## Permission denied (publickey).问题

![254c97810ab8d4a64f2f2d44a982e33](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/254c97810ab8d4a64f2f2d44a982e33.png)

## 基本指令

![image-20240116131105362](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240116131105362.png)

### 1 初始化:
`git init`
### 2 配置
  - `git config user.name “”` 
  - `git config user.email “”`
  - 列举所有配置：`git config -l`
  - 重置配置：`git config --unset [配置名]`
  - 将配置项生效与所有git仓库：`git config --global [配置名]`
### 3 基本概念
  - *工作区 ->  add   ->  版本库[暂存区  ->  commit  ->  master]*
  - 修改工作去的内容会写入对象库的一个新的git对象中
  - HEAD指向一个分支最新的一次修改
### 4 基本操作
  - `git add`
  - `git commit -m “”`
  - `git log`:打印日志
  - `git log --graph --abbrev-commit`:可视化分支
  - `git status`：查看是否被修改（暂存区）
  - `git diff [filename]`：查看修改内容
### [5 版本回退](https://www.cnblogs.com/qdhxhz/p/9757390.html)

#### 工作区(Workspace)、暂存区(Index/Stage)、版本库(Repository)、远程仓库(Remote)

![image-20240228211020338](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240228211020338.png)

   - `git reset --soft [HEAD]`: 回退**版本库**的内容
   - `git reset --mixed [HEAD]`: 回退**暂存区、版本库**的内容
   - `git reset --hard [HEAD]`:回退**工作区、暂存区、版本库**的内容
### 6 撤销修改
- `git checkout --[filename]`:将**工作区**撤销到最后一次add/commit
- `git reset --mixed HEAD`或`git reset --hard HEAD`:**工作区和暂存区**均被修改，回退到HEAD(当前版本)，(HEAD^表示回退到上一个版本，HEAD^^表示回退到上两个版本)
- `git reset --hard HEAD^`：**工作区、暂存区、版本库**均被修改，且commit之后没有push

### 7 删除文件
  1. 法一：`rm [filename]`   ->  `git add [filename]`   ->  `git commit -m “”`
  2. 法二：`git rm [filename]`  ->  `git commit -m “”`

# Git分支操作
### 1.查看分支 
- `git branch`
- `tree git`
### 2.创建分支
- `git branch [name]`
- `git checkout -b [name]`:创建并切换分支
- `git checkout [branchname]`:将HEAD指向其他分支--切换分支
### 3.合并分支
`git merge [branchname]`
### 4.删除分支
`git branch -d [branchname]`**只能在其他分支上删除该分支**
### 5.合并模式

### 6.保存工作区修改
当在dev分支上开发新代码时，master分支发现bug，此时切换到master，需要保存dev分支的修改内容--->`git stash`:保存工作区代码
- `git stash pop`:取出stash中的内容
- `git stash list`:查看stash区内容

### 7. 强制删除分支
`git branch -D [branchname]`

### 8. 重命名分支

`git branch -M []`

# Git远程操作
## 远程推送
1. `git add`
2. `git commit [name] -m “” `
3. `git push origin master:[远程分支名字]`：origin表示往远程推送；master表示推送本地的mater分支

## 远程拉取仓库
`git pull origin master:[本地分支]`：将远程仓库中的新内容拉取到本地仓库；*origin master 表示远端仓库的master分支，若远程分支名与本地分支名一样，可省略拉取 + 合并*
      

## 忽略文件`.gitignore`
- ` vim .gitignore `   更改ignore文件
- ` *[文件后缀] `      表示忽略此类文件   例如： *.jpg  
- ` ![文件名] `         表示不排除该文件   例如：!name.jpg
- `git check-ignore -v [文件名]`   查看该文件被忽略的原因，以及在ignore文件中的位置

### 配置命令别名
` git config --global alias.[别名] ‘[命令名]’ `

## 操作标签
### 1.添加
  - `git tag [标签名] [commit id]`（默认为最新的一次提交）；
  - `git tag -a [标签名] -m “描述信息”  [commit id]`
### 2.查看
  - `git tag`
  - `git show [标签名]`
### 3.删除标签： 
  - `git tag -d [标签名] `
### 4.推动标签：
  -  `git push origin [标签名]`
  -  推送所有标签：`git push origin --tags`
### 5.将删除的标签推送到远端：
  -  `git push origin :[标签名]`    （冒号后面跟本地内容）

# 协作开发
## 1、同一分支下进行
### 协作开发
1. 所有操作在本地分支上完成，最后用push推送到远端分支
- 查看远程分支：`git branch `     
- 查看远端与本地的分支： `git branch -a`
2. 将远程分支拉到本地： `git pull`
3. 查看本地分支与远程分支是否建立连接： `git branch -vv`
4. `git checkout -b [dev] [origin/dev]` ： 创建分支dev并切换到dev链接，并将远程分支origin/dev与本地分支建立连接，建立连接之后就可以直接使用`git push`而不用 `git push origin dev : dev`
- 其他链接方式：`git branch --set-upstream-to=[origin/dev] [dev]`
5. 解决dev分支合并时的冲突：
- `git push`   ->   发成冲突    ->    `git pull`：先将远端dev分支的内容拉到本地   ->   在本地修改   ->    add,commit,push

### 将远端的dev分支合并到master分支
- 方法一：在远端仓库的**Pull Requests**提交申请单，经过审查员审核并选择是否merge
- 方法二：在本地master分支上 :
`git pull master`:让本地master保持远端master的最新状态(此时本地master可能没有内容，无法知道远端的dev merge到master是否会发生冲突)  ->  在dev分支上`git merge master`（有冲突尽量在dev上解决）  ->  切换到master：`git merge dev`  ->  `git push master`

## 二、在不同分支下进行
### 协作开发
1. 开发者一对开发者二的分支二进行开发： 
  - `git pull`: 拉取仓库内容（`git pull [分支名]`可拉取分支内的内容）
  - `git checkout -b [分支名] origin/[远程分支名]`：穿件分支、建立连接、切换到该分支
2. 开发者二中添加开发者一对分支二的修改
  - `git branch --set-upstream-to=origin/[分支名] [分支名]`：建立连接
  - （若想要拉取分支下某一文件的改动必须先建立连接，再来使用`git pull`）

### 将内容合并到master 
branch2先合并到master，若之后branch1再想合并到master分支，可能发生冲突，所以在branch1上先合并并解决冲突
- `git checkout master`  ->   `git pull`
- `git checkout branch1`  ->  `branch1 merge master`   ->  解决冲突/直接合并          ->  `git push origin branch1`
- 将branch1合并到master：**pull request**

## 解决git branch -a 在本地打印出已被删除的远程分支的方法
- `git remote show origin`:展示远程分支的情况
- `git remote prune origin`:修剪远程已被删除的分支

# 企业级开发流程





# pr

1. fork
2. clone到本地
3. 在本地编写
4. `add`  `commit`   `push origin main`
5. Comtribute:open a pull request: create pull requset

![image-20240116141800612](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240116141800612.png)
