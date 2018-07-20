---
title: Git使用指南
date: 2018-07-18 17:46:38
tags: Git
categories: Git
thumbnail: http://img3.imgtn.bdimg.com/it/u=1117912982,3551249993&fm=27&gp=0.jpg
---
# Git使用指南  
> 介绍git常用的部分。

## 1 常用概念

### 1.1 config
刚开始使用git的时候，都会需要配置git变量，指令如下：  

$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com。

gitconfig文件用于配置或读取相应的工作环境变量。这些变量可以存储在三个地方：  

/etc/gitconfig 文件：系统中对所有用户都普遍适用的配置，对应git config 时的 --system 选项  
~/.gitconfig 文件：用户目录下的配置文件只适用于该用户，对应git config 时的 --global 选项   
当前项目中的.git/config文件，配置仅仅针对当前项目有效

每一个级别的配置都会覆盖上层的相同配置，一般来说只需要自己配置user.name和user.email，通过git config --list指令来查看配置信息。


### 1.2 ssh-key  
ssh是一种加密的网络传输协议，作用是保证数据传输的安全。

创建SSH key：
$ ssh-keygen -t rsa -C "your_email@example.com"  
代码参数含义：  

-t 指定密钥类型，默认是 rsa ，可以省略。  
-C 设置注释文字，比如邮箱。  
-f 指定密钥文件存储文件名，一般省略，使用默认名称。

"公钥登录"原理：用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录，不再要求密码。  


### 1.3 工作区和暂存区
电脑中能看到的目录就叫是工作区，而工作区中隐藏了一个.git目录，这个不算工作区，而叫Git的版本库。  
版本库中有很多东西，最重要的包括暂存区、分支和始终指向当前分支的HEAD指针。

![暂存区](/images/stage.png)

工作区修改的文件需要添加到暂存区，然后由暂存区提交到分支上。主要有以下几条常用指令：  

1. git add "filename" : 将指定改动的文件添加到暂存区；
2. git add ./--all : 将所有改动的文件添加到暂存区；
3. git status : 查看暂存区状态；
4. git commit -m "commit message" : 将暂存区的内容提交到分支上。


### 1.4 .git版本库

![.git](/images/git.png)

**hooks文件夹** 用于存储shell脚本，当执行某些git指令后，会触发存储在该文件夹下指定的shell脚本 

**info文件夹** 用于存储该项目仓库的相关信息

**logs文件夹** 用于记录分支提交记录

**objects文件夹** 用于存放每次commit时的数据

**refs文件夹** 用于记录每个分支的最新提交结点，fetch的远程分支和tags

**COMMIT_EDITMSG** 保存最新的commit message，Git系统不会用到这个文件，只是给用户一个参考

**config** GIt仓库的配置文件

**description** 仓库的描述信息，主要给git托管系统使用

**HEAD** 映射到refs的引用，引用中包含了一个分支（branch），通过这个文件Git可以得到下一次commit的parent

**description** 仓库的描述信息，主要给git托管系统使用

**index**  暂存区

