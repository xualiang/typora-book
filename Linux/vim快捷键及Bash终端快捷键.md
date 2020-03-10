## Vim快捷键

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

向右移动3个单词：3e           向左移动3个单词：3b

```

## Bash终端快捷键

- Alt + B：左移一个单词   Alt + F：右移一个单词
- ESC + B: 同上                ESC + F：同上
- Ctrl + A：到行首             Ctrl + E：到行尾
- Ctrl + U：剪切到行首     Ctrl + K：剪切到行尾     Ctrl + W：剪切一个单词
- Ctrl + Y：在当前光标处粘贴Ctrl+U、Ctrl+K、Ctrl+W剪切的内容
- Ctrl + R：搜索历史命令
- Ctrl + L：清屏
- Ctrl + Z：将正在运行的程序放到后台
- Ctrl + D：关闭终端