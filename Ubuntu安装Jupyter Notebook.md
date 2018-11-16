## cannot uninstall a distutils installed project'

sudo pip install --ignore-installed tornado即可

## OSError: [Errno 99] Cannot assign requested address异常

运行Jupyter时增加--ip=0.0.0.0参数
root@787c084a44e4:~# jupyter notebook --ip=0.0.0.0 --no-browser --allow-root