# Single-Shot 3D Detection of Vehicles from Monocular RGB Images via Geometry Constrained Keypoints in Real-Time

------

原文链接：[点这里](https://arxiv.org/abs/2006.13084v1)

## 目录

- [1. 摘要](#1)
- [2. 引言](#2)
- [3. CenterPoint](#3)
- [4. 实验](#4)

<a name="1"></a>

## 1. 摘要

提出一种新的3D单目目标检测方法，用于检测单目RGB图像中的车辆。通过预测额外的回归和分类参数将2D检测扩展到3D空间，从而使运行时间接近纯2D目标检测。在几何约束下附加参数转换为网络内的3D边界框关键点。所提方法具有完整的3D描述，包括三个旋转角度，因为是聚焦于图像平面内的某些关键点，所以可以通过无监督方式获取目标的方向。所提方法可以与任何目前的目标检测框架相结合，只需很少的计算开销，并举例说明在SSD上如何预测3D边界框。在不同的自动驾驶数据集上测试了该方法；使用KITTI 3D目标检测基准和nuScenes目标检测基准对其进行了评估，都取得了有竞争力的结果。在所有测试数据集和图像分辨率方面都优于SOTA方法，速度超过20 FPS。

<a name="2"></a>

## 2. 引言



<a name="3"></a>

## 3. CenterPoint



<a name="4"></a>

## 4. 实验





