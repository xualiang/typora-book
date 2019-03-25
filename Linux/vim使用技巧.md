```markdown
在vi中批量替换
:1,$s/old/new/g
我们知道%等价于1，$，所以上行命令也可以这样写：
：% s / old / new / g


跳至行首：ESC后，输入0
跳至行尾：ESC后，输入$

撤销：ESC后，输入u
恢复：ESC后，输入ctl+r


翻页：ctl+f前翻（forward）    ctl+b后翻（background）

```

