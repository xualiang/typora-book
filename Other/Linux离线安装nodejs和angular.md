## 安装

下载nodejs的离线安装包

在一台能联网的机器上解压nodejs的安装包，使用npm安装angular

将nodejs目录压缩后，拷贝到离线主机上

在离线主机上解压，配置环境变量

node -v  && npm -v && ng version检查

## 配置

npm config set registry http://10.10.70.220:8081/nexus/content/groups/npm/

## 常用命令

npm install

npm install -g --unsafe-perm

npm run-script build（ng build --prod --build）



