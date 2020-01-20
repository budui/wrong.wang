---
title: "安装标准的caffe和opencv"
date: 2019-10-16T16:09:16+08:00
author: Ray Wong
---

如果只是使用最标准的caffe，没有自定义layer的话，使用`conda`安装caffe很简单：

```bash
# 创建一个虚拟环境，使用python2
conda create -n caffe-python2 python=2.7
# 激活这个环境
conda activate caffe-python2
# 安装GPU版本的caffe，如果需要，CUDNN等等依赖conda也会帮你装好
conda install -c defaults caffe-gpu
# 安装2.4版本的OpenCV
conda install -c https://conda.binstar.org/menpo opencv
```
