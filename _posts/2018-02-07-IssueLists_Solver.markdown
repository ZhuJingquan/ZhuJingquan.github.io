---
layout:     post
title:      "IssueLists&Solver"
subtitle:   " \" just record the way bug fixed""
date:       2018-02-07 13:13:00
author:     "朱景泉"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 工作相关
typora-copy-images-to: ../img
---



## 问题总结

#### [参考博客](http://blog.csdn.net/u014629875/article/details/53694656)很多问题虽然琐碎，但很影响工作的效率，所以把这些小事处理好，方法就是积累方法，类似问题能够举一反三。

- Linux 环境变量[设置](http://soft.chinabyte.com/os/169/11412169.shtml)，

　　PATH 决定了shell将到哪些目录中寻找命令或程序

　　HOME 当前用户主目录

　　HISTSIZE　历史记录数

　　LOGNAME 当前用户的登录名

　　HOSTNAME　指主机的名称

　　SHELL 　　当前用户Shell类型

　　LANGUGE 　语言相关的环境变量，多语言可以修改此环境变量

　　MAIL　　　当前用户的邮件存放目录

- Linux 运行库与编译[库设置指令](http://www.cnblogs.com/panfeng412/archive/2011/10/20/library_path-and-ld_library_path.html)

  开发时，设置LIBRARY_PATH，以便gcc能够找到编译时需要的动态链接库。

  发布时，设置LD_LIBRARY_PATH，以便程序加载运行时能够自动找到需要的动态链接库。

- Eigen 库比较特殊只有头文件，没有lib 文件。所以不需要TARGET_LINK_LIBRARIES,只需要[INCLUDE_DIRECTORIES](https://stackoverflow.com/questions/12249140/find-package-eigen3-for-cmake)

  ```cmake
  set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR})
  find_package(Eigen3 REQUIRED)
  include_directories(${EIGEN3_INCLUDE_DIR})
  ```

  ​

lcm 在decode 时会进行hash校验

```cpp
int MWR_RAW_OBJECTS::decode(const void *buf, int offset, int maxlen)
{
    int pos = 0, thislen;

    int64_t msg_hash;
    thislen = __int64_t_decode_array(buf, offset + pos, maxlen - pos, &msg_hash, 1);
    if (thislen < 0) return thislen; else pos += thislen;
    if (msg_hash != getHash()) return -1;// Hash校验

    thislen = this->_decodeNoHash(buf, offset + pos, maxlen - pos);
    if (thislen < 0) return thislen; else pos += thislen;

    return pos;
}
```



注意 出现 “error -1 decoding XXXX”时，找到msg文件，重新用

lcm-gen -x xxx.msg 生成新的xxx.hpp文件