---
layout: page      # 必须
title: pageTitle  # 必须，页面名称
description: 用户自定义页面功能演示       # 页面二级标题，描述性文字
comments: false     # 禁用评论，可选，默认开启
reward: false       # 禁用打赏，可选，默认开启
---

当blockquote、img、pre、figure为第一级内容时，在page布局中拥有card阴影，所有标题居中展示

目前的想法是预定义一系列内容模块，通过像输入 Markdown 标记一样来简单调用。好在 Markdown 没有把所有便于输入的符号占用，最终我定义了@moduleName{ ... }这种标记格式。如果你使用过Asp.Net MVC，一定会很熟悉这种用法，没错，就是razor。

page布局中的title和subtitle对应 Markdown 中的title和description。

基本的内容容器还是card，你可以这样使用card：

@card{

在`page`页中，建议把内容都放到`card`中。

}

需要注意的是：标记与内容之间必须空一行隔开。至于为何要这样，看到最后就明白了。

与card标记类似，分栏的标记是这样的：

@column-2{

@card{

# 左

}

@card{

# 右

}

}

column中的每一列具有等宽、等高的特点，最多支持三栏：

@column-3{

@card{

左

}

@card{

中

}

@card{

右

}

}

在timeline模块中，你的 5 号标题#####和六号标题######将被“征用”，用作时间线上的标记点：

@timeline{

##### 2016

@item{

###### 11月6日

第一行 

第二行 /* ok */

}

@item{

###### 11月6日

第一行

第二行 /* error */

}

}
