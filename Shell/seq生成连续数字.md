```shell
[root@ubuntu]# seq 2
1
2

[root@ubuntu]# seq 1 5
1
2
3
4
5

[root@ubuntu]# seq -s " " 1 10
1 2 3 4 5 6 7 8 9 10

[root@ubuntu]# seq -f %05g 1 5
00001
00002
00003
00004
00005

[root@ubuntu]# seq -w 1 5
01
02
03
04
05

[root@ubuntu]# for i in `seq 1 5`;do echo $i;done
1
2
3
4
5

[root@ubuntu]# for i in $(seq 1 5);do echo $i;done
1
2
3
4
5
 
说明：
-s 指定分隔符，默认是换行
-w 等位补全，就是宽度相等，不足的前面补 0
-f 格式化输出，就是指定打印的格式

可以不指定起始数值，则默认为 1，见第 1 行例子

另外，不用 seq 的话还可以这样：
[root@ubuntu]# for i in {1..10};do echo $i;done
1 和 10 之间是两个半角的点
```

