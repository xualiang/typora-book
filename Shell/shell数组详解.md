## 1、数组定义

```sh

[root@bastion-IDC ~]$ a=(1 2 3 4 5 6 7 8)
[root@bastion-IDC ~]$ echo $a
```



##2、数组读取与赋值

### 1）得到长度：

```sh
[root@bastion-IDC ~]$ echo ${#a[@]}
8
[root@bastion-IDC ~]$ echo ${#a[*]}
8
```

### 2）读取：

```sh
[root@bastion-IDC ~]$ echo ${a[4]}
5
[root@bastion-IDC ~]$ echo ${a[*]}
1 2 3 4 5 6 7 8
用${数组名[下标]} 下标是从0开始 下标是：*或者@ 得到整个数组内容
```

###3）赋值: 

```sh
[root@bastion-IDC ~]$ a[1]=100
[root@bastion-IDC ~]$ echo ${a[*]} 
1 100 3 4 5 6 7 8
[root@bastion-IDC ~]$ a[5]=140
[root@bastion-IDC ~]$ echo ${a[*]} 
1 100 3 4 5 140 7 8
直接通过 数组名[下标] 就可以对其进行引用赋值，如果下标不存在，自动添加新一个数组元素
```

### 4）删除:

```sh
[root@bastion-IDC ~]$ a=(1 2 3 4 5 6 7 8)
[root@bastion-IDC ~]$ unset a
[root@bastion-IDC ~]$ echo ${a[*]}
[root@bastion-IDC ~]$ a=(1 2 3 4 5 6 7 8)
[root@bastion-IDC ~]$ unset a[1]
[root@bastion-IDC ~]$ echo ${a[*]}
1 3 4 5 6 7 8
[root@bastion-IDC ~]$ echo ${#a[*]}
7

直接通过：unset 数组[下标] 可以清除相应的元素，不带下标，清除整个数据。
```

##3、特殊使用

### 1）分片:

```sh
[root@bastion-IDC ~]$ a=(1 2 3 4 5 6 7 8)
[root@bastion-IDC ~]$ echo ${a[@]:0:3}
1 2 3
[root@bastion-IDC ~]$ echo ${a[@]:1:4}
2 3 4 5
[root@bastion-IDC ~]$ c=(${a[@]:1:4})
[root@bastion-IDC ~]$ echo ${#c[@]}
4
[root@bastion-IDC ~]$ echo ${c[*]} 
2 3 4 5

直接通过 ${数组名[@或*]:起始位置:长度} 切片原先数组，返回是字符串，中间用“空格”分开，因此如果加上”()”，将得到切片数组，上面例子：c 就是一个新数据。
```

### 2）替换:

```shell
[root@bastion-IDC ~]$ a=(1 2 3 4 5 6 7 8)
[root@bastion-IDC ~]$ echo ${a[@]/3/100}
1 2 100 4 5 6 7 8
[root@bastion-IDC ~]$ echo ${a[@]}
1 2 3 4 5 6 7 8
[root@bastion-IDC ~]$ a=(${a[@]/3/100})
[root@bastion-IDC ~]$ echo ${a[@]}
1 2 100 4 5 6 7 8

调用方法是：${数组名[@或*]/查找字符/替换字符} 该操作不会改变原先数组内容，如果需要修改，可以看上面例子，重新定义数据。
```