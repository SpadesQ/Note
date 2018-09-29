## Q&A

https://github.com/jwyang/faster-rcnn.pytorch

1.sh make.sh
找不到nvcc

下面过程只需做一次，下次直接source /etc/profile

在 /etc/profile 文件末尾添加如下环境变量
```
export PATH=/usr/local/cuda-9.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
然后在darknet目录中
```
. /etc/profile
source /etc/profile
```

2.make.sh 不是有效字符啥的
重新git clone 工程

3.测试时候File "/home/rs/myGithub/py-faster-rcnn/tools/../lib/datasets/voc_eval.py", line 127, in voc_eval R = [obj for obj in recs[imagename] if obj['name'] == classname]
KeyError: '000001'

Hey,
had the same problem with an old annotation cache. Just delete it
rm data/VOCdevkit2007/annotations_cache/annots.pkl and run the program again.
You should also delete the roidb cache:
rm data/cache/voc_2007_trainval_gt_roidb.pkl

因为上次运行完没删，再运行就出错。