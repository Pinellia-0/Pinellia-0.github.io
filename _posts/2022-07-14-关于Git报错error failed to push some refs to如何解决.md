---
title: 关于Git报错error: failed to push some refs to...如何解决
tags: 计算机相关问题
---



在使用Git推送过程中出现错误

## 问题发现

今天我和往常一样通过Git做博客推送，就在输完`git push`按下回车后直接报错，我真的，直接流汗黄豆啊。

报错如下：

```bash
半夏倾城@DESKTOP -R3RQKU4 MINGW64 /c/b1og/ posts (master)
$ git push
To github.com:Pinellia-0/Pinellia-0.github.io.git
！[rejected]		master -> master (fetch first)
error: failed to push some refs to 'github.com:Pinellia-0/Pinellia-0.github.io.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes 
hint: (e.g., 'git push ...')before pushing again.
hint: See the Note about fast-forwards' in 'git push --help' for detai ls.
```

**<font color=Red>其大致意思是指：本地和远程的文件应该合并以后才允许上传本地的新文件！</font>**



## 常见错误

报错的基本内容都是  `error:failed to push some refsto ‘远程仓库地址’`。

## 问题分析

通过网上查阅资料和`Git`给出的提示不难看看出问题的所在原因：在昨晚我回寝室后打开自己的博客网站发现排版出了点问题，由于源文档在实验室的电脑里保存着，所以我用了寝室的笔记本直接在 `GitHub`上的云库里进行了代码的修改，这样导致了我本地文件和远程库的文件出现了不统一；

我们在关联本地和远程时，两端都是有内容的，但这两份内容没有联系起来，当我们推送到远程或者从远程拉回内容时，都会查看有没有被跟踪的内容，于是你看到 `Git`报的详细错误中总是会让你先拉取再推送，但是拉取出现了失败。

## 解决方法

#### 方法一：

对于 `error: failed to push some refsto'远程仓库地址'`使用以下命令即可

```bash
git pull --rebase origin master
```

然后再进行上传：

```bash
git push -u origin master
```

推送成功。

#### 方法二：

直接输入：

```bash
git pull origin master
```

然后在上传

```bash
git push -u origin master
```

推送成功。

（此方法我没尝试，据说可以。。。。。。）

---

---

## 补充一下

这里我说明一下`git pull origin master`与`git pull --rebase origin master`的区别 

区别：

```
git pull=git fetch + git merge
git pull --rebase=git fetch+git rebase
```

git fetch : 从远程分支拉取代码,可以得到远程分支上最新的代码。

所以git pull origin master与git pull --rebase origin master的区别主要是在远程与本地代码的合并上面了。



现在有两个分支：test和master，假设远端的master的代码已经更改了（在B基础上变动：C,E），test的代码更改了要提交代码（在B基础上变动：D,E），如下所示：

```
      D---E test
      /
 A---B---C---F--- master
```



问题就来了，如果C,F和D,E的更改发生冲突，那么就需要我们合并冲突了，下面我们来看看git merge和git rebase怎么合并的

git merge:

```
       D--------E
      /          \
 A---B---C---F----G---   test, master
```

git rebase

```
A---B---D---E---C‘---F‘---   test, master
```

对比可看出：git merge多出了一个新的节点G，会将远端master的代码和test本地的代码在这个G节点合并，之前的提交会分开去显示。

git --rebase会将两个分支融合成一个线性的提交，不会形成新的节点。

 

```
rebase好处
想要更好的提交树，使用rebase操作会更好一点。
这样可以线性的看到每一次提交，并且没有增加提交节点。
merge 操作遇到冲突的时候，当前merge不能继续进行下去。手动修改冲突内容后，add 修改，commit 就可以了。
而rebase 操作的话，会中断rebase,同时会提示去解决冲突。
解决冲突后,将修改add后执行git rebase –continue继续操作，或者git rebase –skip忽略冲突。
```
