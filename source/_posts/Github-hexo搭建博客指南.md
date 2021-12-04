---
title: Github+hexo搭建博客指南
date: 2018-03-19 20:12:05
categories: hexo建站
tags:
- Github
- hexo
---
![17.11.27-花莲](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-9-22/51716129.jpg)

博客搭好一段时间了，一直也没有写点东西，目前网上已经有很完整的Github + Hexo搭建博客的教程，本文仅列出基本指令和搭建过程中参考的指南以作备忘。

<!--more-->

# 常用指令
## Front-matter
即文章开头预设可设置参数，具体如下：

| 参数 | 描述 | 默认值 |
| - | - | - |
| layout | 布局|  |
| title | 标题 |  |
| date | 建立日期	 | 文件建立日期 |
| updated | 更新日期		 | 文件更新日期 |
| comments | 开启文章的评论功能	 | true |
| tags | 标签（不适用于分页）	 |  |
| categories | 分类（不适用于分页） |  |
| permalink | 覆盖文章网址 |  | |

## 发表博文
```
hexo n "postTitle" == hexo new "我的博客" //新建文章(默认为post模式)
hexo p == hexo publish  //草稿发布(draft类型)
hexo g == hexo generate //生成
hexo d == hexo deploy   //部署
hexo d -g == hexo deploy --generate //生成 + 部署
```
## 服务器
```
hexo s == hexo server   //本地启动服务预览，hexo会监听文件变更无需重启
hexo server -s          //静态模式
hexo server -p 5000     //更改端口
hexo server -i 192.168.1.1  //自定义 IP
```
在写完博客以后，可以先本地验证一下：`hexo s`

发现问题修改保存后，静态内容可以直接生效，否则需要重新生成启动服务：`hexo s -g`

验证完毕后即可推送到线上：`hexo d -g`

# 框架具体搭建过程

> 参考文档<br>

[hexo官方文档](https://hexo.io/zh-cn/docs/front-matter.html)<br>
[利用 hexo + Gitpage 开发自己的博客](http://cherryblog.site/Use-Gitpagehexo-to-develop-their-own-blog.html)
