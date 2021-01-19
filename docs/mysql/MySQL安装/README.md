# MySQL安装

## Linux安装(tar包安装)

1. 下载并解压tar包到安装目录

2. 为数据库创建数据存放目录

   ```bash
   #mysql数据文件目录
   mkdir -p /data/mysql
   ```

3. 创建mysql用户组和mysql用户

   ```bash
   groupadd mysql
   useradd -r -g mysql mysql
   ```

4. 将**解压后的mysql**和**数据存放目录**的文件所有权修改为mysql用户组和mysql用户

   ```bash
   chown -R mysql:mysql /usr/local/mysql
   chown -R mysql:mysql /data/mysql
   ```

5. 更改mysql安装文件夹权限

   ```bash
   chmod -R 755 /usr/local/mysql
   ```

6. 安装libaio依赖

   ```bash
   yum install libaio
   ```

7. 初始化mysql命令

   ```bash
   cd /usr/local/mysql/bin
   ./mysqld --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql --initialize
   ```

   在执行上面时特别要注意一行内容：

   ```
   [Note] A temporary password is generated for root@localhost: o*s#gqh)F4Ck
   root@localhost: 后面跟的是mysql数据库登录的临时密码，各人安装生成的临时密码不一样
   #如果初始化时报错如下：
   error while loading shared libraries: libnuma.so.1: cannot open shared objec
   #是因为libnuma安装的是32位，我们这里需要64位的，执行下面语句就可以解决
   yum install numactl.x86_64
   #执行完后重新初始化mysql命令
   ```

8. 启动mysql服务

   ```bash
   sh /usr/local/mysql/support-files/mysql.server start
   ```

   上面启动mysql服务命令是会报错的，因为没有修改mysql的配置文件，报错内容大致如下：

   ```
   ./support-files/mysql.server: line 239: my_print_defaults: command not found
   ./support-files/mysql.server: line 259: cd: /usr/local/mysql: No such file or directory
   Starting MySQL ERROR! Couldn't find MySQL server (/usr/local/mysql/bin/mysqld_safe)
   ```

9. 编辑配置文件my.cnf，添加配置如下

   ```cnf
   [root@localhost bin]#  vim /etc/my.cnf
   
   [mysqld]
   datadir=/data/mysql
   port=3306
   sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
   symbolic-links=0
   max_connections=600
   innodb_file_per_table=1
   lower_case_table_names=1
   character_set_server=utf8
   ```

   `lower_case_table_names`：是否区分大小写，1表示存储时表名为小写，操作时不区分大小写；0表示区分大小写；不能动态设置，修改后，必须重启才能生效：
    `character_set_server`：设置数据库默认字符集，如果不设置默认为latin1
    `innodb_file_per_table`：是否将每个表的数据单独存储，1表示单独存储；0表示关闭独立表空间，可以通过查看数据目录，查看文件结构的区别；

10. 测试启动mysql服务器

    ```bash
    [root@localhost /]# /usr/local/mysql/support-files/mysql.server start
    ```

11. 添加软连接，并重新启动mysql服务

    ```bash
    ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
    ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
    service mysql restart
    ```

12. 登录mysql，修改密码(密码为步骤7生成的临时密码)

    ```bash
    [root@localhost /]#  mysql -u root -p
    Enter password:
    mysql>set password for root@localhost = password('yourpass');
    ```





## 参考资料

[Linux下Mysql安装（tar安装）](https://www.cnblogs.com/jing99/p/9684273.html)

[Linux下安装mysql-5.7.24](https://www.jianshu.com/p/276d59cbc529)