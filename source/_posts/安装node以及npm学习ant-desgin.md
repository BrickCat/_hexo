---
layout: 小记-mac
title: 安装node以及npm学习ant-desgin
date: 2017-09-20 09:20:18
tags: [react,ant-desgin,dva,npm,node]
categories: [react,node]
---

## 学习ant-desgin遇到的一些问题

　　在Mac上用brew安装的node.js,在使用npm时有些包是下载不下来的，导致项目编译失败。解决办法就是去官网下载LTS版本的node.js。

- 1、首先先把Mac上的npm下载的包删干净，然后在执行 `brew uninstall node` [参考](https://segmentfault.com/a/1190000007445643)
- 2、下载node.js 网址：[https://nodejs.org/en/](https://nodejs.org/en/) <font style="color:red;">注意：下载LTS版本</font>
- 3、进入到你项目的目录里，再次执行 `npm i`