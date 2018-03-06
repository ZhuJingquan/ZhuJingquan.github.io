---
layout:     post
title:      "In puts your name here"
subtitle:   " \" just follow the standard way\""
date:       2018-03-06 10:02:52
author:     "朱景泉"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 工作相关
typora-copy-images-to: ../img
---

> “Yeah It's on. ”

---
typora-copy-images-to: Images
---

#### BAIDU Apollo 环境搭建出坑记：

Q1: Apollo Kernel 与 Apollo-platform 是否必要

官方解释最好安装Linux4.4 kernel

```bash
sudo apt-get install linux-generic-lts-xenial
```



官方答复：[Normal ubuntu kernel is ok](https://github.com/ApolloAuto/apollo-platform/issues/26)

![anwer from contributer](/program/Notes/Images/1514267104434.png)

- [Apollo Kernel](https://github.com/ApolloAuto/apollo-kernel#what-is-the-difference)

> ### What is the difference
>
> - Realtime patch (<https://rt.wiki.kernel.org/index.php/RT_PREEMPT_HOWTO>)
> - Latest e1000e intel ethernet driver
> - Bugfix for Nvidia driver under realtime patch
> - Double free in the inet_csk_clone_lock function patch (<https://bugzilla.redhat.com/show_bug.cgi?id=1450972>)
> - Other cve security patches
>
> [Kernel config files](https://github.com/ApolloAuto/apollo-kernel/tree/master/linux/configs) are modified for Apollo based on Ubuntu's config-4.4.0-X-generic.
>
> The Apollo team would like to thank everybody in the open source community. The GitHub apollo-kernel/linux is based on Linux. Currently, Apollo team maintains this repository. In near future, we’ll send patches back to Linux community.

**Linux 内核轻易不要动**，从14.04 的官方内核升级至Apollo Kernel 后，会存在很多硬件驱动不兼容的问题。

- [Apollo-Platform](https://github.com/ApolloAuto/apollo-platform)

> ### What is the difference
>
> Compared to original ROS, we made the following improvements to enhance its stability and performance:
>
> - ROS Decentralization Feature
> - High Efficient Communication based on Shared Memory Transport Feature
> - Native Support with Protobuf Feature
>
> For more details of each feature, please find in the [Design Docs](https://github.com/ApolloAuto/apollo-platform/blob/master/ros/docs/design).
>
> The Apollo team would like to thank everybody in the open source community. The GitHub apollo-platform/ros is based on [ROS](https://github.com/ros/ros) and [Fast-RTPS](https://github.com/eProsima/Fast-RTPS). Currently, Apollo team maintains this repository. In near future, we’ll send patches back to the corresponding communities.

具体Apollo对ROS哪一部分有更改，可以[Diff后查看结果](https://github.com/ApolloAuto/apollo-platform/issues/9) ，更改主要发生在共享内存功能的引入

> node节点添加共享内存修改的文件，新增了一部分文件，修改了一部分文件，仅修改了roscpp/libros rosbag中的文件
>
> ## new add files
>
> ```
> src/ros_comm/roscpp/include/ros/config_comm.h
> src/ros_comm/roscpp/include/sharedmem_transport/rossharedmem_block.h
> src/ros_comm/roscpp/include/sharedmem_transport/sharedmem_publisher_impl.h
> src/ros_comm/roscpp/include/sharedmem_transport/sharedmem_segment.h
> src/ros_comm/roscpp/include/sharedmem_transport/sharedmem_util.h
>
> src/libros/sharedmem_transport/sharedmem_block.cpp
> src/libros/sharedmem_transport/sharedmem_publisher_impl.cpp
> src/libros/sharedmem_transport/sharedmem_segment.cpp
> src/libros/sharedmem_transport/sharedmem_util.cpp
>
> transport_mode.yaml
>
> ```
>
> ## modify files
>
> ```
> src/ros_comm/roscpp/include/ros/publisher.h
> src/ros_comm/roscpp/include/ros/shm_manager.h
> src/ros_comm/roscpp/include/ros/subscribe_options.h
> src/ros_comm/roscpp/include/ros/subscription_callback_helper.h
> src/ros_comm/roscpp/include/ros/subscription_queue.h
> src/ros_comm/roscpp/include/ros/topic_manager.h
>
>
>
> src/ros_comm/roscpp/src/libros/init.cpp
> src/ros_comm/roscpp/src/libros/publication.cpp
> src/ros_comm/roscpp/src/libros/publisher.cpp
> src/ros_comm/roscpp/src/libros/shm_manager.cpp
> src/ros_comm/roscpp/src/libros/subscription_queue.cpp
> src/ros_comm/roscpp/src/libros/topic_manager.cpp
>
> src/ros_comm/rosbag/src/recorder.cpp
>
>
> ```

技术细节检索关键子“[High Efficient Communication based on Shared Memory Transport Feature](https://github.com/ApolloAuto/apollo-platform/blob/master/ros/docs/design/shm_transport.md)”



Q2：doc里面的how to 文件夹需要认真查看： docker image 如何save？

​	[How to save and load docker image](/program/BaiduAuto/apollo/docs/howto/how_to_save_load_docker_image_locally.md)

​	[How to Run Perception Module on Your Local Computer](/program/BaiduAuto/apollo/docs/howto/how_to_run_perception_module_on_your_local_computer.md)



Q3：





---

[目前](https://github.com/ApolloAuto/apollo-platform/blob/master/ros/docs/design/shm_transport.md)关注百度的源码，具体平台实现，交给系统架构师后续解决

---

Apollo 架构层面

[HOW TO UNDERSTAND ARCHITECTURE AND WORKFLOW](/program/BaiduAuto/apollo/docs/howto/how_to_understand_architecture_and_workflow.md)

#### Fundamentals to undersatnd aplloauto - core

主要包括：

- 如何基于ROS实现RPC

  > Since xmlRPC from ROS1 is really old \(compared
  > to the recent brpc and [grpc](https://yiakwy.github.io/blog/2017/10/01/gRPC-C-CORE)\), baidu develops its own protobuf version of RPC. 

  ​

- Core modules Integration

  > Location\Perception\Planning\Routine

- Hdmap 基于ScanMatching的relocation

- 基于docker的Online simulation

####  apolloauto modules structure

common模块

#### EKF is utilized in data preprocessing

> Since it is used in input data preprocessing, you might see it in hdmap, perception, planning and so on so forth.

#### Selected modules analysis

HMI  python的 Flask 架构（还好以前搞过dyjango）

Dreamviewer （React，Webpack,Threejs）

#### Perception

Initially, this module implemented logics exclusively for Lidar and Radar processes.

- Lidar（64线）:

> - Hadmap: get tranformation matrix convert point world coordinates to local cordinates and build map polygons
> - ROI filter: get ROI and perform Kalman Filter on input data
> - Segmentation: A U-Net based \(a lot of variants\) caffemodel will be loaded and perform forward computation based on data from Hdmap and ROI filtering results
> - Object Building: Lidar return points \(x, y, z\). Hence you need to goup them into "Obstacles" \(vector or set\)
> - Obstacles Tracker: Baidu using uisng HM sovler from Google(隐马尔可夫)（数据关联）. For large bipartite graph, **KM algorithms in Lagrange foramt is usually deployed since**
>   **SGD is extremely simple for that.**

- Radar

>ROI filter: get ROI objects and perform Kalman Filter on input data
>
>Objects Tracker

- Probability Fusion(new in 1.5)

> collects all the info and makes final combination of information from sensors on mother board 
> for track lists and rule based cognitive engine
>
> The major process is **association**, hence HM algorithms here is used again as a bipartite graph(二分图的最大匹配与完美匹配)
>
> **Track lists are maintained** along timestamps and each list will be updated based on probablistic rules engine

---

### How to tune control parameters

#### Background

> ### Input / Output
>
> #### Input
>
> - Planning trajectory
> - Current car status
> - HMI driving mode change request
> - Monitor System
>
> #### Output
>
> The output control command governs functions such as steering, throttle, and brake in the `canbus`.

#### Controller Introduction

#### Lateral controler

**LQR-Based** Optimal Controller(和规划组的讨论)The dynamic model of this controller is a simple bicycle model with side slip. It is divided into two categories, including a closed loop and an open loop.

#### Longitudinal Controller

The longitudinal controller is configured as a **cascaded PID (级联PID)+ Calibration table.** It is divided into two categories, including a closed loop and an open loop.

速度环与位置环PID

Calibration table 提供一个加速度与油门、刹车之间的对照关系。

Useful tools 提供了调参过程。

[规划组比较有用的材料](/program/BaiduAuto/apollo/docs/howto/how_to_tune_control_parameters.md)

