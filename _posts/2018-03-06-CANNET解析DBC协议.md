---
layout:     post
title:      "In puts your name here"
subtitle:   " \" just follow the standard way\""
date:       2018-03-06 10:15:15
author:     "朱景泉"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 工作相关
typora-copy-images-to: ../img
---

> “Yeah It's on. ”

打开DBC文件后：

C++ 位运算方法：



- 查找对应的Msg List ,CANNET 会将包头包尾去掉；
- 对应每个msg，查看列表与Layout；
- 对每一个signal ，进行分段解析，位运算>>左右移位，注意查看信号的单位与是否是有符号数；