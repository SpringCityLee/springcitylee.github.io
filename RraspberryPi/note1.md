### 树莓派重新编译rxtx生成适配arm系统的so文件

------

![librxtxSerial-2.1-7.so](http://note.youdao.com/yws/res/230/2B2B2FCBC1DD4A15A8777825F1D9D2B3)

（1）wget http://rxtx.qbang.org/pub/rxtx/rxtx-2.1-7r2.zip

（2）unzip rxtx-2.1-7r2.zip 

（3）cd rxtx-2.1-7r2 编辑configure文件，把1.2*|1.3*|1.4*|1.5*全部替换成1.2*|1.3*|1.4*|1.5*|1.6*|1.7*|1.8*

（4）uname -r 查看内核版本，编辑/usr/include/linux/version.h末尾添加#define UTS_RELEASE "系统版本"

然后执行以下命令：

1: sh ./configure

2: make

3: make install

参考: http://rxtx.qbang.org/wiki/index.php/Installation_on_Linux

​	 https://blog.csdn.net/hekaibang/article/details/80420109

​	 https://github.com/inindev/raspberry-pi-rxtxlib