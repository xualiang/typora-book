### 添加删除忽略文件都是对.gitignore进行编辑，对于ignore的语法示例：

\# 用'#'开始注释一行. 

\# 忽略掉所有文件名是 foo.txt 的文件.
 foo.txt

\# 忽略所有生成的 html 文件,
 *.html
 \# foo.html是个特例，不添加忽略.
 !foo.html

\# 忽略所有.o和 .a文件，出了special.o文件.
 *.[oa] 
 !special.o

\# 只忽略当前目录下的folder文件和目录，子目录的folder不在忽略范围内
 /folder

\# 只忽略test文件，不忽略test目录(比如当前目录下有test文件夹、test.jpg、test.png等)
 test
 !test/

\# 只忽略test目录，不忽略test文件
 test/

\# 忽略test文件和test目录
 test

**添加、移除忽略文件不生效怎么回事**

辛辛苦苦编辑完忽略规则，却发现设置的规则完全没起作用，或者有的起作用了，有的没起作用，怎么回事？原因是.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。解决方法就是先把要忽略文件的本地缓存删除（改成未track状态），然后再修改提交.gitignore文件：

git rm -r --cached .idea/modules.xml
 git add .gitignore
 git commit -m 'update .gitignore'

关于忽略可以参考：[https://github.com/github/gitignore](https://link.jianshu.com?t=https:/github.com/github/gitignore) 来设置忽略规则。