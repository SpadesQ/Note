## Install YOLO v3

#### 安装这个流程无误[YOLO]（https://pjreddie.com/darknet/yolo/）

#### 使用GPU注意事项
makefile文件里
```
GPU=1
CUDNN=1
OPENCV=1
OPENMP=0
DEBUG=0
```
然后重新make

#### 可能出错的地方
- opencv
请确保opencv已安装好

- darknet - yolo - /bin/sh: 1: nvcc: not found
在 /etc/profile 文件末尾添加如下环境变量
```
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
然后在darknet目录中
```
 . /etc/profile
 source /etc/profile
 make clean
 make
 ```
其中 . 和 /etc/profile 之间有空格

- error while loading shared libraries: libcudart.so.9.0: cannot open shared object file: No such file
- error while loading shared libraries: libcudnn.so.7: cannot open shared object file: No such file or directory.
```
sudo cp /usr/local/cuda-9.0/lib64/libcudart.so.9.0 /usr/local/lib/libcudart.so.9.0 && sudo ldconfig
sudo cp /usr/local/cuda-9.0/lib64/libcublas.so.9.0 /usr/local/lib/libcublas.so.9.0 && sudo ldconfig
sudo cp /usr/local/cuda-9.0/lib64/libcurand.so.9.0 /usr/local/lib/libcurand.so.9.0 && sudo ldconfig
sudo cp /usr/local/cuda-9.0/lib64/libcudnn.so.7 /usr/local/lib/libcudnn.so.7 && sudo ldconfig

```