**FETCH_HEAD** 记录fetch = +refs/heads/*:refs/remotes/origin/*


## 2 常见问题  

### 2.1 冲突（CONFLICT）
当前分支和merge的指定分支存在不一样的地方时，会产生冲突。git会自动解决部分冲突，但是仍然有部分冲突需要依靠手动解决。
使用VSCode能够清晰的看到冲突的地方，根据提示选择保留某分支的修改，或者合并修改。

解决冲突之后需要add并commit，但是涉及到团队合作时，一定要确认commit后程序能够正常运行，再push到远程仓库。  
经常pull远程仓库，并在修改自己的代码后及时push能够有效避免冲突。 

### 2.2 追踪关系（tracking）

git pull的时候可能出现如下提示：
There is no tracking information for the current branch.

#### 2.2.1 问题重现：

某天我在commit完代码后，使用git pull拉了一波代码，然后切换到daily/0.0.1分支的时候，发现代码并没有拉下来，并且使用git pull指令无效。虽然最后使用 git pull origin daily/0.0.1 指令成功pull了代码，但是为什么我有的分支能够直接git pull，有的要加上origin <远程分支名> 呢？

![](/images/1.pullfail.png)
切换到daily/0.0.1时，git pull提示当前分支没有追踪信息，不能够直接pull。

![](/images/2.pullsuccess.png)
切换到lps/0.0.1后，使用git pull却成功了。

#### 2.2.2 概念解释

以下摘自http://www.ruanyifeng.com/blog/2014/06/git_remote.html

```
git pull <远程主机名> <远程分支名>:<本地分支名>
如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

在某些场合，Git会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。
比如，在git clone的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的master分支自动"追踪"origin/master分支。

Git也允许手动建立追踪关系。


git branch --set-upstream master origin/next
上面命令指定master分支追踪origin/next分支。

如果当前分支与远程分支存在追踪关系，git pull就可以省略远程分支名。


$ git pull origin
上面命令表示，本地的当前分支自动与对应的origin主机"追踪分支"（remote-tracking branch）进行合并。

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

$ git pull
上面命令表示，当前分支自动与唯一一个追踪分支进行合并。

```

#### 2.2.3 错误原因及解决
错误的原因是我在已经存在远程分支的情况下，新建了一个与远程分支同名的本地分支，但是并没有将它们建立追踪关系，所以不能直接git pull。

![](/images/4.erroroption.png)

使用git pull origin <远程分支名> 或者指定分支追踪即可解决问题。

### 2.3 覆盖

新的版本可以覆盖旧的版本。

#### 2.3.1 覆盖原理

假设远程分支的版本号为 1， 2， 3；  
甲拉取了远程分支，修改文件后commit并push到了远程分支，此时远程分支的版本号为1， 2， 3， 4-A；
乙也拉取了远程分支，修改文件并commit后想push到远程分支，但发现远程分支已经有新版本了，需要pull远程分支，并解决冲突之后才能够push，此时乙的本地分支版本号为1， 2， 3， 4-B；  
乙执行pull操作后，工作区发生变化，乙通过解决冲突，add并commit后生成了新的版本，版本号为1， 2， 3， （4-A， 4-B） ,5-B;  
4-A是甲进行修改后的分支，在pull远程分支，解决冲突的时候，乙对甲的文件作出的所有的操作都会保留。

#### 2.3.2 解决办法

如果乙分支的5-B版本覆盖了甲的内容，使用版本回退能够将分支版本退回到合并前，然后再次进行合并。  
如果乙在修改完自己的分支后没有进行commit，那么乙如果覆盖了甲，则只能回退到修改前的版本。所以修改完后一定要先commit再pull。

## 3 Sourcetree

### 3.1 变基
git rebase 和git merge 做的事其实是一样的。它们都被设计来将一个分支的更改并入另一个分支，只不过方式有些不同。

我们的项目有很多分叉，是因为不同的版本号merge之后分岔为了两条线，最后通过merge合并在了一起。（git pull实际上是fetch+merge，每一次pull都进行了一次merge操作）

而使用git rebase的结果和merge一样，但是可以减少soucetree的分岔。

### 3.2 操作指令
sourcetree上的指令与直接使用git bash指令的最大区别在于多了一行：  
```
git -c diff.mnemonicprefix=false 
-c core.quotepath=false 
-c credential.helper=sourcetree

```
该行的作用如下：
```
-c <name>=<value> 传递配置参数，给定的值将覆盖配置文件中的值。 
-c: 代表config缩写

diff.mnemonicprefix:
该指令用于配置git diff操作

credential.helper : 证书助手
Credentials helpers是一种外置的程序代码，Git可以从那里取得用户名和密码。  
credentials helpers通常和操作系统或者其他程序提供的安全存储交互。
git config --list | grep credential 查看凭证助手

core.quotepath
在使用git的时候，经常会碰到有一些中文文件名或者路径被转义成\xx\xx\xx之类的，此时可以通过git的配置来改变默认转义。
```

**提交操作**
git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree commit -q -F /var/folders/ks/pdrvgq4s5zv96h_cyt07ltwh0000gn/T/SourceTreeTemp.ANOrLw -a 

这条命令将commit的信息改变为了字符串的形式
等同于
git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree commit -m "第四行" -a

git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree push -v --tags --set-upstream origin refs/heads/jyc/0.0.1:refs/heads/jyc/0.0.1 

tags：标签

**推送操作**
git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree push -v --tags origin refs/heads/jyc/0.0.1:refs/heads/jyc/0.0.1 

与提交操作中的push区别是，这次没有 --set-upstream

**拉取操作**
git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree fetch origin 

git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree pull origin jyc/0.0.1 

**创建和删除分支**
git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree branch newbranch2 

git -c diff.mnemonicprefix=false -c core.quotepath=false -c credential.helper=sourcetree branch -d newbranch

## 总结

使用Git的过程中总能出现各种错误，但是它们都是有迹可循的。我将Git的错误分为两类，一类是Git管理类的错误，比如错误的merge，覆盖等，这种错误会导致开发项目不正确，但不会阻碍git的正常运行。这类错误需要熟悉git的分支管理与合并。另一种错误是Git维护类错误，比如git的logs或者refs出错，或者config的配置不正确，需要用户熟悉git本身的管理机制。