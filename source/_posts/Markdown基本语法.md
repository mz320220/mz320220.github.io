---
title: Markdown入门 & Typora编辑器
date: 2018-09-21 10:14:05
categories:
- Markdown 
---

> [从发现了一个超级好用的markdown编辑软件-Typora](#jumpTypora) <span id = "jumpTop">开始</span>

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。Markdown的语法简洁明了、学习容易，而且功能比纯文本更加强大，因此有很多人用它写博客。世界上最流行的博客平台[WordPress](https://baike.baidu.com/item/WordPress)和大型CMS如[Joomla](https://baike.baidu.com/item/Joomla)、[Drupal](https://baike.baidu.com/item/Drupal)都能很好的支持Markdown。

<!--more-->

# Markdown基本语法

##  1.块元素

### 1.1标题

markdown中使用1-6个`#`符号来表示不同层级的标题，例如：

```wiki
# 一级标题
## 二级标题
###### 六级标题
```

### 1.2 引用

markdown使用`>`符号来引用问题，多个`>`符号表示不同的引用的嵌套，例如：

```wiki
> 这是一个引用
> 
>> 这是嵌套的引用语句
```

### 1.3 列表

输入`*`可以表示一个无序列表，使用`+`或`-`可以实现同样的功能；

输入`1.`可以生成一个有序列表，例如：

```wiki
* 无序节点
* 无序节点
	- 子无序节点
	- 子无序节点

1. 有序节点
2. 有序节点
```

### 1.4 代办事项

代办事项可以通过`- [ ]`或`- [X]`分别表示未完成或已完成事项，书写的使用注意空格：

```wiki
- [ ] 未完成任务
- [X] 已完成任务
```

### 1.5 代码块

markdown使用\`\`\`来表示代码块，可以通过制定代码类型来显示不同的高亮风格，针对不同类型的代码只需要在\`\`\`后面加上java、python等描述即可。例如：

```java
private void print(){
    System.out.println("这是一个Java风格的代码块");
}
```

### 1.6 表格

输入`| firse header | second header |`可以生成一个两列的表格；
表格的对齐方式如下，详细代码如示例：

`:----` ：左对齐

`:----:`：居中对齐

`----:`：右对齐

`----`：默认为居中对齐

```wiki
| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ |:---------------:| -----:|
| col 3 is      | some wordy text | $1600 |
| col 2 is      | centered        |   $12 |
| zebra stripes | are neat        |    $1 |
```

以上代码在markdown中表现的表格为：

| Left-Aligned  | Center Aligned  | Right Aligned |
| :------------ | :-------------: | ------------: |
| col 3 is      | some wordy text |         $1600 |
| col 2 is      |    centered     |           $12 |
| zebra stripes |    are neat     |            $1 |

### 1.7 注脚

注脚通常用于在需要解释名词、书名、人名等时使用，markdown中注脚的表示如下：

```wiki
创建一个注脚测试[^footnote].
[^footnote]: 这是一个针对注脚的测试。
```
以上代码在markdown中的表现为：

创建一个注脚测试。[^1]

### 1.8 分割线

使用`***` 或 ``---`` `+++` 创建一条分割线

在使用这些符号是大于三个即可组成一条平行线，注意在使用`---`时前后空格，避免被当做题记的标记。

### 1.9 生成目录

输入`[toc]`即可在需要的位置生成一个目录，该目录自动抓取所有标题，且会自动更新。



## 2. 行内元素

### 2.1 链接

Markdown中提供两个形式的链接方式：行内链接和参考式链接，以行内链接语法做详细说明。

使用`[text](link "Optional title")` 表示行内链接，其中：

- `[text]`：text为需要添加链接的文字；
- `link`：link为链接的完整地址；
- `Optional title`：为链接显示标题，鼠标放在链接上时会弹出小框表示；

例1.行内链接：
```wiki
这是一个搜索引擎主页：[Baidu](www.baidu.com "百度一下，你就知道")
```

例2.参考式链接：

```wiki
这是一个搜索引擎主页：[Baidu][1]
[1]:www.baidu.com "百度一下，你就知道"
```

Markdown中样式：这是一个搜索引擎主页：[Baidu](https://www.baidu.com/ "百度一下，你就知道")

在编辑状态下，行内链接更加直观可以直接看到链接网址，但是不便于多次重复使用。参考式链接可以将链接统一放在文末进行处理，可以重复利用，但对应链接地址稍微麻烦一点。

### 2.2 URLs

Markdown语法中使用`<>`符号表示URLs。

```wiki
<www.baidu.com>
```

Markdown中的样式：<www.baidu.com>

### 2.3 图片

在Markdown中插入图片类似链接的语法，多了一个`!`

`![Alt text](/path/to/img.jpg "Optional title")`：导入图片，其中：

- `Alt text`：图片无法显示时显示的问题；
- `/path/to/img.jpg`：图片的路径；
- `Optional title`：图片的标题，鼠标放在图片上会显示小框提示；

```java
![GitHub](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-9-19/7391339.jpg "GitHub")
```

![GitHub](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/18-9-19/7391339.jpg "GitHub")

1. 图片路径可以使用绝对路径，也可以使用相对路径。
2. 通常可以在Markdown文档同级目录下建立picture文件夹，统一放置需要的图片，也可以在其中建立子文件夹针对不同图片进行分类管理。
3. 使用Hexo + GitHub搭建博客，为了减少GitHub空间占用，最好使用第三方图床工具进行图片管理。

### 2.4 斜体 & 加粗

使用`**`或`__`表示粗体；

使用`*`或`_`表示斜体；

```wiki
**粗体1**    __粗体2__
*斜体1*    _斜体2_
```

**粗体1**    __粗体2__
*斜体1*    _斜体2_

1. 在使用引用`>`符号时，会忽略`_`下划线的作用；
2. 推荐使用`*`或``**``进行文字强调工作；
3. 如果想要显示`*`等符号，需要使用转义字符`\`;

```wiki
\* : 在Markdown中显示一个 * 
```

### 2.5 代码

使用`` ` ``符表示行内代码。

```wiki
Use `printf()` function.
```

Use `printf()` function.

### 2.6 删除线 & 下划线

`~~`：表示删除线

`<u>`：原生的html标签实现下划线

```wiki
~~删除文本~~
<u>添加下划线</u>
```

~~删除文本~~
<u>添加下划线</u>

### 2.7 高亮

`==`：表示高亮文本，属于Markdown的扩展语法，不一定支持，请谨慎使用。

```wiki
==高亮文本==
```

==高亮文本==

### 2.8 Emoji

在Markdown中支持丰富的Emoji表情，通常格式为`:smile:`，冒号中为表情描述。

```markdown
:smile: :cry: :rose: :dog: :pig: :mouse: :telephone: :airplane:
```

:smile: :cry: :rose: :dog: :pig: :mouse: :telephone: :airplane:

## 3.HTML & 高级用法

在Markdown中可以使用html的标签等来实现Markdown本身不支持的一下属性，例如使用下面的语句展示一条红色文本。

```html
<span style="color:red">this text is red</span> 
```

<span style="color:red">this text is red</span> 

### 3.1 页内跳转

使用HTML代码实现页内跳转。

在待跳转的位置定义跳转指令：`[返回开头](#jumpTop)`

在要跳转到的目标位置定义锚：`<span id = "jumpTop">hehe</span>`

```html
[返回开头](#jumpTop)
<span id = "jumpTop">开始</span>
```

[返回开头](#jumpTop)

### 3.2 视频

使用`<video>`HTML的标签来添加视频，例如：

```html
<video src="xxx.mp4" />
```

### 3.3 其他HTML支持

其他HTML标签的支持，请参考文档：[官方文档](https://support.typora.io/HTML/)

### 3.4 流程图

流程图目前使用较少，参考文档：[Markdown中绘制流程图](https://support.typora.io/Draw-Diagrams-With-Markdown/)

### 3.5 数学公式

数学公式目前使用较少，参考文档：[Markdown中数学公式](https://support.typora.io/Math/)



# Typora工具

## 1. Typora工具介绍

<span id = "jumpTypora">Typora是一款轻便简洁的Markdown编辑器</span>，支持即时渲染技术，这也是与其他Markdown编辑器最显著的区别。即时渲染使得你写Markdown就想是写Word文档一样流畅自如，不像其他编辑器的有编辑栏和显示栏。

- 在Typora工具中，使用列表时可以通过Tab功能键增加子列表节点。

- 在Typora工具中，URLs会自动链接到目标网址。
- 在Typora工具中，可以图形化的创建、使用表格工具。

还有更换主题、Emoji自动带出、支持YAML Front Matter等等特性。

## 2. Typora常用快捷键

### 2.1 块元素操作

| 快捷键       | 作用      | 快捷键       | 作用     |
| ------------ | --------- | ------------ | -------- |
| Ctrl+1~6     | 1~6阶标题 | Ctrl+Shift+Q | 引用     |
| Ctrl+Shift+[ | 有序列表  | Ctrl+Shift+] | 无序列表 |
| Ctrl+Shift+K | 代码块    | Ctrl+Shift+M | 公式块   |
| Ctrl+T       | 创建表格  | Ctrl+Shift+Q | 引用     |

以上列大部分还没有直接使用Markdown语法来的方便，除了生成表格外：`Ctrl+T`

### 2.2 行内元素操作

| 快捷键 | 作用       | 快捷键       | 作用     |
| ------ | ---------- | ------------ | -------- |
| Ctrl+K | 创建超链接 | Ctrl+Shift+I | 插入图片 |
| Ctrl+B | 字体加粗   | Ctrl+I       | 字体倾斜 |
| Ctrl+U | 下划线     | Alt+Shift+5  | 删除线   |

以上即为学习Markdown并结合Typora工具的总结，可以满足绝大部分的使用需要了。



> 本文参考文章
>
> [Markdown Reference(Typora官方文档)](https://support.typora.io/Markdown-Reference/#span-elements)
>
> [Markdown，你只需要掌握这几个](http://www.cnblogs.com/crazyant007/p/4220066.html#fnref1)
>
> [编辑器：Typora](https://blog.csdn.net/qq_41648756/article/details/81057290#%E7%BC%96%E8%BE%91%E5%99%A8typora)

***





[^1]: 测试成功！