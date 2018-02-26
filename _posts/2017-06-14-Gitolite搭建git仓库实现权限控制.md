---
layout: post
title: "Gitolite搭建git仓库实现权限控制"
categories: linux
description: 
keywords: linux,git
---

## 创建用户

 > 创建一个`git`用户，用来运行`gitolite`服务。如果你之前是用git账号来做权限控制的话，记得把`/etc/passwd`里git用户的shell换回`/bin/bash`。

```sh
# git server
adduser git
```

>工作机生成一个rsa密匙,并将公匙拷贝至git服务器下

```sh
# 工作机
ssh-keygen
scp /root/.ssh/id_rsa.pub root@cp01-matrix-sandbox-03.epc.baidu.com:/home/git/admin.pub
```



# 安装gitolite

> 切换到git用户安装gitolite，指定之前要root账户下生成的git_admin.pub密匙
>
> ~/repositories/目录生成两个repository：gitolite-admin.git和testing.git ，gitolite-admin.git是用于管理gitolite配置与权限，testing.git.git是测试库。
>
> 用户目录下生成一个project.list，默认里面有一个testing的git的project

```sh
# git server
su git
cd ~

git clone git://github.com/sitaramc/gitolite

mkdir -p $HOME/bin
gitolite/install -to /home/git/bin/

/home/git/bin/gitolite setup -pk /home/git/admin.pub
```

```sh
[git@cp01-mawmd-rd02 ~]$ ll repositories/
total 8
drwx------ 8 git git 4096 Jun 14 15:41 gitolite-admin.git
drwx------ 7 git git 4096 Jun 14 15:41 testing.git
```



## 配置权限

> 检出gitolite-admin仓库（这里是在本地服务器上直接做检出,输入之前的 git账户密码即可）

```sh
# 工作机
git clone git@cp01-matrix-sandbox-03.epc.baidu.com:gitolite-admin
```

> conf是放置配置文件的目录 - gitolite.conf就是gitolite的配置文件，包含对用户、仓储、仓储权限的配置 - keydir目录用来放置所有的用户公钥 - admin.pub为安装时setup -pk的那个用户公钥。
```sh
[root@cp01-mawmd-rd02 test]# tree
.
└── gitolite-admin
    ├── conf
    │   └── gitolite.conf
    └── keydir
        └── admin.pub

3 directories, 2 files
```



## 创建新 git库并添加授权用户

> 检出gitolite-admin代码

```sh
git clone git@cp01-matrix-sandbox-03.epc.baidu.com:gitolite-admin
cd gitolite-admin
```

> 创建一个xiaocai的git仓库，@all是所有用户

```sh
vim conf/gitolite.conf
----------------------------
repo gitolite-admin
    RW+     =   admin

repo testing
    RW+     =   @all

repo xiaocai
    RW+     =   @all
```

> 编辑完之后用git进行提交

```sh
[root@cp01-mawmd-rd02 gitolite-admin]# git add .
[root@cp01-mawmd-rd02 gitolite-admin]# git commit -m "新增库"
[master a074c50] 新增库
 1 files changed, 3 insertions(+), 0 deletions(-)
[root@cp01-mawmd-rd02 gitolite-admin]# git push origin
Counting objects: 7, done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (4/4), 388 bytes, done.
Total 4 (delta 0), reused 0 (delta 0)
remote: Initialized empty Git repository in /home/git/repositories/xiaocai.git/
To git@cp01-matrix-sandbox-03.epc.baidu.com:gitolite-admin
   737b13e..a074c50  master -> master
```

> 这时候进入git服务器查看，可以看到名为xiaocai.git的裸库已经创建好了(也可以在服务器上先执行 `git init --bare xiaocai.git`)(gitolite删除裸库则需要在服务器上操作)

```sh
[git@cp01-matrix-sandbox-03 ~]$ cd /home/git/repositories
[git@cp01-matrix-sandbox-03 repositories]$ ll
total 12
drwx------ 8 git git 4096 Jun 14 17:19 gitolite-admin.git
drwx------ 7 git git 4096 Jun 14 17:19 testing.git
drwx------ 7 git git 4096 Jun 14 17:19 xiaocai.git
```

> 用户权限控制,创建RD和QA用户组

```sh
@group_qa @user1 @user2 @user3
@group_rd @user4 @user5

repo gitolite-admin
    RW+     =   admin

repo testing
    RW+     =   @all

repo xiaocai
    R       =   @group_qa
    RW+     =   @group_rd user6
```

> 接着要在keydir目录下分别对应用户的rsa公匙

```sh
└── gitolite-admin
    ├── conf
    │   └── gitolite.conf
    └── keydir
        └── admin.pub
        └── user1.pub
        └── user2.pub
        └── user3.pub
        └── user4.pub
        └── user5.pub
        └── user6.pub
```



## 操作符

C

> 代表创建。仅在 通配符版本库 授权时可以使用。用于指定谁可以创建和通配符匹配的版本库。

R, RW, 和 RW+

> R 为只读。RW 为读写权限。RW+ 含义为除了具有读写外，还可以对 rewind 的提交强制 PUSH。

RWC, RW+C

> 只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等）。

RWD, RW+D

> 只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。

RWCD, RW+CD

> 只有当授权指令中定义了正则引用（正则表达式定义的分支、里程碑等），才可以使用该授权指令。其中 C 的含义是允许创建和正则引用匹配的引用（分支或里程碑等），D 的含义是允许删除和正则引用匹配的引用（分支或里程碑等）。

\- （是的，这是一个减号，你没看错）

> 是一条禁用指令。只对写操作起作用，即禁用用户的写操作。



## 参考资料

- https://github.com/sitaramc/gitolite
- http://airk000.github.io/git/2013/09/25/gitby-gitolite.html