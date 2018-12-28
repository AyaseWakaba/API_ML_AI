# Project Requirements 
Target release发布日期|11-25-2018
--|:--:
Epic史诗名称|人与机械同步
Document Status文件现状|未起头
Designer|王汉彦
Developer|王汉彦
QA|王汉彦
Document owner|王汉彦

# 加值宣言 
主要： 运用了人工智能中计算机视觉的图像分析识别人的手势，与语音识别来分析人的语音命令，通过输入手势、语音，输出机械臂的控制信息与机械臂的控制动作。 
辅助： 还运用了机器学习的技术，在网络上共享不同的轨迹与手势，用前人的经验与设置完成各种任务。 
API加值：使用计算机视觉技术的人工智能api，可以直接输入手势命令而无需

# 核心价值 
本产品让科技产品更加快地融合到生活里面，让用户随心所欲地、无门槛地控制机械。 


# Background and strategic fit背景和战略契合处 
现今控制机械臂一般需要熟悉编程，或者利用其他软件辅助，本产品提供一种利用动作捕捉人工智能与语音控制来识别手势从而控制机械臂的软件，使机械臂更加容易被大众运用 

# 用户 
摄像拍摄人员
智能家具爱好者

# 用户痛点 
很多用户想要用机械臂来改善生产力，但目前大部分机械臂都需要懂得编程与熟悉硬件方面的知识，使门槛变高。
* 想用机械臂，但不会编程。
* 搬运重物的时候找不到有空的人帮忙，需要一个力气大的机械。 
* 导演需要拍摄一个运镜难度极高的镜头，或者难以描述的镜头，需要自己设定轨迹与运动。

# AI产品概率性与用户痛点 
百度视觉技术的图像识别技术，有三大保证：
* 稳定性好提供24小时云端高稳定服务，宕机率低，故障恢复快，单图毫秒级响应，服务可用性高达99.95%
* 准确度高：基于百度丰富的海量数据，利用深度学习技术及精准的算法迭代模型，不断提高准确性
* 功能丰富：支持上千种物体识别及场景识别，并在持续增加中，让你更好的读懂世界。
 
所以，该产品利用了手势识别通过现场拍的手势分析出手势的形状，再把手势转换为数字信息输入到机械臂的控制器里，从而控制机械臂。这个功能技术转换精确率较高，普遍情况下都可以使用。该产品因环境因素或者拍照造成识别不准确的状况，概率较小为少数事件，对正面影响并不大。

# 最小化可用产品 
手势识别
语音控制输入
手势识别信息输入

# 核心功能所应用API 
*** HandVu进行手部动作识别分析 ***  
*** 百度语音识别 *** 
*** 百度视觉技术 *** 

# Goals目标 
前期目标
* 实验api的可行性与精度
* 将api分析的输出信息转换为各关节电机的数字信号

后期目标：（目前不做）
* 机械臂拥有智能避障的功能 
* 真的做出一个机械臂


# Assumptions: 列出所有假设 
用户使用主要功能时，主要使用带有摄像头的电脑，与一条已连接电脑的机械臂。

# 需求 Requirements  
 
Title |User story用户案例 |Importance |Note   
--|:--:|:--:|--: 
摄像 |摄像师利用一些大型摄录机器拍摄时，可直接利用机械学习出来的运动轨迹，而无需再找一个熟悉机械臂控制的人员来操作 |重要 |制作轨迹  
智能家居爱好者 |一个容易控制的机械臂能代替很多智能家具，比如利用机械臂打开窗帘、打开电视机等等 |一般 |智能家具

