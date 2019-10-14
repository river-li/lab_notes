---
title: Example
mathjax: true
author: river-li
date: 2019-10-14 11:54:23
categories: 
- Paper
tags:
---

# Example

一级标题设置为论文的标题

该部分简要说明论文内容，研究的相关领域以及工作价值

## Abstract

论文摘要部分的内容，简要介绍论文做了哪些工作，如何做的

注意查看这里的源代码，在摘要结束后需要在这里添加一行截断：

<!--more--->

## Background

该领域的相关研究，如果有一些比较有价值的可以在这里附加相关文章的引用，格式如下：

> MehranJodavi,MahdiAbadi,andElhamParhizkar.2015. JSObfusDetector:A binaryPSO-basedone-classclassifierensembletodetectobfuscatedJavaScript code.In 2015 The International Symposium on Artificial Intelligence and Signal Processing AISP.IEEE,Mashhad,Iran,322–327. https://doi.org/10.1109/AISP. 2015.7123508

## Overview

简要介绍每一部分工作的内容

如果有论文的整体框架或系统流程图可以添加在这部分

插入图片时需要创建一个与markdown文件同名的文件夹，将图片放在文件夹中，插入图片时使用语句：

{% asset_img 1.png 1.png %}

## Markdown 语法简要介绍

这一部分的标题设置为核心的工作，依照论文中的标题即可

### 小标题的用法

这部分需要分部分介绍时设置三级标题，每一级标题下具体展开工作如仍需分级则使用四级标题

#### 小标题说明

markdown中井号的数量代表标题的等级，数量越多标题级别越低，字号越小

在四级标题下如果需要出现分点的内容，建议使用有序列表或无序列表，使用方法如下

- 无序列表表项一
- 无序列表表项二

1. 有序列表表项一
2. 有序列表表项二

### 代码块

论文中出现代码片段时，需要使用代码块

如果是某个函数名在文章中需要被提到，则使用行内代码块即可，例如：`strcpy`函数可能会引起缓冲区溢出

如果是大段的代码则使用行间的代码块

```C
int main()
{
    printf("helloworld\n");
    return 0;
}
```

行间代码块第一行需要表明编程语言，以便系统语法高亮方便阅读

### 数学公式

在文章中出现数学公式时最好使用latex进行公式的编辑，行内出现的符号使用行内公式，$\alpha$, 符号周围一个`$`

行间需要居中的公式使用两个`$`

$$
Avg = \frac{\alpha_1+\alpha_2+...+\alpha_n}{n}
$$

**注意: 论文中用到数学公式的话必须在文件头部设置启用mathjax**
## 总结与评价

总结部分写文章的性能和效果，附加自己的思考和对其价值的评价

## 附录

如果有需要补充说明的内容添加在附录部分

### 提交论文笔记的方法

首先需要在github上fork这个笔记仓库

访问[https://github.com/river-li/lab_notes](https://github.com/river-li/lab_notes)，fork该项目到自己的账号 


clone 自己的项目到本地
```bash
git clone https://github.com/用户名/lab_notes.git
```

接着在本地仓库中添加上游的连接

```bash
git remote add https://github.com/river-li/lab_notes
```

```bash
git checkout -b sidelau-branch
```

在添加自己的博客前需要新建分支，分支命名为姓名-branch

之后添加需要的markdown文件及相关的图片文件

添加完文件后，提交到自己的仓库中

```bash
git add .
git commit -m "Paper from Author"
git push
```

提交到自己的仓库后在https://github.com/river-li/lab_notes发起pull request即可

之后管理员整合通过了pull request即可发布

如果以及clone过并提交过note的，需要在再次提交前保持上流的最新

```bash
git pull --rebase
```

相关的git用法可以自行学习

### markdown文件头部的说明

markdown文件必须头部请参照本文的头部，author替换为自己的昵称

tags以无序列表的形式写上文章所属的领域标签，例如：

- Deep Learning
- Attack Detection

等