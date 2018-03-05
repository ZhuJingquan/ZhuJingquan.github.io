---
layout:     post
title:      "How to parse date from can"
subtitle:   " \" just follow the standard way""
date:       2017-02-05 13:13:00
author:     "朱景泉"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 工作相关
typora-copy-images-to: ../img
---

> “Yeah It's on. ”

## Prerequisite

dbc file ，the layout of the CAN ，the interface with other module



step1

define some facilitates such as  

- Hardware status initialize

```cpp
const bool CMobileye::InitilizeMobileye()
{
    return (this->InitilizeLCM() && this->InitilizeCANA() && this->InitilizeCANB());
} //this should be done at the time of define CMobileye,or when you decied to use the CMobileye object(recommended)
#include <QApplication>

#include <global.hpp>
#include "cmobileye.h"
#include "clcmsubscribe.h"
#include <boost/thread.hpp>

int main(int argc,char** argv)
{
    QApplication a(argc,argv);
    CMobileye g_stMobileye;
    if(!g_stMobileye.InitilizeMobileye())
    {
        return -1;
    }

    CLCMSubscriber stLCMSubscriber;
    if(!stLCMSubscriber.InitializeLCM(g_stMobileye))
    {
        exit (EXIT_FAILURE);
    }

    boost::thread_group threadGroup;
    threadGroup.create_thread(boost::bind(&CLCMSubscriber::run, &stLCMSubscriber));
    threadGroup.create_thread(boost::bind(&CMobileye::CanARecvThread,&g_stMobileye));
//    threadGroup.create_thread(boost::bind(&CMobileye::CanBRecvThread,&g_stMobileye));


//    g_stMobileye.StartThread();
//    return 0;
    a.exec();
    threadGroup.join_all();
    return (EXIT_SUCCESS);
}


// for InitializeLCM and InitializeCANA and InitializeCANB just reference to the standard Way.

//also for initialize CAN status ,you'd better define your CANSetting Struct.
struct CAN_SETTING
{
    DEV_CAN_HANDLE canSensor_A;
    DEV_CAN_HANDLE canSensor_B;
    DEV_CAN_CFG canCfg;

    CAN_SETTING()
    {
        canSensor_A = DEV_HANDLE_NULL;
        canSensor_B = DEV_HANDLE_NULL;
        canCfg.cfgPX2.ids.clear();
        canCfg.cfgPX2.masks.clear();
    }
};

const bool CMobileye::InitilizeCANA()
{
    DEV_CAN_ERROR errorValue_A = {DEV_CAN_ERROR_NONE, DW_SUCCESS};
    m_stCanSetting.canSensor_A = BSP_CAN_open(DEV_CAN6, &errorValue_A);
    if(errorValue_A.errorType != DEV_CAN_ERROR_NONE)
    {
        PrintErrorInfo("Open CAN A communication error",__FILE__,__FUNCTION__,__LINE__);
        return false;
    }
    errorValue_A = BSP_CAN_start(m_stCanSetting.canSensor_A, m_stCanSetting.canCfg);
    if(errorValue_A.errorType != DEV_CAN_ERROR_NONE)
    {
        PrintErrorInfo("Start CAN A communication error",__FILE__,__FUNCTION__,__LINE__);
        return false;
    }
    return true;
}
```



- print function for debug

```cpp
void CMobileye::PrintCanDataFrame(const DEV_CAN_MSG& frame) const
{
    fprintf ( stderr, "=====================================================\n" );
    fprintf ( stderr, "Frame ID: %08X\n", frame.id);

    for ( int i = 0; i < 8; i++ )
    {
        fprintf ( stderr, " %02X", frame.arryData[i]);
    }
    fprintf ( stderr, "\n" );
    fprintf ( stderr, "=====================================================\n" );
}
```

- some datastruct for easy use in latter process

  - all datastruct defined by yourself should build depend on the meta struct DEV_CAN_MSG

    ```cpp
    // note: if plantform element is unequal zero, using it init the send msg
    typedef struct _DEV_CAN_MSG_
    {
    	#ifdef PLANTFORM_IPC
    		CAN_DataFram msgIPC;
    	#endif
    	#ifdef PLANTFORM_PX2
    		dwCANMessage msgPX2;
    	#endif
    	unsigned int id;
    	unsigned char len;
    	unsigned char arryData[BSP_CAN_MSG_MAX_LEN];
    }DEV_CAN_MSG, * P_DEV_CAN_MSG;
    ```

  - each DEV_CAN_MSG must have a related bool flag to check if this part of data has been successfully received.

