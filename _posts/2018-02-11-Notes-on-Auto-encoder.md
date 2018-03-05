---
layout:     post
title:      "Auto-encoder"
subtitle:   " \" just follow the standard way\""
date:       2018-02-11 16:57:21
author:     "朱景泉"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - deep-Learning
typora-copy-images-to: ../img
---

> “Yeah It's on. ”

>- 无监督学习相当于用新向量表示原向量
>  - 重构误差最小  流形假设

模型：数学公式

算法：解决模型的工程方法

神经元个数少时sigmad效果比relu好

训练误差：在测试序列上的测试误差

结构误差：类似病态函数，同一测试集，加入误差后对扰动的敏感度

AE的训练过程，训练误差小的同时，降低结构误差

措施：加入正则约束，对激活值进行约束

Gnerative Adversarial Nets （GAN）生成对抗网络

判别模型 ：产生分类

生成模型：产生分布

