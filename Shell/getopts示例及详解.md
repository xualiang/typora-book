### getopts参数的使用 

先看一个示例：

```sh
#!/bin/bash
echo $*
while getopts ":a:bc:" opt
do
        case $opt in
                a)
                echo $OPTARG
                echo $OPTIND
                ;;
                b)
                echo "b $OPTIND"
                ;;
                c)
                echo "c $OPTIND"
                ;;
                ?)
                echo "error"
                exit 1
        esac
done
echo $OPTIND
shift $(( $OPTIND-1 ))
echo $0
echo $*   
```

上面这个脚本后面传参后执行结果如下：

```sh
$ ./getopts.sh -a 11 -b -c 6
-a 11 -b -c 6
11
3
b 4
c 6
6  
```

为什么会得到上面的结果呢？

`while getopts ":a:bc:" opt` #第一个冒号表示忽略错误；字符后面的冒号表示该选项必须有自己的参数；

$optarg 存储相应选项的参数，如上例中的11、6；

$optind 总是存储原始$*中下一个要处理的元素(不是参数,而是选项,此处是 a,b,c 这三个选项,而不是那些数字,当然数字也是会占有位置的)位置；

optind初值为1，遇到"x"，选项不带参数，optind += 1 ；遇到“x:”，带参数的选项，optarg = argv[optind + 1], optind += 2；遇到“x::”，可选参数，属于#1和#2之一，GNU扩展实现。

上例中-a参数的位置为1，由于后面有参数11 ，后面需要处理的下一个参数-b的位置为3，所认此出输出3；同理-b要处理的下一个参数为-c ，值为4，-c后面有参数+2，后面要处理的参数为位置为6。

使用 getopts 处理参数虽然是方便，但仍然有两个小小的局限：

- 选项参数的格式必须是-d val，而不能是中间没有空格的-dval。
- 所有选项参数必须写在其它参数的前面，因为 getopts 是从命令行前面开始处理，遇到非-开头的参数，或者选项参数结束标记--就中止了，如果中间遇到非选项的命令行参数，后面的选项参数就都取不到了。
- 不支持长选项， 也就是--debug 之类的选项

由getopts为bash内置的一个参数，如果想要使用更强大的功能可以使用系统下的另一个单独命令getopt，这个单独再做一篇来讲述。