```cpp
struct ME_CAN_OBJECT
{
    bool gbDataValid[3];
    DEV_CAN_MSG gstDataFrame[3];

    bool bObjectValid;
    ME_CAN_OBJECT()
    {
        gbDataValid[0] = false;
        gbDataValid[1] = false;
        gbDataValid[2] = false;
        bObjectValid = false;
    }
};

struct ME_CAN_OBJECTS
{
    bool bStatusAValid;
    DEV_CAN_MSG stStatusA;

    bool bStatusBValid;
    DEV_CAN_MSG stStatusB;

    bool bStatusCValid;
    DEV_CAN_MSG stStatusC;

    ME_CAN_OBJECT gstObjects[10];

    ME_CAN_OBJECTS()
    {
        bStatusAValid = false;
        bStatusBValid = false;
        bStatusCValid = false;
    }
};
```



- Step 2  fectch each data frame.

```cpp
void CMobileye::CanARecvThread()
{
    DEV_CAN_MSG gstCanDataFrame;
    DEV_CAN_ERROR errorValue_A = {DEV_CAN_ERROR_NONE, DW_SUCCESS};
    while(1)
    {
        //        errorValue_A = BSP_CAN_recv(m_stCanSetting.canSensor_A, &gstCanDataFrame, 100000);
        errorValue_A = BSP_CAN_recv(m_stCanSetting.canSensor_A, &gstCanDataFrame, 10000);//BSP_CAN_recv is the interface to get data
        if(errorValue_A.errorType == DEV_CAN_ERROR_NONE)
        {
            //            fprintf(stderr,"CAN message from CAN6\n");
//            PrintCanDataFrame(gstCanDataFrame);
            if(m_bLogCAN) // Using lcm to log debug data
            {
                m_msgCanMsg.header.nTimeStamp = GetGlobalTimeStampInMicroSec(); // So you have to define the global variable to hold the raw msg.
                m_msgCanMsg.header.nCounter = (static_cast<unsigned int>(m_msgCanMsg.header.nCounter)+1)%65535;//this 2 parts should be same for ever
                m_msgCanMsg.nCanID = gstCanDataFrame.id;
                memcpy(m_msgCanMsg.gbData, gstCanDataFrame.arryData, 8);
                m_pLCM->publish("CAN_MSG", &m_msgCanMsg);
            }
            ProcessCanDataFrame(gstCanDataFrame);
        }
        else
        {
            std::this_thread::sleep_for(std::chrono::microseconds(1000));
        }
    }
}
```

- Step  3 Parse each frame.

![favicon](../img/favicon.jpeg)

```
void CMobileye::ProcessCanDataFrame(const DEV_CAN_MSG& frame)
{
	// define some global configuralble flag ,for convenient use 
    if(m_bIsLane)
    {
        if ( frame.id>=0x766 && frame.id<=0x76F )
        // different range defines different types of MSG 
        {
#ifdef TEST_LOG
            fprintf(stderr,"log11\n");
#endif
            ProcessMELaneCanDataFrame ( frame);
#ifdef TEST_LOG
            fprintf(stderr,"log1\n");
#endif
            return;
        }
    }

    if(m_bIsObs)
    {
        if ( frame.id>=0x737 && frame.id<=0x757 )
        {
#ifdef TEST_LOG
            fprintf(stderr,"log21\n");
#endif
            ProcessMEObjectsCanDataFrame ( frame );
#ifdef TEST_LOG
            fprintf(stderr,"log2\n");
#endif
            return;
        }
    }

    if(m_bIsTFL)
    {
        if( frame.id>=0x712 && frame.id<=0x71A )
        {
#ifdef TEST_LOG
            fprintf(stderr,"log31\n");
#endif
            ProcessMETFLCanDataFrame( frame);
#ifdef TEST_LOG
            fprintf(stderr,"log3\n");
#endif
            return;
        }
    }
    if( frame.id == 0x690)
    {
#ifdef TEST_LOG
        fprintf(stderr,"log41\n");
#endif
        ProcessMEFailSafeCanDataFrame(frame);
#ifdef TEST_LOG
        fprintf(stderr,"log4\n");
#endif
        return;
    }
    if( (frame.id>=0x770 && frame.id<=0x785) || (frame.id>=0x7E0 && frame.id<=0x7E4) )
    {
#ifdef TEST_LOG
        fprintf(stderr,"log51\n");
#endif
        ProcessMEFreeSpaceCanDataFrame( frame );
#ifdef TEST_LOG
        fprintf(stderr,"log5\n");
#endif
        return;
    }
    if( frame.id>=0x720 && frame.id<=0x726 )
    {
#ifdef TEST_LOG
        fprintf(stderr,"log61\n");
#endif
//        ProcessMETSRCanDataFrame( frame );
#ifdef TEST_LOG
        fprintf(stderr,"log6\n");
#endif
        return;
    }
}
```

 the codes above defines where to process each CanDataFrame.

