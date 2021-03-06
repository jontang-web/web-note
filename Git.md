## 特点
- git是一个分布式版本控制系统；
- git管理的不是最新版本的文件快照，而是把代码仓库完整地镜像下来，每一次的克隆操作，实际上都是一次对代码仓库的完整备份；
- git仓库中保存的是文件的所有版本数据，git工作目录只是从git仓库的压缩数据库中提取出来的某个独立版本内容，放在磁盘上供你试用和修改，git暂存区域是一个文件，保存了下次将提交的文件列表信息，一般在 Git 仓库目录中；
- 对git而言，文件只有两种状态：已跟踪和未跟踪。添加到git管理的文件是已跟踪文件，未添加到git管理的文件是未跟踪文件，例如新创建的文件；
- 对用户而言，文件有以下几种状态：新文件，未修改，已修改和已放入暂存区。新文件：新创建的文件，未添加到git管理，git能检测到新文件，但无法跟踪新文件的修改。未修改：git跟踪的文件（曾经提交过）没有被修改。已修改：git跟踪的文件被修改了，但是没有添加到暂存区。已放入暂存区：git跟踪的文件被修改过或者新文件，并且通过add命令添加到了暂存区，待提交的文件。

## 文件生命周期

```
$ mkdir folder
##新建文件，添加文件内容

$ git add <filename>
##添加到git跟踪（staged暂存区）
##或者
$ git add .
##添加所有新建文件到git跟踪（staged暂存区）

$ git commit -am 注释
##提交修改文件到git管理

$ git pull [远程仓库名] [远程分支名]
##从远程分支拉取文件，默认origin仓库master分支

$ git push [-u] [远程仓库名] [远程分支名]
##-u 与远程分支建立连接并推送文件到远程分支，默认origin仓库master分支
```
## 常用命令
```
$ git config key value
##系统配置，如git config --global user.name xxx设置用户名
```
   

```
$ git config --list
##查看系统配置信息
```

```
$ git --help
##查看git试用手册
```

```
$ git init
##创建git仓库（获取git仓库方式一）
```

```
$ git clone url [别名]
##拷贝现存的git仓库（获取git仓库方式二）
```

```
$ git add <filename>
##  添加文件到git暂存区（添加git跟踪），文件名可以是一个具体的文件，也可以是通配符，如*.txt等，或者“.”添加所有新加的文件到git管理
```
  

```
$ git commit -m "注释"
## 提交暂存区中的文件，不在暂存区中的文件不会被提交，加--amend覆盖上一次的提交
```
```
$ git commit -am "注释"
##把git跟踪的文件添加到暂存区，并且提交 
```

```
$ git status -s
##查看文件状态，-s表示精简信息
```

```
$ git diff
##查看尚未暂存的文件更新内容，-cached 查看已暂存的文件更新内容
```

```
$ git rm <filename>
##从git中移除文件并从工作目录中删除文件。如果删除之前修改过并且已经放到暂存区域的话，则必须要用强制删除选项 -f（译注：即 force 的首字母）。如果文件从 Git 仓库中删除（亦即从暂存区域移除，移除git跟踪），但仍然希望保留在当前工作目录中，使用--cached选项
```

```
$ git mv <sourcename> <targetname>
##文件重命名，相当于mv README.md README git rm README.md git add README三条命令的效果
```

```
$ git log
##查看提交历史
```

- -p
按补丁格式显示每个更新之间的差异。
- --stat
显示每次更新的文件修改统计信息。
- --shortstat
只显示 --stat 中最后的行数修改添加移除统计。
- --name-only
仅在提交信息后显示已修改的文件清单。
- --name-status
显示新增、修改、删除的文件清单。
- --abbrev-commit
仅显示 SHA-1 的前几个字符，而非所有的 40 个字符。
- --relative-date
使用较短的相对时间显示（比如，“2 weeks ago”）。
- --graph
显示 ASCII 图形表示的分支合并历史。
- --pretty
使用其他格式显示历史提交信息。可用的选项包括 oneline，short，full，fuller 和 format（后跟指定格式）。

```
$ git reset HEAD <file>
##把文件从暂存区移出（取消文件暂存）
```

