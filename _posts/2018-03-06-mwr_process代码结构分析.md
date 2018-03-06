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

---
typora-copy-images-to: mwr_process_代码结构分析
---

# nwr_process代码结构分析

## 代码架构（与 Lane_Fusion 类似无太大变化）

Lane_fusion 中的fuser类分别被esr，rsds，srr替换。

clcmsubscriber has-a clcmhandler;

clcmhander has-a esr,rsds,srr

esr，rsds，srr中分别has-a trackmanager类，trackmanager类has-a tracker

## （大的处理流程）处理流程：

Clcmhandler

CallbackxxxMsg ()//including docoding msg into objects,trans form into vehicle coordinates.

FilterxxxMsg（const CANET_MWR_ESR21PC& msg,const TRANSFORM& transform,const VCU_VEHICLE_INFO vcu）

#### （filter 的work flow）包括：

- 从CAN下取出数据：

```cpp
DecodeXXXCanMsg(const CANNET_MWR_ESR21PC& msg);
{
  取出数据至 MWR_MEASURED_OBJECT;//这一部分不需要动
   MWR_MEASURED_OBJECT 转换至MWR_RAW_OBJECT;
    m_vecRAWXXXObjs.pushback(MWR_RAW_OBJECT);
}
```

​	MWR_MEASURED_OBJECT  (Measured 即意味着 objects in polar coordinates)

```cpp
//defined in header.h
enum MWR_TYPE
{
  MT_FRONT_ESR,
  MT_FRONT_LEFT_RSDS,
  MT_FRONT_RIGHT_RSDS,
  MT_REAR_LEFT_SRR,
  MT_REAR_RIGHT_SRR,
  MT_REAR_ESR
};


struct MWR_MEASURED_OBJECT
{
  int64_t nTimeStamp;//ms
  int8_t	nSensorType;
  bool bIsObjmoving;//CFI_WORLD??
  int8_t nID;// Valid if(nID!=0x00)?
  float fTrackPower;//unit::dB
  float fRange;//unit:m;
  float fRangeRate;//unit:m/s;
  float fAngle;//unit:rad
  float fWidth;
  bool bIsNoisy;//true:noisy object
  
  MWR_MEASURED_OBJECT()
  {
    bIsObjMoving=true;
  }
};
```

- MWR_RAW_OBJECT  解析后的MWR_MEASURED_OBJECT(这是上面定义的结构体，因为是中间数据，所以没有使用lcm 生成msg头文件)转换到RAW

  ```cpp
  int8_t nID;
  int8_t bMoving;
  float fTrackPower;
  float fX;
  float fY;
  float fVelX;
  float fVelY;
  float fWidth;
  //保留Polar 坐标系的数据？
  float fTheta;//朝向？
  float fRange;
  float fRangeRate;
  ```

  ​

- remove noise

  ```cpp
  for()

  {

  	if(noise)

  	{

  		m_vecRawFrontEsrObjs.erase(m_vecRawFrontEsrObjs.begin()+i)		//STL 的iterator

  	}

  }

  ```

  ​

- using EKF to track and manege objects,

  ```
  MWR_RAW_OBJECTS FilteredObjs;
  m_stXXXTrackManager.TrackManageObjects(m_vecRawxxxObjs,FilteredObjs);
  ```

  ​

- transform the valid object into vehicle coordinate ,publish to lcm

  TRANSFORM 里面就是一堆数组，进行简单的坐标变换，（借助Eigen写一个简单点的调用接口）；
  filter 完后的Objects 的header 依然沿用原来的msg 的header；

```

```



## Filter算法实现（基于EKF）

```cpp
\\tracker.h
#include <MWR_RAW_O>
```





