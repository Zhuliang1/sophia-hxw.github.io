---
layout: post
title:  "darknet2caffe"
date:   2021-07-08 10:28:23 +0700
categories: [hisi]
---
# 1.安装caffe docker
### 更换apt源
```
mv /etc/apt/sources.list /etc/apt/sources.list.bak
rm /etc/apt/sources.list.d/ -r
echo deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse >> /etc/apt/sources.list
echo deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse >> /etc/apt/sources.list
echo deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse >> /etc/apt/sources.list
echo deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse >> /etc/apt/sources.list
echo deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse >> /etc/apt/sources.list
apt-get update
```

### 安装vim
```
apt-get install vim
```

# 2.添加layers，重新编译caffe
```
Copy
caffe_layers/mish_layer/mish_layer.hpp,caffe_layers/upsample_layer/upsample_layer.hpp
into
include/caffe/layers/
Copy 
caffe_layers/mish_layer/mish_layer.cpp mish_layer.cu,caffe_layers/upsample_layer/upsample_layer.cpp upsample_layer.cu into src/caffe/layers/Copy 
caffe_layers/pooling_layer/pooling_layer.cpp
into 
src/caffe/layers/
Add below code into src/caffe/proto/caffe.proto

// LayerParameter next available layer-specific ID: 147 (last added: recurrent_param) 
message LayerParameter { 
    optional TileParameter tile_param = 138; 
    optional VideoDataParameter video_data_param = 207; 
    optional WindowDataParameter window_data_param = 129; 
++optional UpsampleParameter upsample_param = 149; 
//added by chen for Yolov3, make sure this id 149 not the same as before. 
++optional MishParameter mish_param = 150; 
//added by chen for yolov4,make sure this id 150 not the same as before. 
} 
// added by chen for YoloV3 
++message UpsampleParameter{ 
++     optional int32 scale = 1 [default = 1]; 
++} 
// Message that stores parameters used by MishLayer 
++message MishParameter { 
++     enum Engine { 
++         DEFAULT = 0; 
++         CAFFE = 1; 
++         CUDNN = 2; 
++     } 
++ optional Engine engine = 2 [default = DEFAULT]; 
++}
```
# 3.remake caffe.
### 生成caffe.pb.h
```
mkdir include/caffe/proto protoc ./src/caffe/proto/caffe.proto --cpp_out=. mv ./src/caffe/proto/caffe.pb.h ./include/caffe/proto/
```
### 修改Makefile.config
```
cd /opt/caffe
cp Makefile.config.example Makefile.config
vim Makefile.config
```
### 修改Makefile
```
make all -j8
make runtest -j8
make pycaffe
```
### 模型转换
```
cd darknet2caffe
python darknet2caffe.py cfg/yolov4.cfg weights/yolov4.weights prototxt/yolov4.prototxt caffemodel/yolov4.caffemodel
```