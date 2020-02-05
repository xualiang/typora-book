1. 本地工程的mave设置为阿里云，自动构建，下载jar包
2. 将jar包和pom在服务器后台批量上传到nexus的私服库中
3. 在nexus管理界面上update index并检查上传的内容是否已经存在
4. 删除本地repo中的jar包和pom，重新构建



此外，还可以在nexus中设置代理，自动下载公网上的资源。