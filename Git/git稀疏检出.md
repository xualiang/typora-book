# Git只获取部分目录的内容（稀疏检出）

Git在1.7版本后，已经支持只Checkout部分内容，这个功能叫做 sparse checkout（稀疏检出）。

## 打开 sparse checkout 功能

如果本地还没有建版本库，要用这个功能，先进入要放版本库的目录，在命令行执行几条命令：

```sh
$ git init <project>
$ cd <project>
$ git remote add origin ssh://<user>@<repository's url>
$ git config core.sparsecheckout true
$ echo "path1/" >> .git/info/sparse-checkout
$ echo "path2/" >> .git/info/sparse-checkout
$ git pull origin master
```



第一条命令`git init <project>`，先建立一个空的版本库，用实际的目录名替代。
第二条命令`cd <project>`，进入创建的新的版本库的目录。
第三条命令`git remote add origin ssh://<user>@<repository's url>`，添加远程库的地址。
第四条命令`git config core.sparsecheckout true`，打开sparse checkout功能。
第五第六条命令`echo "path1/" >> .git/info/sparse-checkout`，添加2个目录到checkout的列表。路径是版本库下的相对路径，也可以用文本编辑器编辑这个文件。
第七条命令`git pull origin master`，拉取远程的 master 分支，也可以拉其他分支。

如果只拉取最近一次的变更，忽略以前的变更记录，在拉取时可以加参数depth，如`git pull --depth=1 origin master` （浅克隆）

如果以后修改了 .git/info/sparse-checkout，增加或删除部分目录，可以执行如下命令重新Checkout

```
$ git checkout master
```

或执行以下命令：

```sh
$ git read-tree -mu HEAD
```



如果本地已经建了版本库，要使用这个功能，可以进入版本库的目录，执行以下命令

```shell
$ git config core.sparsecheckout true
$ echo "path1/" >> .git/info/sparse-checkout
$ echo "path2/" >> .git/info/sparse-checkout
$ git checkout master
```



要关闭 sparse checkout功 能，仅仅修改设置，将core.sparsecheckout设为false是不生效的，需要修改 .git/info/sparse-checkout 文件，用一个”*“号替代其中的内容，然后执行 checkout 或 read-tree 命令。

## sparse-checkout 文件设置

子目录的匹配
在 sparse-checkout 文件中，如果目录名称前带斜杠，如`/docs/`，将只匹配项目根目录下的docs目录，如果目录名称前不带斜杠，如`docs/`，其他目录下如果也有这个名称的目录，如`test/docs/`也能被匹配。
而如果写了多级目录，如`docs/05/`，则不管前面是否带有斜杠，都只匹配项目根目录下的目录，如`test/docs/05/`不能被匹配。

通配符 “*“ (星号)
在 sparse-checkout 文件中，支持通配符 “*“，如可以写成以下格式：

```
*docs/
index.*
*.gif
```



排除项 “!” (感叹号)
在 sparse-checkout 文件中，也支持排除项 “!”，如只想排除排除项目下的 “docs” 目录，可以按如下格式写：

```
/*
!/docs/
```



要注意一点：如果要关闭sparsecheckout功能，全取整个项目库，可以写一个”*“号，但如果有排除项，必须写”/*“，同时排除项要写在通配符后面。