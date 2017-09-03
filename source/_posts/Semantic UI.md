---
title: 安装 Semantic UI
date: 2017-09-02 23:43:05
tags: Semantic UI
categories: UI
---

## Semantic UI
　　Semantic作为一款开发框架，帮助开发者使用对人类友好的HTML语言构建优雅的响应式布局。它还提供了一套很方便的定制主题的方法，你可以用自己的想法去改变界面组件的样式。在这个教程里我们学习一下安装 Semantic UI 。

## 准备工具
　　我们使用shell去安装semantic UI，前提是你已经安装好了`nmp` `gulp`。
## 开始安装
- 创建一个目录
```Shell
mkdir semantic
cd semantic

```
- 用npm安装semantic UI
```shell
npm install semantic-ui

```
 一路回车就OK了。
- 查看是否安装成功
```shell
├── node_modules
├── package-lock.json
├── semantic
└── semantic.json
```
这是安装之后的文件夹和文件列表。


## 编译
- 进入到semantic的目录里**（是安装完之后的semantic的目录）** 然后执行 gulp 命令。
```shell
　cd semantic
　gulp bulid
```
- 查看编译后的文件
编译完之后会在文件夹中生成`dist`文件夹。
```shell
dist		gulpfile.js	src		tasks
```
- 现在到`dist`文件夹里看看都生成了什么东吧~
```shell
components		semantic.js		semantic.min.js
semantic.css		semantic.min.css	themes
```

　　components 目录下面是单独的一些组件，如果你只想使用 Semantic UI 里的某些组件，可以在这个目录下面找到这些组件。如果你想使用全部的组件，可以使用 semantic.css 与 semantic.js ，或者使用它们的最小化之后的版本，semanitc.min.css 与 semantic.min.js 。
　　themes 目录下是主题的样式，现在主要有四种主题:
```shell
basic		default		github		material
```
修改主题只需要进入到`src`文件夹中修改**theme.config**文件即可。例如把**default**换成**github**。




