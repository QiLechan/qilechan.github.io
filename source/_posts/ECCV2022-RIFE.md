---
title: AI补帧：ECCV2022-RIFE算法初尝试
tags:
- 技术
categories:
- 技术
date: 2023-01-27
excerpt: AI补帧算法ECCV2022-RIFE的初次尝试
---
## 什么是补帧
通过提高每秒展示的帧数，来使得画面更为流畅，称为补帧。
每一帧都是静止的图象，快速连续地展示帧是眼睛产生了画面在运动的假象。一般而言，帧数越高，画面会越流畅。
补帧通过在原视频中插入连续的画面，来达到画面流畅的目的。
AI补帧即是通过深度学习算法，在两帧之间生成更多帧，达到提高帧数的目的。

## ECCV2022-RIFE算法
ECCV2022-RIFE算法是一种先进的人工智能算法RIFE的衍生算法，由旷视科技和北京大学的研究者提出。项目地址位于[GitHub](https://github.com/megvii-research/ECCV2022-RIFE)
这种高效的实时中间流估计算法，不仅运行速度实现了数倍甚至数十倍的提升，而且伪影也较以往方法少得多。

## 算法的部署与运行
以下内容在系统为Ubuntu 20的服务器上经过测试，使用的GPU为NVIDIA TITAN Xp。

### 部署算法
（请配置好Python3环境）
首先使用Git克隆仓库，或是直接下载zip包。将其放在合适的目录。
在程序目录根部，可以看到名为requirements.txt，这是运行脚本所需要的的库的清单。在该目录打开终端，输入“pip3 install -r requirements.txt”以安装依赖的库。
接着安装ffmpeg，Ubuntu下使用包管理器可以快速安装（sudo apt install ffmpeg),如果使用Windows，请至官方下载网站下载[安装](https://ffmpeg.org/download.html)，并配置好环境变量。
由于该算法是深度学习算法，可以对其进行训练。这里我们使用项目作者所提供的[训练模型](https://drive.google.com/file/d/1APIzVeI-4ZZCEuIRE1m6WYfSCaOsi_7_/view?usp=sharing)。
下载完成后，将其解压至程序根目录。
算法部署完成。

### 运行算法
这里只给出简单的例子，更多用法请看[这里](https://github.com/megvii-research/ECCV2022-RIFE#run)
`python3 inference_video.py --exp=1 --video=video.mp4`
生成二倍帧率的视频
`python3 inference_video.py --exp=2 --video=video.mp4`
生成四倍帧率的视频

## 实例效果

### Demo
原视频（25fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/demo.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>
2X（50fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/demo_2X_50fps.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>

### 动漫
原视频（24fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/snqx1.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>
2X（48fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/snqx1_2X_48fps.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>
4X（96fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/snqx1_4X_96fps.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>
可以看到，该算法在动漫补帧上有一定缺陷，需要配合其他工具进行优化。算法对重复帧处理效果不佳，导致人物嘴部出现问题，以及动漫的转场也有问题。

原视频（30fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/stm2.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>
2X（60fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/stm2_2X_60fps.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>
4X（120fps）
<div style="position: relative;width: 100%; height: 0 padding-bottom: 75%;"><video controls="" style="text-align:center; width: 100%; height: 100% padding-bottom: 75%;"><source src="https://cos.qileoffice.top/stm2_4X_120fps.mp4" type="video/mp4">您的浏览器不支持HTML5，无法播放此视频。请升级您的浏览器</video></div>