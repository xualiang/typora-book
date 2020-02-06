```sh
测试脚本如下：
[root@hd01 ~]# cat test.sh 
echo 123
export haha=456
echo $haha

使用sh调用，脚步中的export不生效
[root@hd01 ~]# sh test.sh 
123
456
[root@hd01 ~]# echo $haha

使用.调用，脚步中的export生效
[root@hd01 ~]# . test.sh 
123
456
[root@hd01 ~]# echo $haha
456


```

