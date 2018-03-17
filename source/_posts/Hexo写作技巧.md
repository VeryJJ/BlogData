---
title: Hexo写作技巧
categories: 工具
tags: [Hexo]
date: 2018-03-17 21:17:08
---

Hexo 写作的技巧与备忘，不喜勿喷。

<!-- more -->

+ 链接：
    - [Hexo命令](https://hexo.io/docs/commands.html)
    
## 文章插入图片

#### 原生Markdown语法
原生Markdown语法插入图片有三种方式

##### 1. 插入本地图片  
只需要在基础语法的括号中填入图片的位置路径即可，支持绝对路径和相对路径。 
例如： 

```markdown
![image](/home/picture/1.png)
```

评价：不灵活不好分享，本地图片的路径更改或丢失都会造成markdown文件调不出图。

##### 2. 插入网络图片  
只需要在基础语法的括号中填入图片的网络链接即可，现在已经有很多免费/收费图床和方便传图的小工具可选。 
例如： 

```markdown
![image](http://baidu.com/pic/doge.png)
```

评价：将图片存在网络服务器上，非常依赖网络和网络图片存储

##### 3. 把图片存入markdown文件
用base64转码工具把图片转成一段字符串，然后把字符串填到基础格式中链接的那个位置。 
基础用法： 

```markdown
![avatar](data:image/png;base64,iVBORw0......) 
```

这个时候会发现插入的这一长串字符串会把整个文章分割开，非常影响编写文章时的体验。如果能够把大段的base64字符串放在文章末尾，然后在文章中通过一个id来调用，文章就不会被分割的这么乱了。 
比如： 

```markdown
![avatar][doge] 
[doge]:data:image/png;base64,iVBORw0...... 
```

评价：麻烦，费劲。


#### Hexo方式
##### 安装插件与配置
1. 把主页配置文件_config.yml 里的post_asset_folder:这个选项设置为true
1. 在你的hexo目录下执行这样一句话npm install hexo-asset-image --save，这是下载安装一个可以上传本地图片的插件
1. 等待一小段时间后，再运行hexo n "xxxx"来生成md博文时，/source/_posts文件夹内除了xxxx.md文件还有一个同名的文件夹

##### 使用方式
在xxxx.md中想引入图片时，先把图片复制到xxxx这个文件夹中，然后只需要在xxxx.md中按照markdown的格式引入图片

```markdown
![你想输入的替代文字](xxxx/图片名.jpg)
```

注意： xxxx是这个md文件的名字，也是同名文件夹的名字。只需要有文件夹名字即可，不需要有什么绝对路径。你想引入的图片就只需要放入xxxx这个文件夹内就好了，很像引用相对路径。





