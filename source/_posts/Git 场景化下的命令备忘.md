---
title: Git 场景化下的命令备忘
date: 2018-03-15 16:09:21
categories: 工具 #文章分类
tags: [Git]
---

## 查看分支

```
        git branch
        git branch -a        查看远端所有分支
        git branch -v        查看本地分支，以及分支上最新的commit提交信息
        git branch -vv      在-v基础上，多现实本地分支和远程分支的关联关系
```

<!-- more -->

## 查看某次commit的修改

```
git show commit号
```

## 查看某个文件的历史修改记录

```
git log 文件名
git log -p 文件名
git log --author=提交人     只查看提交人的提交记录
git log --pretty=oneline 单行显示提交记录
git log --name-only    显示每次commit修改的文件列表
git log --name-status     查看commit记录里的文件修改状态
git log --grep='abc'    显示commit描述匹配abc的commit记录
git log -S "代码内容"   按代码内容搜索commit记录，如果代码内容部分想用正则表达式，则将-S换成-G
git log --pretty='%H %Cblue%cd %C(yellow)%cn %Cred%s' 按commit号+提交日期+提交人+commit标题 显示

pretty格式
%H   提交对象（commit）的完整哈希字串 
%h   提交对象的简短哈希字串 
%T   树对象（tree）的完整哈希字串 
%t   树对象的简短哈希字串 
%P   父对象（parent）的完整哈希字串 
%p   父对象的简短哈希字串 
%an  作者（author）的名字 
%_ae  作者的电子邮件地址 （由于新浪博客显示问题，请去除 %_ae 中的 _ ）
%_ad  作者修订日期（可以用 -date= 选项定制格式）（由于新浪博客显示问题，请去除 % ad 中的 _ ）
%ar  作者修订日期，按多久以前的方式显示 
%cn  提交者(committer)的名字 
%_ce 提交者的电子邮件地址（由于新浪博客显示问题，请去除 %_ce 中的 _ ）
%_cd  提交日期 （由于新浪博客显示问题，请去除 %_cd 中的 _ ）
%cr  提交日期，按多久以前的方式显示 
%d:  ref名称
%s:  提交的信息标题
%b:  提交的信息内容
%Cred: 切换到红色 
%Cgreen: 切换到绿色 
%_Cblue: 切换到蓝色 （由于新浪博客显示问题，请去除 %_Cblue 中的 _）
%Creset: 重设颜色 
%C(...): 制定颜色, as described in color.branch.* config option 
%n:  换行
```

## 查看未commit的本地修改

```
git diff 
git diff  文件名
```

## git diff 比较commit之间的差异

```
git diff commit                                      比较HEAD与commit之间的差异
git diff commit_1 commit_2                 比较两个commit之间的差异
git diff commit_1..commit_2                与git diff commit_1 commit_2 一样效果
```

## 从服务器拉代码

        git pull --rebase     (推荐)会把本地未push得commit放到缓冲区，然后把远程最新版本拉过来，再应用本地commit，这样不会造成本地有新commit时，merge的效果。
        git pull       直接更新，若本地和远端都有新commit，都执行自动merge。

## 拉分支

        git checkout -b branchName       创建本地新分支
        git checkout -b branchName remotes/origin/branchName  以远端分支创建本地新分支
        git push origin $newBranch:$newBranch             将本地分支提交到远端进行创建


## 删除远程分支


```
$ git push origin :master
# 等同于
$ git push origin --delete master
```

## 提交修改

        git add .            将修改加到stage状态区
        git commint -m "注释" 
        git push             推送所有分支
        git push origin develop       只推送develop分支


## 添加文件

        git add -A


## 删除文件

        git rm 文件名
        git rm -r 目录名       


## Push

    git push                                    push所有分支
    git push origin master              将本地主分支推到远程主分支
    git push –u origin master          将本地主分支推到远程（如无远程主分支则创建，用于初始化远程仓库）
    git push origin <local_branch>                                  创建远程分支，origin是远程仓库名。
    git push origin <local_branch>:<remote_branch>    创建远程分支

### 强制push

    如果远程主机的版本比本地版本更新，推送时Git会报错，要求先在本地做git pull合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用–force选项。        
    git push --force origin

## 合并分支
#### merge
        git merge remotes/origin/mc-s-3    将远端mc-s-3分支merge到本地 
#### rebase
        git rebase develop
        git rebase remotes/origin/develop
#### 配置mergetool
        git config –global merge.tool bc3
        git config –global mergetool.bc3.path 软件执行文件地址
#### merge策略
```
Git merge 策略的总结:

1、使用 -s 指定策略，使用 -X 指定策略的选项
2、默认策略是recursive
3、策略有 ours，但是没有theirs (Git老版本好像有)
4、策略ours直接 忽略 合并分支的任何内容，只做简单的合并，保留分支改动的存在
5、默认策略recursive有选项ours 和 theirs
6、-s recursive -X ours 和 -s ours 不同，后者如第3点提到直接忽略内容，但是前者会做合并，遇到冲突时以自己的改动为主
7、-s recursive -X theirs的对立面是 -s recursive -X ours

`注：-s recursive -X ours   合并分支，冲突时以本地为主`
```

