## Install opencv_3.3.1

```
unzip opencv-3.3.1.zip 
cd opencv-3.3.1
mkdir build
cd build
cmake -D CMAKEBUILDTYPE=RELEASE -D CMAKEINSTALLPREFIX=/usr/local ..
make -j8
make all install -j8
sudo make install
```



