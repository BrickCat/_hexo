---
title: axios post提交参数java后台接收不到参数解决办法
date: 2017-11-08 13:37:31
tags: [axios,javascript,常见问题]
categories: javascript
---

## 问题描述
在浏览器中使用`axios`POST提交数据在后台接收的时候接收不到。
<!-- more -->
## 解决办法

1. 首先我们先来了解一下什么是`axios`
    Axios 是一个基于 promise 的 HTTP 库，可以用在浏览器和 node.js 中。
2. 问题原因
    由于axios默认发送数据时，数据格式是Request Payload，而并非我们常用的Form Data格式，所以Java后台获取不到数据。在发送之前需要对数据进行处理。
3.  解决办法
```javascript
    var params = new URLSearchParams();
    params.append('firstName',Fred);
    
    axios.post('/user', params)
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
```
