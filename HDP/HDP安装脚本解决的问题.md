存在的问题：

1、日志打印不准确（日志显示成功，其实失败，如：JDK、RPM的安装）

2、nohup方式执行互信配置失败  已解决

3、mysql安装不完整，metastore数据库及用户创建，导致hive和pig安装失败. 已解决

4、子节点安装是串行的，效率较低  已解决

5、最后一步所有节点安装mysql时，向其他节点传输rpm有问题  ：已解决

6、/etc/hosts没同步到其他节点  已解决

7、后3个节点ntp未安装成   已解决

8、刚进入页面时，还有一处Ambari Logo没变

9、拷贝OS慢  已解决

10、解决打印太多日志问题，输出到/dev/null   已解决

11、解决必须删除mysql yum源，retry才能成功的问题 已解决

12、nohup方式Ambari setup和启动失败  已解决

13、多考虑脚本retry问题

14、`cat /etc/hosts|awk -F " " 'NR==3 {print$2}'`

15、主节点install HDP，解压过程没有提示，以为卡主

16、自动挂载os iso

umask多次写入etc

脚本中的judage、software目录写死等

NTP



17、即使有失败的步骤，最终还是显示成功了

18、加入操作系统版本检查  已增加

19、考虑重复安装和卸载的问题

20、测试变量不export，是否可以  可以

21、测试source只加到install.sh中是否ekyi  ： 不可以