**.gitignore和exclude的区别**

    两个文件的本质都是用来忽略一些项目文件,用处不同而已
    
    .gitignore文件会被提交到仓库,一般这个文件内容忽略的东西也是别人想忽略的
    exclude文件不需要提交到仓库,忽略个人文件

**查看每行的提交记录**

    git blame file

**合并多个commit提交,让提交看起来干净一点**

```
//不包括commitId这个提交
git rebase -i commitId
↓
↓
pick 382a2a5 add zoo //第1个commit
pick cd82f29 add cat1 //第2个commit
pick 1de2076 add cat2 //第3个commit
pick 3da3212 add cat3 //第4个commit
pick 2bab3e7 add dog1 //第5个commit
pick 27f6ed6 add dog2 //第6个commit
↓
↓
//cat3和cat2合并到cat1
//dog2合并到dog1
pick 382a2a5 add zoo
pick cd82f29 add cat1
squash 1de2076 add cat2
squash 3da3212 add cat3
pick 2bab3e7 add dog1
squash 27f6ed6 add dog2
↓
↓
接下来进行两次msg的提交
↓
↓
提交远程,强制推送
git push -f
```

**.gitignore规则**

    *.class //忽略所有class
    !HelloWorld.class //HelloWorld.class不忽略
    /*.iml //只会忽略根目录的iml文件
    aaa/ //忽略任意aaa目录
    aaa //忽略任意aaa目录和文件

