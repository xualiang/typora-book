

/etc/udev/rules.d/ 和 /usr/lib/udev/rules.d/是什么关系？需要修改规则时，我们应该修改哪个文件？
另外还有/lib/udev/rules.d/，又是什么？

1. 首先，/lib/udev/rules.d目录仅是一个软连接，指向了/usr/lib/udev/rules.d/目录，两者相同。
2. /usr/lib/udev/rules.d/中包含默认规则文件，请不要编辑这些文件。
3. /etc/udev/rules.d目录是放置自定义规则文件的地方，而且在与/usr/lib/udev/rules.d/目录下存在同名文件时，/etc/udev/rules.d目录下的自定义规则文件具有更高的优先级。
4. 如果要完全禁用一个默认的规则文件时，可以在/etc/udev/rules.d目录下创建一个软连接指向/dev/null，例如禁用默认规则文件/usr/lib/udev/rules.d/80-net-setup-link.rules：
```sh
ln -s /dev/null /etc/udev/rules.d/80-net-setup-link.rules
```