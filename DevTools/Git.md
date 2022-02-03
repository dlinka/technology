##### 常用命令

```shell
#查看每行的提交记录
git blame file


git init
#工作区提交到暂存区
git add file
#暂存区提交到当前分支
git commit -m "XXX"
git status
git log --pretty=oneline
git reflog

#工作区和暂存区比较
git diff file
#工作区和版本库比较
git diff HEAD -- file

#回退到上一个版本
git reset --hard HEAD^
#回退到指定版本
git reset --hard XXX
#回退最后一次commit或者add的状态
git checkout -- file
#暂存区回退到工作区
git reset HEAD file

#创建分支
git branch name
#切换分支
git checkout name
#查看所有分支(*代表当前分支)
git branch
#删除分支
git branch -d name
#强行删除分支
git branch -D name

#合并指定分支到当前分支
git merge name
#合并指定分支到当前分支(合并会留下log)
git merge --no-ff -m "XXX" name

#显示远程仓库相关信息
git remote -v
#推送本地分支到远程分支
git push origin name
git pull
#设置远程分支和本地分支的链接
git branch --set-upstream-to=origin/name name
```

##### .gitignore

    .gitignore文件和exclude文件的区别是前者会被提交到仓库，文件中忽略的东西是所有人想忽略的，后者不会被提交到仓库，文件中忽略的东西是个人想忽略的
    
    .gitignore规则
    !HelloWorld.class //HelloWorld.class不忽略
    /*.iml 						//只会忽略根目录的iml文件
    aaa/ 							//忽略aaa目录
    aaa 							//忽略aaa目录和文件

#### rebase

```shell
p, pick = use commit
#使用这个提交,但是重新编辑message
r, reword = use commit, but edit the commit message
#合并到上一个提交
s, squash = use commit, but meld into previous commit
f, fixup = like “squash”, but discard this commit’s log message
#删除这个提交
d, drop = remove commit
```

##### 合并多个commit提交

```shell
#不包括commitId这个提交
git rebase -i commitId
↓
↓
#进入vi
pick 382a2a5 add zoo //第1个commit
pick cd82f29 add cat1 //第2个commit
pick 1de2076 add cat2 //第3个commit
pick 3da3212 add cat3 //第4个commit
pick 2bab3e7 add dog1 //第5个commit
pick 27f6ed6 add dog2 //第6个commit
↓
↓
#将cat3->cat2->cat1
#将dog2->dog1
#为什么不是dog1->dog2?那这样结果就是dog1
pick 382a2a5 add zoo
pick cd82f29 add cat1
squash 1de2076 add cat2
squash 3da3212 add cat3
pick 2bab3e7 add dog1
squash 27f6ed6 add dog2
↓
↓
#接下来进行两次msg的提交
↓
↓
#提交远程,强制推送
git push -f
```

****



