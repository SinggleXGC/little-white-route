# RabbitMQ安装

因为RabbitMQ是基于erlang开发的所以需要先安装erlang.



## 安装erlang

1. 安装依赖项

   ```
   yum install -y epel-release
   ```

2. 添加存储库条目

   ```
   wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
   rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
   ```

3. 安装

   ```
   yum install -y erlang
   ```

4. 验证是否安装成功

   ```
   erl -version
   ```



## 安装RabbitMQ

1. 先下载安装包

2. 因为安装包是tar.xz格式的所以需要用到xz

   ```
   yum install -y xz
   ```

3. 解压安装包

   ```
   xz -d rabbitmq-server-generic-unix-3.8.9.tar.xz
   tar -xvf rabbitmq-server-generic-unix-3.8.9.tar
   mv rabbitmq_server-3.8.9/ /opt/ 
   ```

4. 配置环境变量

   在/etc/profile文件最后加上

   ```
   export PATH=$PATH:/opt/rabbitmq_server-3.8.9/sbin
   ```

5. 刷新环境变量

   ```
   source /etc/profile
   ```



## 常见命令

### 启动

```
rabbitmq-server -detached
```



### 停止

```
rabbitmqctl stop
```



### 状态

```
rabbitmqctl status
```



## web管理

开启web插件

```
rabbitmq-plugins enable rabbitmq_management
```

访问：http://127.0.0.1:15672/

默认账号密码：guest guest（这个账号只允许本机访问）



### 创建一个用户并允许远程访问

1. 添加一个用户

   ```
   rabbitmqctl add_user xgc 123456
   ```

2. 设置用户角色信息，这里设置为超级管理员管理角色（可登陆管理控制台(启用management plugin的情况下)，可查看所有的信息，并且可以对用户，策略(policy)进行操作）

   ```
   rabbitmqctl set_user_tags xgc administrator
   ```

3. 设置用户权限

   ```
   rabbitmqctl set_permissions -p "/" xgc ".*" ".*" ".*"
   ```

   