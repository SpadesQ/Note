## Faster-RCNN运行时遇到的问题

从https://github.com/smallcorgi/Faster-RCNN_TF下载源码，根据说明下载各种文件

在运行demo的时候遇到错误

roi_pooling_layer/roi_pooling.so: undefined symbol: _ZTIN10tensorflow8OpKernelE

第一步到lib下查看make.sh文件，添加

TF_LIB=$(python -c 'import tensorflow as tf; print(tf.sysconfig.get_lib())')

第二步修改g++配置，如下所示：

g++ -std=c++11 -shared -o roi_pooling.so roi_pooling_op.cc -D_GLIBCXX_USE_CXX11_ABI=0 \
roi_pooling_op.cu.o -I $TF_INC -L $TF_LIB -ltensorflow_framework -D GOOGLE_CUDA=1 \

-fPIC $CXXFLAGS -lcudart -L $CUDA_PATH/lib64

第三步，查看自己的CUDA路径，CUDA_PATH=/usr/local/cuda-9.0/，查看/usr/local是否存在cuda，根据自己的文件夹名字修改路径，随后在终端中执行export PATH=$PATH:/usr/local/cuda-9.0/bin/ (/usr/local/cuda-9.0为你的CUDA路径) ，并修改make.sh文件，将CXXFLAGS+='-undefined dynamic_lookup'修改为如下所示

CXXFLAGS='-D_MWAITXINTRIN_H_INCLUDED'

最后重新make，回到根目录运行demo

修改后的make.sh文件如下：
```
TF_INC=$(python -c 'import tensorflow as tf; print(tf.sysconfig.get_include())')

TF_LIB=$(python -c 'import tensorflow as tf; print(tf.sysconfig.get_lib())')
CUDA_PATH=/usr/local/cuda-9.0/  #此处修改为自己环境下的CUDA路径
CXXFLAGS=''

if [[ "$OSTYPE" =~ ^darwin ]]; then
         CXXFLAGS='-D_MWAITXINTRIN_H_INCLUDED'
fi

cd roi_pooling_layer

if [ -d "$CUDA_PATH" ]; then
        nvcc -std=c++11 -c -o roi_pooling_op.cu.o roi_pooling_op_gpu.cu.cc \
                -I $TF_INC -D GOOGLE_CUDA=1 -x cu -Xcompiler -fPIC $CXXFLAGS \

                -arch=sm_37


        g++ -std=c++11 -shared -o roi_pooling.so roi_pooling_op.cc -D_GLIBCXX_USE_CXX11_ABI=0 \
roi_pooling_op.cu.o -I $TF_INC -L $TF_LIB -ltensorflow_framework -D GOOGLE_CUDA=1 \
-fPIC $CXXFLAGS -lcudart -L $CUDA_PATH/lib64      #修改后的g++配置

else
        g++ -std=c++11 -shared -o roi_pooling.so roi_pooling_op.cc \
                -I $TF_INC -fPIC $CXXFLAGS
fi

cd ..
```
