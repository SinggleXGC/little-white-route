# maven安装

## Linux系统下安装

1. 下载maven安装包

   ```bash
   wget https://mirrors.bfsu.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
   ```

2. 解压

3. 配置环境变量

   在/etc/profile文件下添加下面内容

   ```
   export M2_HOME=/opt/apache-maven-3.6.3
   export PATH=$PATH:$M2_HOME/bin
   ```

4. 刷新配置文件

   ```
   source /etc/profile
   ```

5. 验证是否安装成功

   ```bash
   [root@xgc maven]# mvn -version
   Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
   Maven home: /opt/apache-maven-3.6.3
   Java version: 1.8.0_192, vendor: Oracle Corporation, runtime: /opt/jdk1.8.0_192/jre
   Default locale: en_US, platform encoding: UTF-8
   OS name: "linux", version: "3.10.0-862.14.4.el7.x86_64", arch: "amd64", family: "unix"
   ```




## 切换源

修改conf/settings.xml文件。在mirrors元素中增加下面内容

```xml
<!--下面是配置内容-->
<mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
<mirror>
    <id>repo2</id>
    <name>Mirror from Maven Repo2</name>
    <url>http://repo2.maven.org/maven2/</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```



