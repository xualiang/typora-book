## Jenkins中安装Gitlab Plugin

不再多说，请见我之前的文档[打造自己的jenkins docker镜像](./打造自己的jenkins docker镜像.md)

## Jenkins中配置Gitlab地址

系统管理->系统配置

<img src="../images/image-20200319114135831.png" alt="image-20200319114135831" style="zoom:30%;" />

点击“添加”，添加凭据：

<img src="../images/image-20200319114638343.png" alt="image-20200319114638343" style="zoom:30%;" />

上图中的API token需要在Gitlab中提前生成好，方法如下：

<img src="../images/image-20200319131642835.png" alt="image-20200319131642835" style="zoom:30%;" />

## Jenkins中的项目启用Gitlab webhook

![image-20200319135247060](../images/image-20200319135247060.png)

点击“高级”，然后点击“Generate”生成token，如下：

![image-20200319135139300](../images/image-20200319135139300.png)

## Gitlab中创建webhook

![image-20200319135655130](../images/image-20200319135655130.png)

如果有报错：Urlis blocked: Requests to the local network are not allowed，这是因为新版的gitlab为了安全默认禁止了本地局域网地址调用web hook，需要使用admin账号登录gitlab，进行如下设置：

![image-20200319140014552](../images/image-20200319140014552.png)

![image-20200319140035128](../images/image-20200319140035128.png)

