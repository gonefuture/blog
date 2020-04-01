---
title: git入门简介 （转载自zooooooooy）
---
### git基础概念
git是目前最好的版本控制系统（没有之一）

#### branch
git里面核心的内容是分支（branch）,不同于svn的是，分支的成本很低，只记录变化的文件。git官方是非常推荐是使用分支来开发，我们在自己本地进行某些特性开发的时候，也是可以开辟一个本地分支。操作方式分为两种

*	自己本地建一个分支，可选择方式有两种，本地写好后自己merge到develop分支，删除本地分支。----个人建议采用这种方式
*	如果是业务特性，例如feature分支，可能对现有在develop上开发的其他工程师造成影响，且有两个人以上开发这个feature分支。或者需要为某些特性单独做一些特别的处理。这就需要推送到远程分支上面去。 ----个人建议如果一个人开发的话，开新的本地分支是有效的操作方式
操作命令如下
	*	git branch (name)
	*	git checkout (name)	
	*	git checkout -b (name) 是上面两个命令的合集
	*	git merge <name> git有个概念是当前分支，如果用命令行的话可以清晰的看到，这个merge的意思是在将<name>分支合并到当前分支上
	*	git push origin (name) 推送到远程分支	
	*	git branch -d (name) 删除本地分支 git中一个远程分支对应一个本地分支，名字可以不一样，不过建议最好保持一致。
	*	git push origin :(name) 冒号前面是空的代表将一个空推送到远程分支，即是删除远程分支
	*	git push origin --delete (name) git1.7 以上支持这个方式 建议删除时不要填写master,develop等比较关键的分支

#### commit	
git第二个概念是commit，提交记录，每次提交都会产生一个sha1-hash值，Git仓库中内容和头信息（Header）的一个校验和(checksum)。
主要使用来保证每次提交的完整性。

git log
![log](/images/log.png)

可以看到每个commit后都有一个加密的串。这个串可以做一些回滚或者checkout的操作。---下面的这些操作建议少使用。

*	git reset --soft (id) 撤销当前的提交，会保持当前的提交和之间的跨度提交，将代码回退到id对应的版本。
*	git reset --hard (id) 撤销当前的提交，将代码硬回退到id对应的版本。
*	git reset . 撤销当前本地的所有提交和修改。回退到你拉代码时的版本。可以重新get代码
*	git reset –soft HEAD 回退一个版本
*	git reset –soft HEAD~3 向前回退三个版本
*	git reset –hard origin/master 回退到服务器最新

有时候可能需要撤销当前的修改，做其他的任务的时候，操作方式分为三种。

*	直接提交当前的提交
*	暂存当前的提交
	*	git stash 暂存起来
	*	git stash pop 拿出暂存区的内容
        ![stash](/images/stash.png)
	*	git stash list 查看暂存区列表


*	直接提交抛弃到此次的修改
	*	git checkout . 也可以撤销某个文件的修改 .替换成修改的文件，需要带上路径

#### stage（暂存区）
![data](/images/data-flow.png)

操作步骤如下：

*	git add . 增加索引 提交到暂存区
*	git commit -m 'message'提交到本地仓库
*	git push origin (BranchName) 提交到远程仓库
*	git commit -a -m 'message'

上面三个步骤是将本地代码提交到远程服务器的基本操作。


#### git flow
git工作流程是以分支为基础的操作流程
![flow](/images/git-flow.png)
>可以看下项目里面现在的分支规则

这个地方需要发布代码的遵循这个规则。现在基本是有（徐蕾）负责。


#### 工作中git的使用
我一般遇到的问题和操作方式，感兴趣的也可以自己习惯一套自己的命令方式，不反对使用sourceTree和git tortoise等工具提交源码，原则是保证代码合并的时候不出问题。

*	建立仓库,推送代码,建议使用ssh的方式，现在git支持http和ssh两种方式。
	*	git init
	*	git add .
	*	git commit -m 'message'
	*	git remote add origin (git url)
	*	git push origin master
	*	git checkout -b develop
	*	git push origin develop
	*	git remote set-url origin (git url)
*	拉取代码，提交，推送代码
	*	git status
	*	git add .
	*	git commit -m 'message'
	*	git pull --rebase
	*	git push origin ()
*	遇到冲突的时候
	*	先解决掉本地的冲突，建议使用开发工具解决
	*	git rebase --continue | git merge -m 'message'
	*	git add .
	*	git push origin ()
*	stash（隐藏起来） 当你需要临时修改其他的bug，但是现在已经的修改又不能提交时，建议采用这个方式
	*	git stash
	*	git stash pop
	*	git stash list
*	建议多使用git的本地分支去辅助代码开发
*	建议每次提交时以一个单元去使用，意思就是一个独立的单元生成一个提交，在继续后面的开发。一次推送可以推送多个单元的提交
*	未完待续