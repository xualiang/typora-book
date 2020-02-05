makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为 makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令。

makefile带来的好处就是--“自动化编译”,一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。make是一个命令工具，是一个解释makefile中指令的命令工具，一般来说，大多数的IDE都有这个命令，比如：Delphi的make,Visual C++的nmake,Linux下GNU的make.可见，makefile都成为了一种在工程方面的编译方法。

 make

根据Makefile文件编译源代码、连接、生成目标文件、可执行文件。

make clean

清除上次的make命令所产生的object文件（后缀为“.o”的文件）及可执行文件。

make install

将编译成功的可执行文件安装到系统目录中，一般为/usr/local/bin目录。

make dist

产生发布软件包文件（即distribution package）。这个命令将会将可执行文件及相关文件打包成一个tar.gz压缩的文件用来作为发布软件的软件包。

它会在当前目录下生成一个名字类似“PACKAGE-VERSION.tar.gz”的文件。PACKAGE和VERSION,是我们在configure.in中定义的AM_INIT_AUTOMAKE（PACKAGE, VERSION）。

make distcheck

生成发布软件包并对其进行测试检查，以确定发布包的正确性。这个操作将自动把压缩包文件解开，然后执行configure命令，并且执行make,来确认编译不出现错误，最后提示你软件包已经准备好，可以发布了。

make distclean

类似make clean,但同时也将configure生成的文件全部删除掉，包括Makefile文件。