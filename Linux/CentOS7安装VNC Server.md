```shell
安装图形化支持
不建议安装GNOME Desktop，它会占用大量系统资源，安装完后大约要占用1G左右的空间，而且安装过程也较长。以root权限安装“X Window System”即可
# yum groups install "X Window System"
# yum install gnome-classic-session gnome-terminal nautilus-open-terminal control-center liberation-mono-fonts
修改系统启动级别
# systemctl set-default graphical.target　　#graphical.target相当于level5，multi-user.target相当于level3
安装vncserver
# yum install tigervnc-server
配置vncserver实例
分别配置root用户和st-jun用户，配置略有不同
root用户，服务名是vncserver@:1.service：
# cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
修改拷贝过来的模板配置文件，主要是[Service]部分

# vim /etc/systemd/system/vncserver@\:1.service
[Service]
Type=forking
User=root
# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=-/usr/bin/vncserver -kill %i
ExecStart=/sbin/runuser -l root -c "/usr/bin/vncserver %i"
PIDFile=/root/.vnc/%H%i.pid
ExecStop=-/usr/bin/vncserver -kill %i

st-jun用户，服务名是vncserver@:2.service：
# cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:2.service

# vim /etc/systemd/system/vncserver@\:2.service
[Service]
Type=forking
User=st-jun
# Clean any existing files in /tmp/.X11-unix environment
ExecStartPre=-/usr/bin/vncserver -kill %i
ExecStart=/usr/bin/vncserver %i
PIDFile=/home/st-jun/.vnc/%H%i.pid
ExecStop=-/usr/bin/vncserver -kill %i

普通用户的ExecStart不同于root，加/sbin/runuser则会在启动服务时报以下错误
Job for vncserver@:2.service failed because the control process exited with error code. See "systemctl status vncserver@:2.service" and "journalctl -xe" for details.
设置vncpasswd
# vncpasswd    #root用户实例的vnc密码
# su - st-jun
$ vncpasswd    #普通用户一定要切换到用户自己的环境下
密码设置完成后回到root权限下，启动服务
加载进程，重启服务
# systemctl daemon-reload
# systemctl start vncserver@:1.service
# systemctl start vncserver@:2.service
# systemctl enable vncserver@:1.service    
# systemctl enable vncserver@:2.service    #开机启动
配置系统防火墙
# firewall-cmd --zone=public --add-port=5901/tcp
# firewall-cmd --zone=public --add-port=5902/tcp
# firewall-cmd --reload

```

