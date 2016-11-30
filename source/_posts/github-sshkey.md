---
title: github配置ssh key
date: 2016-11-30 10:43:18
categories: Program
tags:
- github
- ssh key
---
### 配置ssh key，免密码提交代码至github  
0x01:在本机生成ssh所需的公钥和私钥  
```bash
user@user MINGW64 ~
$ cd ~

user@user MINGW64 ~
$ ssh-keygen -t rsa -C "hahaha9@gmail.com" -f ~/.ssh/PCWIN10（-f参数指定生成的密钥名称，也可以不指定，默认为id_rsa）
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):(这里需要输入密码，如果为空，提交的时候就不需要输入密码)
Enter same passphrase again:（重复刚刚的密码）
Your identification has been saved in PCWIN10.（生成的私钥）
Your public key has been saved in PCWIN10.pub.（生成的公钥，需要把这个文件中的内容提交到github上）
The key fingerprint is:
SHA256:IIhHNNc8fLREgPg1N+YEuQyUudsTHrtzY7f1yMMYjCM hahaha9@gmail.com
The key's randomart image is:
+---[RSA 2048]----+
| .+ooBo*+        |
| o.+= O.*.       |
|. o..=.X..       |
| .  o.=..        |
|     + +So       |
|    . E o o      |
|       + . +.    |
|      o + oooo   |
|       + o..o..  |
+----[SHA256]-----+

```
0x2:添加ssh key至github网站
![image](http://note.youdao.com/yws/public/resource/9b81f59da40d11de3c042aea1ef77e85/xmlnote/436FF14EB51744D78B225F06676F2C88/1356)

0x3:将私钥添加到本地gitbash  
```bash
$ ssh-agent bash --login -i
$ ssh-add ~/.ssh/PCWIN10(你自己的私钥的路径)

```

0x4:本地测试是否已经正常
```bash
$ ssh -T git@github.com
Hi xxxx! You've successfully authenticated, but GitHub does not provide shell access.

```
已经认证成功。

0x4:将https提交类型的remote设置为ssh类型  
查看当前本地仓库的远程仓库链接，发现是https类型的，提交代码的时候会要求输入github的帐号和密码，比较麻烦，我们将它改成ssh类型的，加上之前设置的ssh key，就可以实现免密码提交代码了。Let's do it！
```bash
$ git remote -v
origin  https://github.com/yourname/My_Blog.git (fetch)
origin  https://github.com/yourname/My_Blog.git (push)

```
更改远程仓库链接为ssh类型
```bash
$ git remote set-url origin git@github.com:你的github用户名/你的远程仓库名.git
```
再次查看远程仓库链接
```bash
$ git remote -v
origin  git@github.com:yourname/xxx.git (fetch)
origin  git@github.com:xxxx/xxx.git (push)

```
接下来就可以自由地不用输入帐号密码就可以push了，感觉爽歪歪。
```bash
$ eval `ssh-agent`
Agent pid 6720
$ ssh-add ~/.ssh/PCWIN10
Identity added: /c/Users/xxx/.ssh/PCWIN10 (/c/Users/xxxx/.ssh/PCWIN10)
$ git push

```
ps: ssh-agent
>ssh-agent是一种控制用来保存公钥身份验证所使用的私钥的程序，ssh-agent其实就是一个密钥管理器，运行ssh-agent以后，使用ssh-add将私钥交给ssh-agent保管，其他程序需要身份验证的时候可以将验证申请交给ssh-agent来完成整个认证过程。