the follow is how to handle each CanDataFrame,but the first thing you need to do is check the data integrity of the date frame ,cause one group of date splits into several CAN msgs, so you have to accumulated untill get all the data (how to check all the date ,you need to read dbc ,and then define your own data struct to hold the whole data )

```
/**
 * @brief CMobileye::ProcessMELaneCanDataFrame
 * @param frame
 */
void CMobileye::ProcessMELaneCanDataFrame ( const DEV_CAN_MSG& frame )
{
    unsigned int nIndex = frame.id - 0x766; //just for convinent judge data integrity  and follow the order of the struct you defined
    m_stCanLanesMsg.gbCanFrameValid[nIndex] = true;
    memcpy ( m_stCanLanesMsg.gcCanDataFrame[nIndex], frame.arryData, 8 );
    if ( frame.id == 0x76B )
    {
        m_stCanLanesMsg.gbCanFrameValid[4] = true;//set 0x76A to true;

        //decompose lane data frame; and jump to the core part of data decompose.
        if ( DecomposeMELanesCanDataFrame(m_stCanLanesMsg) )
        {
            //publish lane data frame;
            PublishMELanes();
        }
        ResetMELanesCanDataFrame(); // finally we come to the end of this workflow ,just clear all the data.
    }
}
```

![favicon_whole](../img/favicon_whole.jpeg)

Step 4 Decompose Data according to dbc files.

```
const bool CMobileye::DecomposeMELanesCanDataFrame(const ME_CAN_LANES& can_lanes)
{
    bool bComplete = true;
    for ( int i=0; i<10; i++ )
    {
        bComplete = bComplete && can_lanes.gbCanFrameValid[i];
    }
    if ( !bComplete )
    {
        if(m_bIsFrameAll)
        {
            PrintErrorInfo ( "Uncomplete mobileye lane can data frames", __FILE__, __FUNCTION__, __LINE__ );
        }
        return false;
    }

    //left lane;
    Decompose0x766(can_lanes);
    Decompose0x767(can_lanes);

    //right lane;
    Decompose0x768(can_lanes);
    Decompose0x769(can_lanes);

    //next left lane;
    Decompose0x76C(can_lanes);
    Decompose0x76D(can_lanes);

    //next right lane;
    Decompose0x76E(can_lanes);
    Decompose0x76F(can_lanes);

    Decompose0x76B(can_lanes);
    return true;
}
```

 Step 5 Decompose data in the struct you defined ,and fill the interface datatype with other module(for quick develop just write one and copy)

```
inline void CMobileye::Decompose0x766(const ME_CAN_LANES& can_lanes)
{
    unsigned short nLaneType = can_lanes.gcCanDataFrame[0][0] & 0x0F;

    unsigned short nQuality = ( can_lanes.gcCanDataFrame[0][0] & 0x30 ) >>4;

    int nNumber = can_lanes.gcCanDataFrame[0][2]*256 + can_lanes.gcCanDataFrame[0][1];
    nNumber = ( ( can_lanes.gcCanDataFrame[0][2] & 0x80 ) ==0x00 ) ?  nNumber: - ( ( ~ ( nNumber-1 ) ) &0x7FFF );
    float fPosition = CanNumber2Float ( nNumber, 0.00390625, 0.0f );

    nNumber = can_lanes.gcCanDataFrame[0][4]*256 + can_lanes.gcCanDataFrame[0][3];
    //nNumber = ( ( m_stCanLanes.gcCanDataFrame[0][4] & 0x80 ) ==0x00 ) ?  nNumber : - ( ( ~ ( nNumber-1 ) ) &0x7FFF );
    float fCurvature =  CanNumber2Float ( nNumber, 0.000000976563, - 0.031999023438 );

    nNumber = can_lanes.gcCanDataFrame[0][6]*256 + can_lanes.gcCanDataFrame[0][5];
    //nNumber = ( ( m_stCanLanes.gcCanDataFrame[0][6] & 0x80 ) ==0x00 ) ? nNumber : - ( ( ~ ( nNumber-1 ) ) &0x7FFF );
    float fCurvatureDerivative = CanNumber2Float ( nNumber, 0.00000000372529, -0.00012206658721 );
    float fWidth = CanNumber2Float ( can_lanes.gcCanDataFrame[0][7], 0.01f, 0.0f );

    m_msgMobileyeLanes.stLeftLine.nLineType = nLaneType;
    m_msgMobileyeLanes.stLeftLine.nQuality = nQuality;
    m_msgMobileyeLanes.stLeftLine.a = fPosition;
    m_msgMobileyeLanes.stLeftLine.c = fCurvature;
    m_msgMobileyeLanes.stLeftLine.d = fCurvatureDerivative;
    m_msgMobileyeLanes.stLeftLine.fWidth = fWidth;
    return;
}
```



## Issue Solve

- PrintCanDataFrame 

  ![BugFixStep](../img/BugFixStep.jpeg)

