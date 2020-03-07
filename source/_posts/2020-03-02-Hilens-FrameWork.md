---
title: Hilens FrameWork
date: 2020-03-02 23:39:07
tags:Hilens
categories: 笔记
---

# Hilens Framework

- 初始化Hilens

  ```python
  import hilens
  def verify():   
      return "hello"
  
  def main():
      # 初始化HiLens
      rc = hilens.init(verify())
  
      # 完成后，释放资源
      hilens.terminate()
  
  if __name__ == '__main__':
      main()
  ```

## 输入模块

- 视频采集器

  接口调用 hilens.VideoCapture(camera)

  返回值为摄像头的采集器

  由于此文档为初学文档，建议不传值。默认使用Hilens Kit自带的摄像头

- 读取摄像头上的视频帧

  hilens.VideoCapture.read(),用于读取一帧数据。返回YUV_NV21颜色排布数据。参数为numpy数组（dtype为unit8），兼容CV2

- 获取视频宽度

  hilens.VideoCapture.width

- 获取视频高度

  hilens.VideoCapture.height

- 示例

  ```python
  #! /usr/bin/python3.7
  
  import hilens
  import numpy as np
  
  def run():
      # 构造摄像头
      cap0 = hilens.VideoCapture()           # 自带摄像头
      cap1 = hilens.VideoCapture("IPC1")     # 摄像头配置中name为"IPC1"的IPC。摄像头配置可登录Huawei Hilens
                                             # 控制台，在“技能开发>技能管理>新建技能”中的“运行时配置”添加
      cap2 = hilens.VideoCapture("rtsp://192.168.1.1/video") # 地址为rtsp://192.168.1.1/video的RTSP视频流
      cap3 = hilens.VideoCapture(0)          # 目前只支持单路uvc摄像头，编号为0
  
      # 获取视频尺寸
      w = cap0.width
      h = cap0.height
      hilens.info("width: %d, height: %d" % (w, h))
  
      # 读取视频数据
      frame0 = cap0.read()
      # 其他处理
      pass
  
  if __name__ == '__main__':
      hilens.init("hello") 
      run()
      hilens.terminate()
  ```

  