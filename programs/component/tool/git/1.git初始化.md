[TOC]

# 文件

# 创建仓库
使用当前目录作为Git仓库，我们只需使它初始化
git init
使用我们指定目录作为Git仓库
git init [name]

# 配置
git config --list
# 基础操作
git config -e [--global]
只能在git文件夹下使用
添加文件到暂存区
git add [list] ...
添加当前目录所有文件
git add .
添加全部
git add -A
删除工作区文件，并将操作写入暂存区
git rm [list]...
git mv

# 提交
提交暂存区到仓库区
git commit -m "commit"
提交工作区到暂存区
git commit -a
提交时显示diff信息
git commit -v

# 分支
列出所有本地分支
git branch
列出所有远程分支
git branch -r
列出所有本地分支和远程分支
git branch -a
新建一个分支，但依然停留在当前分支
git branch [branch-name]
新建一个分支，并切换到该分支
git checkout -b [branch]
切换到指定分支，并更新工作区
git checkout [branch-name]
切换到上一个分支
git checkout -
合并指定分支到当前分支
git merge [branch]
删除分支
git branch -d [branch-name]
删除远程分支
git push origin --delete [branch-name]
git branch -dr [remote/branch]

