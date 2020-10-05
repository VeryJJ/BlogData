---
title: 服务API设计之——API版本规范
tags: [API, API版本]
date: 2018-09-30 17:32:07
categories: 技术
---



# API版本规范

## 发布RELEASE版本

正式发布的api包必须是RELEASE版本

eg.

```
<dependency>
    <groupId>cn.gov.zcy.paas.template</groupId>
    <artifactId>template-api</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
```

## 版本号风格

使用 《[Semantic Versioning](https://semver.org/)》风格



<!--more-->



### 概述

Version号由 "MAJOR.MINOR.PATCH" 三段组合构成，version号增加含义：

1. MAJOR version：【主版本号】代表API发生了不兼容的变更，即使是微小的不兼容。
2. MINOR version：【次版本号】代表以兼容的方式新增了功能、特性
3. PATCH version：【补丁版本号】代表以兼容的方式做了bugfix



### 用法 / FAQ

### 版本号以0开始

- X.Y.Z 三个版本号都是以0开始。
- 【特别注意】当版本号是 "1.0.9.RELEASE"时，它的下一个补丁版本号是"1.0.10.RELEASE"  ！！！ 
  - 而不是"1.1.0.RELEASE"，这里不存在满十进位之说。



### 初始 MAJOR version

- 初始MAJOR version以0开始，代表业务的初始开发阶段，这过程中功能上任何改变都可能发生，此时的API是不稳定的。
- 初始版本一旦发布生产环境，即将MAJOR version变更为1，即 1.0.0.RELEASE。是第一个基线版本。



### 预发布版本

- 可以通过在补丁版本之后紧跟附加连字符和一系列点分隔标识符来表示预发布版本。标识符必须仅包含ASCII字母数字和连字符[0-9A-Za-z-]。标识符不能为空。数字标识符不得包含前导零。
- 预发布版本的优先级低于关联的普通版本。
- 预发布版本表示版本不稳定，可能无法满足其关联的正常版本所表示的预期兼容性要求。示例：1.0.0-alpha，1.0.0-alpha.1,1.0.0-0.3.7,1.0.0-x.7.z.92





