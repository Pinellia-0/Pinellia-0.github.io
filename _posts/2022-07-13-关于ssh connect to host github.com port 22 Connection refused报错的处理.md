---
title: 关于ssh connect to host github.com port 22 Connection refused报错的处理
tags: 随笔

---

摩西摩西~

`GitHub` 对大家来说一定不陌生，无论是学习还是`交`（爬）`朋`（项）`友`（目）。

## 问题发现

昨日搭建个人博客的时候发现了这个问题，在我使用Git命令来操作GitHub验证密钥时提示报错，错误如下：`ssh: connect to host github.com port 22: Connection refused` 。

```bash
$ ssh -T git@github.com
ssh: connect to host github.com port 22: Connection refused
```

这让我这个计算机菜鸟头可疼了，本人的专业是电子信息工程，在计算机网络和软件方面的知识属实是才疏学浅，只能去网上查阅资料和问问我那计科的朋友。

## 排查思路

`ssh: connect to host github.com port 22: Connection refused`这个错误提示的是连接指`github.com`的22端口被拒绝了。

原本我认为的是<u>github.com</u>挂了，但我的浏览器能正常访问<u>github.com</u>。

在网上查阅众多资料，发现很多人都遇到了这种报错，我总结了2种原因和对应的解决方式：

### 1、使用GitHub的443端口

我的22端口可能是设定错误或者被防火墙给噶了，可以选择尝试链接GitHub的443端口。（我用的就是这个方法）。

~~~bash
$ vim ~/.ssh/config
```
# Add section below to it
Host github.com
  Hostname ssh.github.com
  Port 443
```
$ ssh -T git@github.com
Hi xxxxx! You've successfully authenticated, but GitHub does not
provide shell access.
~~~

具体操作步骤：在`~/.ssh/config`文件下添加以下内容

```bash
Host github.com
  Hostname ssh.github.com
  Port 443
```

这样ssh链接GitHub时就使用的443端口。

如果 `~/.ssh`目录下没有config文件，直接新建一个即可（直接随便新建一个文件改后缀就行）。

修改完config文件后，使用 `ssh -T git@github.com`来测试和GitHub的网络通信是否正常，如果提示 `Hi xxxxx! You've successfully authenticated, but GitHub does not provide shell access.`就表示一切正常。

但是，这个方案在有些人行不通，**这个方案有效的前提是**：执行命令`ssh -T -p 443 git@ssh.github.com`后不再提示`connection refused`，所以要尝试这个方案的小伙伴先执行这条命令测试下。

### 2、使用https协议，不使用ssh协议

在你的GitHub的本地repo目录，执行如下命令：

```bash
$ git config --local -e
```

然后把里面的url配置项从git格式

```bash
url = git@github.com:username/repo.git
```

修改为https格式

```bash
url = https://github.com/username/repo.git
```

这个其实修改的是repo根目录下的`./git/config`文件。

----

---

### 如果还不能解决的话,试试这个吧

**（此方法摘抄于[无忌 - 知乎 (zhihu.com)](https://www.zhihu.com/people/thucuhkwuji)）**

既然和GitHub建立ssh连接的时候提示`connection refused`，那我们就详细看看建立ssh连接的过程中发生了什么，可以使用`ssh -v`命令，`-v`表示verbose，会打出详细日志。

```bash
$ ssh -vT git@github.com
OpenSSH_9.0p1, OpenSSL 1.1.1o  3 May 2022
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: Connecting to github.com [::1] port 22.
debug1: connect to address ::1 port 22: Connection refused
debug1: Connecting to github.com [127.0.0.1] port 22.
debug1: connect to address 127.0.0.1 port 22: Connection refused
ssh: connect to host github.com port 22: Connection refused
```

从上面的信息马上就发现了诡异的地方，连接<u>[http://github.com](http://github.com)</u>的地址居然是`::1`和`127.0.0.1`。前者是IPV6的localhost地址，后者是IPV4的localhost地址。

到这里问题就很明确了，是DNS解析出问题了，导致[http://github.com](http://github.com)域名被解析成了localhost的ip地址，就自然连不上GitHub了。

Windows下执行`ipconfig /flushdns` 清楚DNS缓存后也没用，最后修改hosts文件，增加一条github.com的域名映射搞定。

```bash
140.82.113.4 github.com
```

查找[http://github.com](http://github.com)的ip地址可以使用[https://www.ipaddress.com/](https://www.ipaddress.com/)来查询，也可以使用`nslookup`命令

```bash
nslookup github.com 8.8.8.8
```

`nslookup`是域名解析工具，`8.8.8.8`是Google的DNS服务器地址。直接使用

```bash
nslookup github.com
```

就会使用本机已经设置好的DNS服务器进行域名解析，`ipconfig /all`可以查看本机DNS服务器地址。

这个问题其实就是DNS解析被污染了，有2种可能：

- DNS解析被运营商劫持了
- 使用了科学上网工具

按照上面写的解决方案操作即可解决。