```
$ git checkout <branch-name>
##切换分支
```
## 远程仓库
远程仓库是指托管在因特网或其他网络中的你的项目的版本库。==默认的远程仓库名称是origin，分支是master==
##### 创建远程仓库
```
##新建目录
$ mkdir [目录名]
$ cd [目录名]

##初始化本地仓库
$ git init

##添加README
$ touch README.md
$ git add README.md

##提交
$ git commit -am "readme"

##添加远程仓库（本地仓库和远程仓库建立连接）
$ git remote add [origin:远程仓库名] [ssh://git@gitee.com:secuTang/test.git:远程仓库地址]

##推送代码到远程仓库
$ git push -u [origin:远程仓库名] [master:远程仓库分支]
```
##### 多个远程仓库
```
##添加多个远程仓库
$ git remote add [A:远程仓库名] [ssh://git@gitee.com:secuTang/A.git:远程仓库地址]
$ git remote add [B:远程仓库名] [ssh://git@gitee.com:secuTang/B.git:远程仓库地址]

##与多个远程仓库建立连接
$ git push -u 仓库A <branch-name>
##相当于
$ git branch --set-upstream-to=仓库A/<branch-name> <本地分支名>
$ git push 仓库A <branch-name>

##从远程仓库拉取代码
$ git pull 仓库A <branch-name>
```



##### 其他操作
```
##查看远程仓库更多信息
$ git remote show [remote-name]

##远程仓库重命名
$ git remote rename [old-name] [new-name]

##移除远程仓库
$ git remote rm [remote-name]
```
## 分支（branch）

```
##创建分支并切换到新创建的分支
$ git checkout -b dev origin/dev
##相当于
$ git branch dev
$ git checkout dev

##查看所有分支
$ git branch -a

##切换分支
$ git checkout <branch-name>

##合并分支到当前分支
$ git merge <branch-name>
##例如在master分支下执行命令，则合并<name>分支到master分支

$ git branch -d <branch-name>
##删除本地分支

$ git push <远程仓库> --delete <branch-name>
##删除远程分支
```
## 标签（tag）
Git 可以给仓库历史中的某一个提交打上标签，以示重要
##### 创建tag
1. 轻量标签

本质上是只将提交校验和存储到一个文件中，不保存任何其他信息。创建轻量标签不需要任何参数，如：
```
$ git tag v1.4
$ git show v1.4

//不包含额外信息
commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```
2. 附注标签

附注标签是存储在 Git 数据库中的一个完整对象，可以被校验，其中包含打标签者的名字、电子邮件地址、日期时间， 此外还有一个标签信息。通常会建议创建附注标签，如：
```
$ git tag -a v1.4 -m "my version 1.4"
```
3. 后期打标签

如果想要对已经提交的记录打标签，可以这样做：
```
$ git log --pretty=oneline
b1a1cfc0e3272e50d874e2a7d3da78d00e04afa1 (HEAD -> saas/1.3.0, origin/saas/1.3.0) format dateTime
359de061c66f1e62bc945bc15db0e490e97395dc Merge branch 'saas/1.3.0' of ssh://registry.code.tuya-inc.top:10023/TuyaFE-backendService/security into saas/1.3.0
b01c951e02c45b4b5d65023e3801a4fd53dafee1 format dateTime
cfd26f8e98238cd6e6113e426d7673394f75027b Merge remote-tracking branch 'origin/saas/1.3.0' into saas/1.3.0
afee81af8164d5cb8dc267287d36afeacfd76058 fix channel add reg
e4e672c7c60f440675960c441c70bba725092185 Merge branch 'saas/1.3.0' of ssh://registry.code.tuya-inc.top:10023/TuyaFE-backendService/security into saas/1.3.0
7541ab90e6415fecc801d8aeaf67ccef150c20ff fix

//假设要对“format dateTime”的提交打标签，找到提交对应的校验和
$ git tag -a v1.3.0 b1a1cfc0
```
4. 推送标签到远程仓库

默认情况下，git push 命令并不会传送标签到远程仓库服务器上，在创建完标签后你必须显式地推送标签到共享服务器上。如：
```
$ git push origin v1.5
//--tags把所有不在远程仓库服务器上的标签全部推送上去
$ git push origin --tags
```
##### 查看tag
```
$ git tag
//查看通配符匹配的所有标签，必须带上参数-l或者--list
$ git tag -l "v1.8.5*"
//查看标签信息
$ git show v1.3.0
```
##### 删除tag
1. 删除本地仓库标签
```
$ git tag -d <tagname>
```
2. 删除远程仓库标签
```
$ git push origin --delete <tagname>
```
##### 标签检出
查看某个标签所指向的文件版本，可以使用 git checkout 命令，如：
```
$ git checkout -b version2 v2.0.0
```