## 回退未commit的修改
        git checkout [path] 将指定路径的修改还原到最新版本
## 回退已commit，未push的修改
        git reset HEAD <file> 
        --mixed 选项：默认的
        --soft 选项：改动会回退到stage状态
          --hard 选项：改动会直接丢失。

        git rebase -i 想要删除的commit的前一个commit号。
        出来的界面里，将想要删除的commit描述改为drop，保存即可。

## 回退已push的修改
        git revert 指定的commit号。跳出来的界面，选择要回退的commit内容（取消前面的#）   
        可以随便选某个commit删除  

        若revert一个merge的commit，则要指定parent 号
        git revert commit 号 -m 1。
                这样就选parent 1，那么parent 1又是哪一个呢？一般来说，如果你在master上mergezhc_branch,那么parent 1就是master，parent 2就是zhc_branch.

## 重排commit顺序
        git rebase -i commit号
        出来的界面中，将列出来的commit行重新排序再保存，就等于修改commit顺序了。
## 修改commit的描述
#### 未push
        方法一：
        git rebase -i commit号
        对应commit号前改为edit，保存。出来后git commit --amend。将commit描述修改掉，保存。
        出来后再git rebase --continue即可。

        方法二：
        git commit --amend    修改最近的一次commit

## 代码仓库迁移
        git clone --bare robbin_site robbin_site.git 
        git remote remove origin
        git remote add origin git@120.27.160.167:ZCY/doc-round-1.git
        git push –-all -–progress origin
## 导出指定版本的代码版本
        git archive -o ../updated.zip HEAD $(git diff --name-only HEAD^)
    例如：git archive -o ./version.zip 指定commit号
        或者  git archive --format zip -output "./archive.zip" HEAD
## tag功能
#### 创建tag
        git tag -a v1.0.0 -m '备注'
#### 查看tag
        git tag
#### 切换tag
        git checkout tag名
#### 删除tag
        git tag -d v1.0.0
#### 指定commit打tag
        git tag -a v1.0.0 commit号
#### 发布标签
        git push origin v1.0.0       将本地v1.0.0标签推送到git服务器
        git push origin -tags         将本地所有tag一次性推送到git服务器

## 创建补丁
#### 当前分支所有超前master的提交：
        git format-patch -M master
#### 某次提交以后的所有patch:
        git format-patch 4e16                --4e16指的是commit名
#### 从根到指定提交的所有patch:
        git format-patch                          --root 4e16
#### 某两次提交之间的所有patch:
        git format-patch 365a..4e16 -o <patch_dir>      --365a和4e16分别对应两次提交的名称
#### 某次提交（含）之前的几次提交：
        git format-patch –n 07fe             --n指patch数，07fe对应提交的名称
        故，单次提交即为：
        git format-patch -1 07fe

# 应用补丁
方法一（推荐）
```
1、在同一个仓库下找到对应的commit号
2、切换到对应分支下，git cherry-pick commit 号
3、如果冲突，git  mergetool 解决冲突。
4、git status根据提示commit代码，并push

cherry-pick 一个commit区间
git cherry-pick <start-commit-id>^..<end-commit-id>    start-commit-id是版本树里较早的commit

cherry-pick一个merge commit
git cherry-pick <commit-id> -m parent-number   -m代表 --mainline
实际例子：git cherry-pick 32b234 -m 1      1，2分别代表什么
```
# 查看未push到远程仓库的commit
## 1、查看到未传送到远程代码库的`提交次数`
```
    git status        //只能看次数

显示结果类似于这样：
# On branch master
# Your branch is ahead of 'origin/master' by 2 commits.
```

## 2、查看到未传送到远程代码库的`提交描述/说明`
```
git cherry -v

显示结果类似于这样：
+ b6568326134dc7d55073b289b07c4b3d64eff2e7 add default charset for table items_has_images
+ 4cba858e87752363bd1ee8309c0048beef076c60 move Savant3 class into www/includes/class/
```

## 3、查看到未传送到远程代码库的`提交详情`
```
git log master ^origin/master
这是一个git log命令的过滤，^origin/master可改成其它分支。
显示结果类似于这样：
commit 4cba858e87752363bd1ee8309c0048beef076c60
Author: Zam <zam@iaixue.com>
Date:   Fri Aug 9 16:14:30 2013 +0800

    move Savant3 class into www/includes/class/

commit b6568326134dc7d55073b289b07c4b3d64eff2e7
Author: Zam <zam@iaixue.com>
Date:   Fri Aug 9 16:02:09 2013 +0800

    add default charset for table items_has_images
```

## 查看两个分支的差异
#### 查看dev中有，而master中没有的
```
git log dev ^master

反之：查看master中有，dev中没有的
git log master ^dev
```

#### 查看dev中比master多了哪些提交（A比B多了哪些，就把A放..右边）
```
git log master..dev
```

#### 不在乎谁多谁少，只想看差异的提交
```
git log --left-right dev...master      #--left-right 会帮助显示差异的commit属于哪个分支
```


