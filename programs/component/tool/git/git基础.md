[TOC]

```
开始一个工作区（参见：git help tutorial）
   clone             克隆仓库到一个新目录
   init              创建一个空的 Git 仓库或重新初始化一个已存在的仓库

在当前变更上工作（参见：git help everyday）
   add               添加文件内容至索引
   mv                移动或重命名一个文件、目录或符号链接
   restore           恢复工作区文件
   rm                从工作区和索引中删除文件
   sparse-checkout   初始化及修改稀疏检出

检查历史和状态（参见：git help revisions）
   bisect            通过二分查找定位引入 bug 的提交
   diff              显示提交之间、提交和工作区之间等的差异
   grep              输出和模式匹配的行
   log               显示提交日志
   show              显示各种类型的对象
   status            显示工作区状态

扩展、标记和调校您的历史记录
   branch            列出、创建或删除分支
   commit            记录变更到仓库
   merge             合并两个或更多开发历史
   rebase            在另一个分支上重新应用提交
   reset             重置当前 HEAD 到指定状态
   switch            切换分支
   tag               创建、列出、删除或校验一个 GPG 签名的标签对象

协同（参见：git help workflows）
   fetch             从另外一个仓库下载对象和引用
   pull              获取并整合另外的仓库或一个本地分支
   push              更新远程引用和相关的对象

```

# 配置user信息
git config --global user.name xxx
git config --global user.email xxx

# config的三个作用域
缺省等同于local
local # 只对某个仓库有效
global # 对当前用户所有仓库有效
system # 对系统所有登录的用户有效

# 显示config的配置
git config --list --[global|local|system]

# git alias 别名

# 建立git仓库
两种场景：
## 把已有的项目代码纳入git管理
```
>cd 项目代码所在的文件夹
>git init
```
## 新建的项目直接用git管理
```
>cd 某个文件夹
>git init your_project # 会在当前路径下创建和项目名词同名的文件夹
>cd your_project
```

# 向仓库添加文件
工作目录-->暂存区-->版本历史

add：存入暂存区
commit：存入历史版本
push：提交到master


# add
git add -u：将文件的修改、文件的删除，添加到暂存区。
git add .：将文件的修改，文件的新建，添加到暂存区。
git add -A：将文件的修改，文件的删除，文件的新建，添加到暂存区。

# 给文件重命名


# 清除暂存区
git reset --hard

git log --all 查看所有分支的历史
git log --all --graph 查看图形化的 log 地址
git log --oneline 查看单行的简洁历史。
git log --oneline -n4 查看最近的四条简洁历史。
git log --oneline --all -n4 --graph 查看所有分支最近 4 条单行的图形化历史。
git help --web log 跳转到git log 的帮助文档网页


# 切换分支
git checkout xxx

---

detached head
分离头指针/home/cai/Downloads

git commit -am "commit"
不存入暂存区直接提交

git branch xxx commitid 将分离头指针产生的commit创建一个分支


gitk

---

HEAD branch

git branch -av

git checkout -b xx xx


git diff HEAD HEAD^


git diff HEAD HEAD^^

git bransh -d [id] 删除分支
git bransh -D [id] 强制删除

git branch -d branch_name:使用-d 在删除前Git会判断在该分支上开发的功能是否被merge的其它分支。如果没有，不能删除。如果merge到其它分支，但之后又在其上做了开发，使用-d还是不能删除。-D会强制删除。老师能加一讲课来讲讲merge和rebase的区别吗。

---
git commit --amend 对最近的commit修改message