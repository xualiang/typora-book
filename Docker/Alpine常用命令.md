```markdown
一：Alpine Linux开启SSH远程登陆
1.简介：
最重要的一个服务了，远程登陆需要用它，文件传输需要用它，必备功能。不管你是在实体机上跑，虚拟机上跑，docker里面跑，这个都是必须的。
2.配置
配置文件位置：/etc/ssh/sshd_config
配置文件选项：#PermitRootLogin prohibit-password
修改为：PermitRootLogin yes
3.配置命令
看不懂上面的，直接用下面这句。
sed -i "s/#PermitRootLogin.*/PermitRootLogin yes/g" /etc/ssh/sshd_config
4.重启服务
改了配置不会直接生效，需要重启服务器或者服务。
重启服务器：reboot
重启服务：rc-service sshd restart
二：Alpine Linux源管理
1.简介
源这个概念在linux早就存在了，其实就是类似于软件市场的存在，apple在iphone上发扬光大了，并且自己管理安全的软件，使得iphone上软件兼容性等等问题得到改善，用户体验比较好，android基于linux核心开发，也有了软件市场，最著名的就是google市场，因为被墙，所以国内各个大软件厂商也都有了自己的市场。
每个市场（源）都有自己的服务器，linux默认的都是外国的服务器，我们访问比较慢，所以就有了镜像服务器放在国内，让我们访问快一些。管理源，就是增加镜像服务器。
而且，linux因为是大众维护更新代码，所以还区分了稳定版，测试版……各种版本的市场，这些都需要进行源管理。
2.国内源简介：
这几个都有alpine的源
清华大学：https://mirror.tuna.tsinghua.edu.cn/alpine/
阿里云：https://mirrors.aliyun.com/alpine/
中科大：http://mirrors.ustc.edu.cn/alpine/
还有一些没有alpine的
网易：http://mirrors.163.com/
3.配置：
直接抄中科大的帮助http://mirrors.ustc.edu.cn/help/alpine.html
一般情况下，将 /etc/apk/repositories 文件中 Alpine 默认的源地址 http://dl-cdn.alpinelinux.org/ 替换为 http://mirrors.ustc.edu.cn/ 即可。
可以使用如下命令：
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
也可以直接编辑 /etc/apk/repositories 文件。以下是 v3.5 版本的参考配置：
https://mirrors.ustc.edu.cn/alpine/v3.5/main
https://mirrors.ustc.edu.cn/alpine/v3.5/community
也可以使用 latest-stable 指向最新的稳定版本：
https://mirrors.ustc.edu.cn/alpine/latest-stable/main
https://mirrors.ustc.edu.cn/alpine/latest-stable/community
更改完 /etc/apk/repositories 文件后请运行 apk update 更新索引以生效。
三：Alpine Linux 包管理
1.简介
Alpine使用apk进行包管理，下面介绍常用命令
2.apk update
$ apk update #更新最新镜像源列表
3.apk search
$ apk search #查找所以可用软件包
$ apk search -v #查找所以可用软件包及其描述内容
$ apk search -v 'acf*' #通过软件包名称查找软件包
$ apk search -v -d 'docker' #通过描述文件查找特定的软件包

4.apk add
$ apk add openssh #安装一个软件
$ apk add openssh openntp vim   #安装多个软件
$ apk add --no-cache mysql-client  #不使用本地镜像源缓存，相当于先执行update，再执行add

5.apk info
$ apk info #列出所有已安装的软件包
$ apk info -a zlib #显示完整的软件包信息
$ apk info --who-owns /sbin/lbu #显示指定文件属于的包

6.apk upgrade
$ apk upgrade #升级所有软件
$ apk upgrade openssh #升级指定软件
$ apk upgrade openssh openntp vim   #升级多个软件
$ apk add --upgrade busybox #指定升级部分软件包
7.apk del
$ apk del openssh  #删除一个软件
 四：Alpine Linux服务管理
1.简介
alpine没有使用fedora的systemctl来进行服务管理，使用的是RC系列命令
2.rc-update
rc-update主要用于不同运行级增加或者删除服务。

alpine:~# rc-update --help
Usage: rc-update [options] add <service> [<runlevel>...]
   or: rc-update [options] del <service> [<runlevel>...]
   or: rc-update [options] [show [<runlevel>...]]
Options: [ asuChqVv ]
  -a, --all                         Process all runlevels
  -s, --stack                       Stack a runlevel instead of a service
  -u, --update                      Force an update of the dependency tree
  -h, --help                        Display this help output
  -C, --nocolor                     Disable color output
  -V, --version                     Display software version
  -v, --verbose                     Run verbosely
  -q, --quiet                       Run quietly (repeat to suppress errors)

 
3.rc-status
rc-status 主要用于运行级的状态管理。

alpine:~# rc-status --help
Usage: rc-status [options] <runlevel>...
   or: rc-status [options] [-a | -c | -l | -m | -r | -s | -u]
Options: [ aclmrsuChqVv ]
  -a, --all                         Show services from all run levels
  -c, --crashed                     Show crashed services
  -l, --list                        Show list of run levels
  -m, --manual                      Show manually started services
  -r, --runlevel                    Show the name of the current runlevel
  -s, --servicelist                 Show service list
  -u, --unused                      Show services not assigned to any runlevel
  -h, --help                        Display this help output
  -C, --nocolor                     Disable color output
  -V, --version                     Display software version
  -v, --verbose                     Run verbosely
  -q, --quiet                       Run quietly (repeat to suppress errors)

 
4.rc-service
rc-service主用于管理服务的状态

alpine:~# rc-service --help
Usage: rc-service [options] [-i] <service> <cmd>...
   or: rc-service [options] -e <service>
   or: rc-service [options] -l
   or: rc-service [options] -r <service>
Options: [ ce:ilr:INChqVv ]
  -e, --exists <arg>                tests if the service exists or not
  -c, --ifcrashed                   if the service is crashed then run the command
  -i, --ifexists                    if the service exists then run the command
  -I, --ifinactive                  if the service is inactive then run the command
  -N, --ifnotstarted                if the service is not started then run the command
  -l, --list                        list all available services
  -r, --resolve <arg>               resolve the service name to an init script
  -h, --help                        Display this help output
  -C, --nocolor                     Disable color output
  -V, --version                     Display software version
  -v, --verbose                     Run verbosely
  -q, --quiet                       Run quietly (repeat to suppress errors)

 
5.openrc
openrc主要用于管理不同的运行级。

alpine:~# openrc --help
Usage: openrc [options] [<runlevel>]
Options: [ a:no:s:SChqVv ]
  -n, --no-stop                     do not stop any services
  -o, --override <arg>              override the next runlevel to change into
                                    when leaving single user or boot runlevels
  -s, --service <arg>               runs the service specified with the rest
                                    of the arguments
  -S, --sys                         output the RC system type, if any
  -h, --help                        Display this help output
  -C, --nocolor                     Disable color output
  -V, --version                     Display software version
  -v, --verbose                     Run verbosely
  -q, --quiet                       Run quietly (repeat to suppress errors)

6.我常用的RC系列命令
1.增加服务到系统启动时运行，下例为docker
rc-update add docker boot
2.重启网络服务
rc-service networking restart
3.列出所有服务
rc-status -a
五：关机重启
$ reboot #重启系统
$ poweroff #关机

```

