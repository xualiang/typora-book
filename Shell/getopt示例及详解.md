在[shell脚本之shift和getopts](http://www.361way.com/shell-shift-getopts/4973.html)篇中有提到getopts，除了bash自带的内部变量getopts外，util-linux-ng包还提供了一个工具getopt ，该工具较bash内置的getopts更强大，其不仅支持短参-s，还支持--longopt的长参数，甚至支持-longopt的简化参数。相较于getopts ，getopts 不但支持长短选项，其还支持选项和参数放在一起写。

### 一、getopt命令的用法 

```sh
getopt [options] -o|--options optstring [options] [--] parameters
```

选项说明：

-a：使getopt长参数支持"-"符号打头，必须与-l同时使用

-l：后面接getopt支持长参数列表

-n program：如果getopt处理参数返回错误，会指出是谁处理的这个错误，这个在调用多个脚本时，很有用

-o：后面接短参数列表，这种用法与getopts类似

-u：不给参数列表加引号，默认是加引号的（不使用-u选项），例如在加不引号的时候 --longoption "arg1 arg2" ，只会取到"arg1"，而不是完整的"arg1 arg2"

其有两种使用方法，如下

方法1：

```sh
ARGV=($(getopt -o 短选项1[:]短选项2[:]...[:]短选项n -l 长选项1,长选项2,...,长选项n -- "$@"))
for((i = 0; i < ${#ARGV[@]}; i++)) {
    eval opt=${ARGV[$i]}
    case $opt in
    -短选项1|--长选项1)
       process
       ;;
    # 带参数
    -短选项2|--长选项2)
       ((i++));
       eval opt=${ARGV[$i]}
       ;;
    ...
    -短选项n|--长选项n)
       process
       ;;
    --)
       break
       ;;
    esac
}
```

方法2：

```sh
ARGV=($(getopt -o 短选项1[:]短选项2[:]...[:]短选项n -l 长选项1,长选项2,...,长选项n -- "$@"))
eval set -- "$ARGV"
while true
do
    case "$1" in
    -短选项1|--长选项1)
        process
        shift
        ;;
    -短选项2|--长选项2)
        # 获取选项
        opt = $2
        process
        shift 2
        ;;
    ......
    -短选项3|--长选项3)
        process
        ;;
    --)
break
;;
esac
}
```

注意：如果getopt命令本身没有使用-o|--option选项的话，那么--后面的第一个参数被当做短选项。

### 二、示例 

```sh
#!/bin/bash
# A small example program for using the new getopt(1) program.
# This program will only work with bash(1)
# Note that we use `"$@"' to let each command-line parameter expand to a
# separate word. The quotes around `$@' are essential!
# We need TEMP as the `eval set --' would nuke the return value of getopt.
TEMP=`getopt -o ab:c:: --long a-long,b-long:,c-long:: -n 'example.bash' -- "$@"`
if [ $? != 0 ] ; then echo "Terminating..." >&2 ; exit 1 ; fi
# Note the quotes around `$TEMP': they are essential!
eval set -- "$TEMP"
while true ; do
        case "$1" in
                -a|--a-long) echo "Option a" ; shift ;;
                -b|--b-long) echo "Option b, argument \`$2'" ; shift 2 ;;
                -c|--c-long)
                        # c has an optional argument. As we are in quoted mode,
                        # an empty parameter will be generated if its optional
                        # argument is not found.
                        case "$2" in
                                "") echo "Option c, no argument"; shift 2 ;;
                                *)  echo "Option c, argument \`$2'" ; shift 2 ;;
                        esac ;;
                --) shift ; break ;;
                *) echo "Internal error!" ; exit 1 ;;
        esac
done
echo "Remaining arguments:"
for arg do echo '--> '"\`$arg'" ; done 
```


运行结果如下：

```sh
[root@361way bash]$ sh get.sh -a par1 'another arg' --c-long 'wow!*\?' -cmore -b " very long "
Option a
Option c, no argument
Option c, argument `more'
Option b, argument ` very long '
Remaining arguments:
--> `par1'
--> `another arg'
--> `wow!*\?'
```

使用eval 的目的是为了防止参数中有shell命令，被错误的扩展。

平时使用时，可以使用的样例为：

```sh
ARGS=`getopt -a -o I:D:T:e:k:LMSsth -l instence:,database:,table:,excute:,key:,list,master,slave,status,tableview,help -- "$@"`
[ $? -ne 0 ] && usage
#set -- "${ARGS}"
eval set -- "${ARGS}"
while true
do
        case "$1" in
        -I|--instence)
                instence="$2"
                shift
                ;;
        -D|--database)
                database="$2"
                shift
                ;;
        -T|--table)
                table="$2"
                shift
                ;;
        -e|--excute)
                excute="yes"
                shift
                ;;
        -k|--key)
                key="$2"
                shift
                ;;
        -L|--list)
                LIST="yes"
                ;;
        -M|--master)
                MASTER="yes"
                ;;
        -S|--slave)
                SLAVE="yes"
                ;;
        -A|--alldb)
                ALLDB="yes"
                ;;
        -s|--status)
                STATUS="yes"
                ;;
        -t|--tableview)
                TABLEVIEW="yes"
                ;;
        -h|--help)
                usage
                ;;
        --)
                shift
                break
                ;;
        esac
shift
done 
```