# 使用者交互及设计: User interaction and design 
[产品原型](https://ayasewakaba.github.io/api_AXURE/ "产品原型")

# 问题 Questions 
初步能利用机械臂模仿使用者在摄像头前的手势

# 不做 Not doing 
* 机械臂拥有智能避障的功能 
* 真的做出一个机械臂
# 代码展示
```
* HandVu - a library for computer vision-based hand gesture
* recognition.
* Copyright (C) 2004 Mathias Kolsch, matz@cs.ucsb.edu
*
* This program is free software; you can redistribute it and/or
* modify it under the terms of the GNU General Public License
* as published by the Free Software Foundation; either version 2
* of the License, or (at your option) any later version.
*
* This program is distributed in the hope that it will be useful,
* but WITHOUT ANY WARRANTY; without even the implied warranty of
* MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
* GNU General Public License for more details.
*
* You should have received a copy of the GNU General Public License
* along with this program; if not, write to the Free Software
* Foundation, Inc., 59 Temple Place - Suite 330, 
* Boston, MA  02111-1307, USA.
*
* $Id: hv_OpenCV.cpp,v 1.15 2006/01/03 21:44:15 matz Exp $
**/

#ifdef WIN32
#include <windows.h>
#endif

#include <stdio.h>
#include <cv.h>
#include <highgui.h>
#include <ctype.h>
#include <time.h>

#include "HandVu.h"


IplImage *capture_image = 0;
IplImage *display_image = 0;

bool async_processing = false;
int num_async_bufs = 30;
IplImage *m_async_image = 0;
int m_async_bufID = -1;
bool sync_display = true;

CvPoint origin;
int select_object = 0;
int sel_area_left=0, sel_area_top=0, sel_area_right=0, sel_area_bottom=0;
bool correct_distortion = false;


void OnMouse( int event, int x, int y, int /*flags*/, void* /*params*/ )
{
  if( !capture_image )
    return;

  if( capture_image->origin )
    y = capture_image->height - y;

  if( select_object )
  {
    sel_area_left = MIN(x,origin.x);
    sel_area_top = MIN(y,origin.y);
    sel_area_right = sel_area_left + CV_IABS(x - origin.x);
    sel_area_bottom = sel_area_top + CV_IABS(y - origin.y);

    sel_area_left = MAX( sel_area_left, 0 );
    sel_area_top = MAX( sel_area_top, 0 );
    sel_area_right = MIN( sel_area_right, capture_image->width );
    sel_area_bottom = MIN( sel_area_bottom, capture_image->height );

    if( sel_area_right-sel_area_left > 0 && sel_area_bottom-sel_area_top> 0 )
      hvSetDetectionArea(sel_area_left, sel_area_top,
                         sel_area_right, sel_area_bottom);
  }

  switch( event )
  {
  case CV_EVENT_LBUTTONDOWN:
    origin = cvPoint(x,y);
    sel_area_left = sel_area_right = x;
    sel_area_top = sel_area_bottom = y;
    select_object = 1;
    break;
  case CV_EVENT_LBUTTONUP:
    select_object = 0;
    break;
  }
}


void showFrame(IplImage* img, hvAction action)
{
  if (action==HV_DROP_FRAME) {
    // HandVu recommends dropping the frame entirely
    // printf("HandVuFilter: dropping frame/n");
    return;
  } else if (action==HV_SKIP_FRAME) {
    // HandVu recommends displaying the frame, but not doing any further
    // processing on it - keep going
    // printf("HandVuFilter: supposed to skip frame/n");
  } else if (action==HV_PROCESS_FRAME) {
    // full processing was done and is recommended for following steps;
    // keep going
    //printf("HandVuFilter: processed frame/n");
  } else {
    assert(0); // unknown action
  }
  
  hvState state;
  hvGetState(0, state);
  
  cvShowImage( "HandVu", img );
}


void displayCallback(IplImage* img, hvAction action)
{
  if (sync_display) {
    cvCopy(img, display_image);
  } else {
    showFrame(img, action);
  }
}


int main( int argc, char** argv )
{
  CvCapture* capture = 0;

  if (argc<2) {
    printf("you need to specify a conductor file as first argument/n");
    printf("for example: ../config/default.conductor/n");
    return -1;
  }

  string conductor_fname(argv[1]);//声明配置参数的对象
  printf("will load conductor from file:/n%s/n", conductor_fname.c_str());//屏显提示

  /*是否设定特定的摄像头，并初始化摄像头  */
  if( argc == 2 || argc == 3) {
    int num = 0;
    if (argc==3) {
      num = atoi(argv[2]);
    }
    capture = cvCaptureFromCAM( num );
    if (!capture) {
      capture = cvCaptureFromAVI( argv[2] ); 
    }
  }

  if( !capture )
  {
    fprintf(stderr,"Could not initialize capturing through OpenCV./n");
    return -1;
  }

  /* 屏显提示 */
  printf( "Hot keys: /n"
    "/tESC - quit the program/n"
    "/tr - restart the tracking/n"
    "/t0-3 - set the overlay (verbosity) level/n"
    "use the mouse to select the initial detection area/n" );

  //设定采集图像大小
  int p = 0; // according to docs, these calls don't work in OpenCV beta 4 yet
  p = cvSetCaptureProperty(capture, CV_CAP_PROP_FRAME_WIDTH, 640);
  p = cvSetCaptureProperty(capture, CV_CAP_PROP_FRAME_HEIGHT, 480);

  //获取一帧
  capture_image = cvQueryFrame( capture );
  if ( !capture_image ) {
    fprintf(stderr,"Could not retrieve image through OpenCV./n");
    return -1;
  }

  /* allocate all the buffers */
  CvSize size = cvGetSize(capture_image);
  hvInitialize(size.width, size.height);//初始化要分析的图像大小
  hvLoadConductor(conductor_fname);//装载参数
  hvStartRecognition();//开始识别
  hvSetOverlayLevel(2);//设置识别的覆盖区级别
  /* 设置同步／异步识别 */
  if (async_processing) {
    hvAsyncSetup(num_async_bufs, displayCallback);
    if (sync_display) display_image = cvCloneImage(capture_image);
  }
  /* 设置鼠标事件的回调参数 */
  cvSetMouseCallback( "HandVu", OnMouse );
  /* 设置窗口 */
  int success = cvNamedWindow( "HandVu", 1 );
  if (success!=1) {
    printf("can't open window - did you compile OpenCV with highgui support?");
    return -1;
  }
  fprintf(stderr, "initialized highgui/n");
    hvStartGestureServer(1394,10);

  for (;;) {
    int c;
    
    if (async_processing) {
      // asynchronous processing in HandVu

      if (sync_display) cvShowImage("HandVu", display_image);
      
      // ------- main library call ---------
      hvAsyncGetImageBuffer(&m_async_image, &m_async_bufID);
      cvCopy(capture_image, m_async_image);
      hvAsyncProcessFrame(m_async_bufID);
      // -------
      
    } else {//In the condition从这个分支走
      // synchronous processing in HandVu
      
      // ------- main library call ---------
      hvAction action = HV_INVALID_ACTION;
      action = hvProcessFrame(capture_image);
      // -------
      
      showFrame(capture_image, action);
      
    }
    
    c = cvWaitKey(10);
    if( c == 27 || c == 'q' )
      break;
    switch( c )
      {
      case 'r':
        hvStopRecognition();
        hvStartRecognition();
        break;
      case '0':
        hvSetOverlayLevel(0);
        break;
      case '1':
        hvSetOverlayLevel(1);
        break;
      case '2':
        hvSetOverlayLevel(2);
        break;
      case '3':
        hvSetOverlayLevel(3);
        break;
      case 'u':
        if (hvCanCorrectDistortion()) {
          correct_distortion = !correct_distortion;
          hvCorrectDistortion(correct_distortion);
        }
        break;
      default:
        ;
      }
    
    // capture next image
    capture_image = cvQueryFrame( capture );
    if ( !capture_image ) {
      fprintf(stderr,"Could not retrieve image through OpenCV./n");
      break;
    }
  }
  
  cvReleaseCapture( &capture );
  cvDestroyWindow("HandVu");

  return 0;
}
```

## 清单 
[产品原型](https://ayasewakaba.github.io/api_AXURE/ "产品原型")

