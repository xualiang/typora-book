## 安装

1. 挂载cpu2017镜像，并拷贝至另一个目录

   ```sh
   mount cpu2017-1.0.2.iso /spec
   cp -r /spec /homme/xucl/cpu2017
   ```

2. 解压linux-mips64el-1.0.5.tar至cpu2017主目录：

   ```sh
   tar xf linux-mips64el-1.0.5.tar -C /homme/xucl/cpu2017/
   ```

3. 安装：

   ```sh
   cd /home/xucl/cpu2017
   ./install.sh
   ```

4. 将tools-linux-mips64el.tar.xz解压到cpu2017主目录：

   ```sh
   cd /homme/xucl/cpu2017/tools/bin/linux-mips64el
   tar xf tools-linux-mips64el.tar.xz -C /homme/xucl/cpu2017
   ```

5. 上传龙芯提供的配置文件gcc48-linux-mips64el-16-core-rc1.0.cfg到config目录，并重命名：

   ```sh
   mv gcc48-linux-mips64el-16-core-rc1.0.cfg loongson.cfg
   # 将配置文件中的关键字gs464e修改为loongson3a
   ```

6. 执行：

   ```sh
   source /home/xucl/cpu2017/shrc
   ```

7. 安装依赖：

   ```sh
   yum install gcc gcc-c++ gcc-gfortran.mips64el libstdc++-devel libgfortran
   # 可能只需要安装gcc、gcc-c++、gfortran
   ```

# 测试

```sh
runcpu -c loongson.cfg -T base -n 1 -C 8 -I -i ref all
```



