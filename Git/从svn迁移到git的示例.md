```sh
创建authors.txt 文件：
文件中必须包含svn所有用户的账号，否则会导致git svn clone命令执行失败，svn的账号和密码文件：/software/svnrepo/dev/conf/passwd
xucl:test-git-to-svn xualiang$ cat authors.txt 
fengdianlong = fengdianlong <fengdianlong@telchina.net>
guanxiangyu = guanxiangyu <guanxiangyu@telchina.net>
jinjiadong = jinjiadong <jinjiadong@telchina.net>
liqg = liqg <liqg@telchina.net>
lishanbao = lishanbao <lishb@telchina.net>
liwenlu = liwenlu <liwenlu@telchina.net>
lizhanq = lizhanq <lizhanq@telchina.net>
wangyuwei = wangyuwei <wangyuwei@telchina.net>
wuyong = wuyong <wuyong@telchina.net>
xucunliang = xucunliang <xucunliang@telchina.net>
yangyong = yangyong <yangyong@telchina.net>
yuguangfan = yuguangfan <yuguangfan@telchina.net>
xingm = xingm <xingm@telchina.net>
lp = lp <lp@163.com>
jiangzhendong = jiangzhendong <jiangzhendong@telchina.net>
aiyongjian = aiyongjian <aiyj@telchina.net>
huanglei = huanglei <huanglei@telchina.net>
xiaoquanqin = xiaoquanqin <xiaoquanqin@telchina.net>
liang = liang <liang@telchina.net>
lipeng = lipeng <lipeng@telchina.net>
caolch = caolch <caolch@telchina.net>

执行git svn clone命令：
git svn clone svn://10.10.50.204/dev/videobd/src/videobds-web --authors-file=authors.txt
git svn clone svn://10.10.50.204/dev/videobd/src/videobds-java --authors-file=authors.txt
git svn clone svn://10.10.50.204/dev/videobd/src/videobds-dispose --authors-file=authors.txt

在gitlab上，使用root用户创建police群组，在police群组下创建3个项目，添加项目组用户并将用户添加到police群组

为每个项目添加远程仓库：
以web项目为例：
cd videobds-web/
git remote add origin http://10.10.50.204:9080/police/videobds-web.git

push到远程仓库：
git push origin master

在gitlab上为每个项目创建devlop分支，并设置为保护分支，禁止所有人删除分支，允许主程序猿和开发人员合并和提交。master分支默认是保护分支，不用设置，查看一下即可。

设置忽略文件

创建tag

修改jenkins持续集成环境，将svn地址改为git地址

```

