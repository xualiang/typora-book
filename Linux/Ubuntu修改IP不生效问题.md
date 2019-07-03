**1. Ubuntu Server 16.04修改IP**

```bash
sudo vi /etc/network/interfaces
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto eth0
iface eth0 inet static
        address 192.168.0.88
        netmask 255.255.255.0
        network 192.168.0.0
        broadcast 192.168.0.255
        gateway 192.168.0.1
        # dns-* options are implemented by the resolvconf package, if instatlled
        dns-nameservers 192.168.0.1
        dns-search pcat
```

```
sudo /etc/init.d/networking restart
```

ubuntu还有别的重启方式（但不一定有效）

```
sudo service network restart
```

desktop版可以这样重启：

```
sudo service network-manager restart
```

 如果只是修改了某个网卡(例如eth0)的信息，也可以通过重启网卡的方式使其修改生效。

```
sudo ifdown eth0
sudo ifup eth0
```

 但是按照以上方法设置后，**ifconfig里显示的还是旧ip**，而且新旧两个ip都可以ping通，只有重启机子后才会显示新的ip。

**解决办法**：

```bash
ip addr flush dev eth0
ifdown eth0
ifup eth0
```

 