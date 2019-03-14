1、将ESXI6.5的iso文件挂载到服务器的光驱，一步一步安装完ESXI

2、按键F2设置ESXI的IP、hostname、网关、掩码等，关闭ip6

3、找一台Windows机器（只要能够联通ESXI服务器即可），将Vcenter的安装包上传到Windows，双击vcsa-ui-installer目录下的install，安装过程中指定一台ESXI的服务器即可

 

使用虚拟机模拟VCSA6.5安装的过程中，报错问题：

1、安装进行至80%时，报错：**Task failed on server: Module 'CPUID' power on failed.**

原因：ESXI中嵌套运行ESXI（在ESXI虚拟机上安装Vcenter，因为Vcenter实际上就是一个预装了vcenter的suse虚拟机），需要开启ESXI虚拟机的CPU虚拟化，如下：

![vSphere Web Client  侖 三  写 航 器  上 一 页  0  乙 j10 ． 10 ． 的 89  fhDatacenter  冂 10 · 10 ． ． 185  dockerTestOO  dockerTest01  dockerTest02  dockerTest03  d ． t t ． 刈 1  XUCå-test-esxi2  实 名 认 证 01  冂 10 ． 10 ． ． 186  奉  凸 氵 xucl-test-esxil  入 门 摘 要 监 控  设 置  虚 拟 机 硬 件  虚 拟 机 选 项  虚 拟 机 SDRS 規 则  VApp 选 项  客 尸 机 用 尸 映 射  更 多  冫 0 后 0 操 作 ，  配 置 权 眼  怯 照 数 据 存 储 网 终  虚 拟 机 硬 件  . CPU  使 用 情 况  份 额  预 留  硬 件 虚 拟 化  性 能 计 数 器  内 存  Update Manager  6 个 CPU ， 已 使 用 2914MHz  60 （ 正 常 }  0 MHz  已 用  已 禁 用  16384 MB,  819MB 活 动 内 存 ](../images/775C1B5C-636C-0A4E-BA38-6BABE6B3A835.png)

 

 

解决了问题1后，重新安装vcenter，安装至100%后，报错如下，vcenter的IP10.10.51.30无法ping通

 

![vCenter Server Appliance Installer  Installer  Install - Staae 1: Deolov vCenter Server with an Embedded Platform Services  The installer is unable to connect to the vCenter Appliance Management Interface.  Installer log files are located at  Platform Services Controller.  Deployment complete  Unable to proceed with stage 2 of the deployment process. Click close to exit the installer  You may attempt to continue with stage 2 by logging in to the appliance at https:J/10.10.51.30:5480/  Installer log files are located at ](../images/17417FD8-0E77-4E48-A285-C5B502E26D16.png)

The installer is unable to connect to the vCenter Appliance Management Interface.

Installer log files are located at C:\Users\xualiang\AppData\Local\Temp\vcsaUiInstaller

 

解决方法：登录http://10.10.50.185，网络设置中修改vswitch0的安全模式为混杂模式，之后能够ping通vcenter的IP。属于虚拟机嵌套的网络设置问题。需要设置185的原因：vcenter安装到10.10.51.36ESXI上，而36安装到了185上。如下：

 

![令 C | A 不 安 全 》  ： / ／ 10 ． 10 ． 50 ． 185 / i/#/host/networking/vswitches/vSwitchO  ： ． 颀 用 0 工 作 生 活 学 习  t 专 ： 数 据 库 设 计 " ． 0 MAC 资 源  亡 专 过 ： 全 面 解 析 数 ．  《 0 VSwitctO  到 添 加 上 行 链 路 丿 编 辑 设 置 》 e 刷 新 《 操 作  0 菜 鸟 工 具 ． 不 止 于 ． "  ESXi 、  。 导 航 器  ， 冂 主 机  管 理  虚 拟 机  xucl-test-esxil  更 0 拟 机 ． "  目 存 储  • 目 datastore04  更 存 储 ． "  ， 网 络  ' VM Network  U  0  0  vSwitchO  标 准 vs  类 型 ：  端 二 组 ：  2  上 行 链 路 ：  上 此 0 拟 交 换 机 没 有 上 行 链 路 冗 余 。 您 应 添 加 其 他 上 行 琏 路 适 配 器 。  vSwitch 详 细 倌 息  15 〔 ℃  端 囗  4 佣 2 ( 4074 可 用 )  链 路 岌 现  侦 听 / Ci 00 ove P 们 t 仳 ( CDP )  加 的 虚 拟 机  7 ( 1 活 动 }  信 标 时 间 间 矚  网 卡 绑 定 簾 略  通 知 交 换 机  基 于 源 端 囗 ID 的 路 由  反 向 巭 貉  牧 还 恢 复 ](../images/ABCD2EA3-DEE9-9142-8572-455E142EC055.png)