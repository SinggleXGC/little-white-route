# Linux安装JDK

1. 去官网下载JDK压缩包
   
   ```bash
   wget https://repo.huaweicloud.com/java/jdk/8u172-b11/jdk-8u172-linux-x64.tar.gz
   ```

2. 解压JDK压缩包

   ```bash
   tar -zxvf jdk-8u192-linux-x64.tar.gz 
   ```

3. 将解压后的文件夹移动到要安装的目录

4. 配置环境变量

   在`/etc/profile`文件最后增加下面内容(下面的目录填写自己的实际目录)

   ```
   export JAVA_HOME=/opt/jdk1.8
   export PATH=$JAVA_HOME/bin:$PATH
   export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   ```

5. 更新配置

   ```bash
   source /etc/source
   ```

6. 查看是否安装成功

   ```bash
   java -version
   ```

   