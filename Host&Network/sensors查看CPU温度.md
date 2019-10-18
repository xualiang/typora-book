在Linux下可以通过lm_sensors来查看CPU的温度(当然你的硬件首先要支持）,要使用这个功能要有内核相关模块(比如I2C)的支持，下面说一下操作方法：

先看一下你的机器上是否安装了lm_sensors,

$ rpm -q lm_sensors

 

如果没有安装就先安装一下

$ sudo yum install -y lm_sensors

 

检测传感器：

$ sudo sh -c "yes|sensors-detect"

 

 

如果以上步骤没有问题，执行下面的命令就可以查看CPU的温度了：

$ sensors

 

it8718-isa-0290

Adapter: ISA adapter

in0:       +1.07 V  (min =  +0.00 V, max =  +4.08 V)

in1:       +1.82 V  (min =  +0.00 V, max =  +4.08 V)

in2:       +3.36 V  (min =  +0.00 V, max =  +4.08 V)

in3:       +2.94 V  (min =  +0.00 V, max =  +4.08 V)

in4:       +0.29 V  (min =  +0.00 V, max =  +2.10 V)

in5:       +4.08 V  (min =  +0.00 V, max =  +4.08 V)   ALARM

in6:       +4.08 V  (min =  +0.00 V, max =  +4.08 V)   ALARM

in7:       +3.07 V  (min =  +0.00 V, max =  +4.08 V)

in8:       +3.09 V

fan1:     2463 RPM  (min =    0 RPM)

fan2:        0 RPM  (min =    0 RPM)

temp1:       -55°C  (low  =  +127°C, high =  +127°C)   sensor = thermistor                                        

temp2:        -2°C  (low  =  +127°C, high =  +127°C)   sensor = thermistor                                        

temp3:       +21°C  (low  =  +127°C, high =  +127°C)   sensor = diode

vid:      +2.050 V

 

coretemp-isa-0000

Adapter: ISA adapter

Core 0:      +44°C  (high =  +100°C)

 

coretemp-isa-0001

Adapter: ISA adapter

Core 1:      +27°C  (high =  +100°C)

 

 

\--------------------- 

作者：gaoming655 

来源：CSDN 

原文：https://blog.csdn.net/gaoming655/article/details/7320457 

版权声明：本文为博主原创文章，转载请附上博文链接！