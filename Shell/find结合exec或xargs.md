```sh
find /opt/kafka -maxdepth 10 -mindepth 1 -name "*.sh" -type f -exec grep -iH "Xmx1G" {} \;
```

## 查询当前目录下所有包含nexus关键字的文件

```sh
find ./ -type f | xargs grep -H nexus
```

