## Install opencv_3.3.1

#### ubuntu下卸载opencv步骤
进入build文件夹，命令行执行下列命令。如新安装，此步骤可跳过。
```
make uninstall
cd ..
sudo rm -r build
sudo rm -r /usr/local/include/opencv2 /usr/local/include/opencv /usr/include/opencv /usr/include/opencv2 /usr/local/share/opencv /usr/local/share/OpenCV /usr/share/opencv /usr/share/OpenCV /usr/local/bin/opencv* /usr/local/lib/libopencv*
```

#### 进入 Opencv 官网 [opencv.org](opencv.org), 选择最新版本的 OpenCV 下载(我这里下载的是 OpenCV 3.3.1)

#### 安装libgtk2.0-dev和pkg-config
```
sudo apt-get update
sudo apt-get install libgtk2.0-dev
sudo apt-get install pkg-config
```

#### 安装opencv-3.3.1
```
unzip opencv-3.3.1.zip 
cd opencv-3.3.1
mkdir build
cd build
cmake -D CMAKEBUILDTYPE=RELEASE -D CMAKEINSTALLPREFIX=/usr/local ..
make
make install
```
