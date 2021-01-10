---
title: 解决homebrew长时间停在Updating Homebrew 这个步骤
tags: [homebrew]
date: 2020-12-19 22:33:11
categories: 工具
---

在国内的网络环境下使用 Homebrew 安装软件的过程中可能会长时间卡在 Updating Homebrew 这个步骤。

例：执行 brew install composer 命令
```shell
➜  ~ brew install composer
Updating Homebrew... # 如果碰到长时间卡在这里，参考以下 2 种处理方法
```

<!--more-->

 <br>

## 方法 1：按住 control + c 取消本次更新操作
```shell
➜  ~ brew install composer
Updating Homebrew...
^C
按住 control + c 之后命令行会显示 ^C，就代表已经取消了 Updating Homebrew 操作

大概不到 1 秒钟之后就会去执行我们真正需要的安装操作了

➜  ~ brew install composer
Updating Homebrew...
^C==> Satisfying dependencies
==> Downloading https://getcomposer.org/download/1.7.2/composer.phar
...
这个方法是临时的、一次性的
```
 <br>
## 【推荐】方法 2：使用 Alibaba 的 Homebrew 镜像源进行加速
平时我们执行 brew 命令安装软件的时候，跟以下 3 个仓库地址有关：

- brew.git

- homebrew-core.git

- homebrew-bottles

通过以下操作将这 3 个仓库地址全部替换为 Alibaba 提供的地址
<br>

### 1. 替换 / 还原 brew.git 仓库地址
- 替换成阿里巴巴的 brew.git 仓库地址:
```shell
cd "$(brew --repo)"
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
```

<br>
- 还原为官方提供的 brew.git 仓库地址

```shell
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git
```
 <br>

### 2. 替换 / 还原 homebrew-core.git 仓库地址
- 替换成阿里巴巴的 homebrew-core.git 仓库地址:

```shell
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git
```

<br>

- 还原为官方提供的 homebrew-core.git 仓库地址

```shell
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```
<br>

### 3. 替换 / 还原 homebrew-bottles 访问地址
这个步骤跟你的 macOS 系统使用的 shell 版本有关系

所以，先来查看当前使用的 shell 版本
```shell
echo $SHELL
```
- 如果你的输出结果是 /bin/zsh，参考3.1的 zsh 终端操作方式
- 如果你的输出结果是 /bin/bash，参考3.2的 bash 终端操作方式

#### 3.1 zsh 终端操作方式
- 替换成阿里巴巴的 homebrew-bottles 访问地址:

```shell
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
```
<br>

- 还原为官方提供的 homebrew-bottles 访问地址

```shell
vi ~/.zshrc
然后，删除 HOMEBREW_BOTTLE_DOMAIN 这一行配置
source ~/.zshrc
```
<br>

#### 3.2 bash 终端操作方式
- 替换 homebrew-bottles 访问 URL:

```shell
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```
<br>

- 还原为官方提供的 homebrew-bottles 访问地址

```shell
vi ~/.bash_profile
然后，删除 HOMEBREW_BOTTLE_DOMAIN 这一行配置
source ~/.bash_profile
```