#### 整个目录比较差异详情
```
git difftool develop..pre-online --dir
```


## Git stash 暂存
```
git stash     
    将当前工作区里未commit的修改放到暂存区，将代码恢复到最近的一次修改

git stash list
    查看暂存区的列表

git show stash@{0} 
    see the last stash
    
git stash pop
    apply lastest stash and remove it from th list
    
git stash clear
    清空暂存栈

git stash apply stash@{1}
    指定暂存区里的某一次stash，应用到本地
```

## 删除本地git branch -a 能看到，而远程已经删掉的分支记录
```
    git fetch -p
```

## 更改时间显示方式
```
--date=(relative|local|default|iso|rfc|short|raw)
  Only takes effect for dates shown in human-readable format,
  such as when using "--pretty".  log.date config variable
  sets a default value for log command’s --date option.

--date=relative shows dates relative to the current time, e.g. "2 hours ago".

--date=local shows timestamps in user’s local timezone.

--date=iso (or --date=iso8601) shows timestamps in ISO 8601 format.

--date=rfc (or --date=rfc2822) shows timestamps in RFC 2822 format,
  often found in E-mail messages.

--date=short shows only date but not time, in YYYY-MM-DD format.

--date=raw shows the date in the internal raw git format %s %z format.

--date=default shows timestamps in the original timezone
  (either committer’s or author’s).


####格式化显示
例子：--date=format:'%Y-%m-%d %H:%M:%S'
参数：
%a        Abbreviated weekday name
%A        Full weekday name
%b        Abbreviated month name
%B        Full month name
%c        Date and time representation appropriate for locale
%d        Day of month as decimal number (01 – 31)
%H        Hour in 24-hour format (00 – 23)
%I        Hour in 12-hour format (01 – 12)
%j        Day of year as decimal number (001 – 366)
%m        Month as decimal number (01 – 12)
%M        Minute as decimal number (00 – 59)
%p        Current locale's A.M./P.M. indicator for 12-hour clock
%S        Second as decimal number (00 – 59)
%U        Week of year as decimal number, with Sunday as first day of week (00 – 53)
%w        Weekday as decimal number (0 – 6; Sunday is 0)
%W        Week of year as decimal number, with Monday as first day of week (00 – 53)
%x        Date representation for current locale
%X        Time representation for current locale
%y        Year without century, as decimal number (00 – 99)
%Y        Year with century, as decimal number
%z, %Z        Either the time-zone name or time zone abbreviation, depending on registry settings; no characters if time zone is unknown
%%        Percent sign
```

#### 全局更改方式
```
git config --global log.date relative
```

---

## 代码量
#### 统计当天提交的代码量
```
git log --author="$(git config --get user.name)" --no-merges --since=1am --stat
```

#### 统计报告-gitstats
+ 用GitStatX图形化工具查看

#### 统计报告-gitinspector
```
gitinspector --format=html --since=2018-01-01 --until=2018-12-30 --timeline --localize-output -w ./ > ~/tmp/gitinspector/zcy-payment-center-201801.html

```

gitinspector命令说明
```
➜  car-manage git:(master) gitinspector --help
用法：/usr/local/bin/gitinspector [选项]... [目录] 
在目录列出有关库的信息,如果没有指定目录，那么将使用现目录。如果有多个目录，
将采用指定的最后一个目录

长选项的强制性参数对短选项也适用
布尔参数只能给予长选项
  -f, --file-types=EXTENSIONS    一串逗号分隔的文件类型
                                   这些文件将会被用于计算统计数据.
                                   默认文件类型:
                                   java,c,cc,cpp,h,hh,hpp,py,glsl,rb,js,sql
  -F, --format=FORMAT            指定生成的输出文件的格式；
                                   默认格式是'text' 和
                                   可选格式:
                                   html,htmlembedded,text,xml
      --grading[=BOOL]           按照学生成评判项目的格式，
                                   显示统计数据和信息；
                                   等同于 -HlmrTw 选项
  -H, --hard[=BOOL]              记录行数并且寻找重复的内容;
                                   如果数据库较大，这个可能会需要一些时间
  -l, --list-file-types[=BOOL]   列出所有现在的数据库分支的文件格式
  -L, --localize-output[=BOOL]   在翻译版本存在的前提下，将输出结果翻译到系统语言
  -m  --metrics[=BOOL]           在分析提交时，检查特定指标
  -r  --responsibilities[=BOOL]  显示每位作者主要职责
      --since=DATE               只显示从特定时间起的结果
  -T, --timeline[=BOOL]          显示提交时间轴, 包括作者名称
      --until=DATE               只显示特定时间前的结果
  -w, --weeks[=BOOL]             按周来显示统计数据，而非月
  -x, --exclude=PATTERN          按特定格式排除不应该被统计
                                   的文件，作者名字或邮箱;可以按文件名，作者名，
                                   作者邮箱。可以重复
  -h, --help                     显示这个帮助信息并退出
      --version                  显示版本信息并退出

gitinspector 会过滤信息并且仅统计那些修改，增加或减少，指定文件类型的提交，
如需详细信息，请参考 -f 或 --file-types 选项

```







