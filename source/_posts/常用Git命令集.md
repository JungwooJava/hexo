---
title: 常用Git命令集
date: 2021-01-15 02:09:57
tags: [Git]
---

2021年，公司代码库的保管逐步由SVN切换到Git了，因此需要熟练一些Git的命令行来满足日常工作的提交。

附本人常用的Git命令行如下：

```shell
#创建分支+切换到该分支
git switch -c dev

#切回master分支(不支持switch命令行是因为Git的版本过低)
git switch master

#dev分支的成果合并到master
git merge dev

#更推荐！合并分支
git merge --no-ff -m "merge with no-ff" dev

#删除分支
git branch -d dev

#解决冲突
git add readme.txt
git commit -m "fix conflict"


#查看分支
git branch

#添加
git add readme.txt

#提交
git commit -am "append something"
-a 告诉命令自动暂存已修改和删除的文件，但是您未告知Git的新文件不受影响
-m 使用自己提供的messege作为说明

#查看冲突
git status

#撤销修改
git checkout  -- readme.txt

#储存当前工作现场
git stash

#查看stash
git stash list

#恢复git stash
git stash apply

#恢复+删除stash
git stash pop

#恢复指定的stash
git stash apply stash@{0}

#删除stash
git stash drop

#从master创建分支
git checkout master
git checkout -b issue-101


#查看分支历史
git log --graph --pretty=oneline --abbrev-commit


#从版本库中彻底删除文件
git rm readme.txt
git commit -m "remove readme.txt"


#远程仓库克隆
git clone git@github.com:cx_jungwoo/piggy.git


#版本回退到上一个版本
git reset --hard Head^
或者
git reset --hard 1094a

#git历史命令（了解commit id）
git reflog

#查看远程库信息
git remote -v

#推送分支
git push origin master
git push origin dev

#推送分支失败解决办法
git pull
git status
git log --graph --pretty=oneline --abbrev-commit
git rebase

#打标签
git tag V1.0

#看标签
git tag

#删除标签
git tag -d  V1.0

#推送标签到远程
git push origin V1.0

#一次性推送全部标签
git push origin --tags

#删除远程标签
git push origin :refs/tags/v0.9

```





