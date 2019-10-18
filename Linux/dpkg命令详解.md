语法
    dpkg (选项) (参数)
选项
    -i            安装软件包；
    -r            删除软件包；
    -P            删除软件包的同时删除其配置文件；
    -L            显示于软件包关联的文件；
    -l            显示已安装软件包列表；
    --unpack        解开软件包；
    -c            显示软件包内文件列表；
    --confiugre        配置软件包。
参数
    Deb软件包：指定要操作的.deb软件包
例证
    dpkg -i         package.deb         #安装包
    dpkg -r            package             #删除包
    dpkg -P         package             #删除包（包括配置文件）
    dpkg -L         package             #列出与该包关联的文件
    dpkg -l         package                #显示该包的版本
    dpkg --unpack         package.deb          #解开deb包的内容
    dpkg -S         keyword                #搜索所属的包内容
    dpkg -l                                #列出当前已安装的包
    dpkg -c         package.deb            #列出deb包的内容
    dpkg --configure     package           #配置包
指定安装路径（安装.deb软件到其他目录）
    
    sudo dpkg -i --instdir=/opt/apache apache2
    然后可以建立一个软链接
        ln -s /opt/gsopcast/usr/local/bin/gsopcast  /usr/local/bin

用法归纳   

dpkg是一个Debian的一个命令行工具，它可以用来安装、删除、构建和管理Debian的软件包。
下面是它的一些命令解释：
1）安装软件
命令行：dpkg -i <.deb file name>
示例：dpkg -i avg71flm_r28-1_i386.deb
2）安装一个目录下面所有的软件包
命令行：dpkg -R
示例：dpkg -i -R /usr/local/src
3）释放软件包，但是不进行配置
命令行：dpkg –unpack package_file 如果和-R一起使用，参数可以是一个目录
示例：dpkg –unpack avg71flm_r28-1_i386.deb
4）重新配置和释放软件包
命令行：dpkg –configure package_file
如果和-a一起使用，将配置所有没有配置的软件包
示例：dpkg –configure avg71flm_r28-1_i386.deb
5）删除软件包（保留其配置信息）
命令行：dpkg -r
示例：dpkg -r avg71flm
6）替代软件包的信息
命令行：dpkg –update-avail <Packages-file>
7）合并软件包信息
dpkg –merge-avail <Packages-file>
8）从软件包里面读取软件的信息
命令行：dpkg -A package_file
9）删除一个包（包括配置信息）
命令行：dpkg -P
10）丢失所有的Uninstall的软件包信息
命令行：dpkg –forget-old-unavail
11）删除软件包的Avaliable信息
命令行：dpkg –clear-avail
12）查找只有部分安装的软件包信息
命令行：dpkg -C
13）比较同一个包的不同版本之间的差别
命令行：dpkg –compare-versions ver1 op ver2
14）显示帮助信息
命令行：dpkg –help
15）显示dpkg的Licence
命令行：dpkg –licence (or) dpkg –license
16）显示dpkg的版本号
命令行：dpkg –version
17）建立一个deb文件
命令行：dpkg -b direc×y [filename]
18）显示一个Deb文件的目录
命令行：dpkg -c filename
19）显示一个Deb的说明
命令行：dpkg -I filename [control-file]
20）搜索Deb包
命令行：dpkg -l package-name-pattern
示例：dpkg -I vim
21)显示所有已经安装的Deb包，同时显示版本号以及简短说明
命令行：dpkg -l
22）报告指定包的状态信息
命令行：dpkg -s package-name
示例：dpkg -s ssh
23）显示一个包安装到系统里面的文件目录信息
命令行：dpkg -L package-Name
示例：dpkg -L apache2
24）搜索指定包里面的文件（模糊查询）
命令行：dpkg -S filename-search-pattern
25）显示包的具体信息
命令行：dpkg -p package-name
示例：dpkg -p cacti