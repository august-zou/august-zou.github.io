---
layout: post
title: 使用Markdown写文章
categories: [writing]
tags: [Markdown,语法，文字]
fullview: true
---

### 关于 Markdown
![markdown](/images/markdown.png)    
使用特定标点符号作为有意义和格式的标识符，通过Markdown渲染引擎，生成html文件，呈现有样式的文章。非常适合作为一种书写语言。

### Markdwon 语法

#### 段落
回车：行末尾两个或两个以上的空格

#### 标题

1. Setext形式是用底线的形式，利用=（第一标题）和-（第二标题），

2. Atx形式则是在行首插入1至6個#，对应到6阶标题：    
    # 这是 H1    
    ## 这是 H2    
    ###### 这是 H6

#### 清单
1. 无编号方式   
    *   Red
    *   Green
    *   Blue    
        
2. 有编号方式
    1.  Bird
    2.  McHale
    3.  Parish
    
#### 超链接
    This is [an example](http://example.com/ "Title") inline link.

    [This link](http://example.net/) has no title attribute.

#### 图片
    ![Alt text](/path/to/img.jpg)
