### 导出版本库dev

```sh
svnadmin dump /software/svnrepo/dev > /tmp/dev.dump
```



### 在目标服务器创建版本库

```sh
svnadmin create /software/svnrepo/dev
```



### 导入版本库dev

```sh
svnadmin load /software/svnrepo/dev < /tmp/dev.dump
```



### 启动svn服务

```sh
svnserve -d -r /software/svnrepo
```

