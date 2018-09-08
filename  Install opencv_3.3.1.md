## Install opencv_3.3.1

### 进入 Opencv 官网 opencv.org, 选择最新版本的 OpenCV 下载(我这里下载的是 OpenCV 3.3.1).
### 安装libgtk2.0-dev和pkg-config
执行命令
```
sudo apt-get update
sudo apt-get install libgtk2.0-dev
sudo apt-get install pkg-config
```
### 安装opencv-3.3.1
```
unzip opencv-3.3.1.zip 
cd opencv-3.3.1
mkdir build
cd build
cmake -D CMAKEBUILDTYPE=RELEASE -D CMAKEINSTALLPREFIX=/usr/local ..
make
make install
```

