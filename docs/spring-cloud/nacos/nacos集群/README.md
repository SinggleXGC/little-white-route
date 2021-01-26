# nacos集群

## 部署集群架构图

![deployDnsVipMode.jpg](./img/1561258986171-4ddec33c-a632-4ec3-bfff-7ef4ffc33fb9.jpeg)



## 持久化

默认Nacos使用嵌入式数据库实现数据的存储。所以，如果启动多个默认配置下的nacos节点，数据存储是存在一致性问题的。

为了解决这个问题，nacos采用了集中式存储的方式来支持集群化部署，目前只支持

MySQL的存储。



### 持久化配置

nacos默认自带的是嵌入式数据库derby，切换成mysql如下

1. 在nacos\conf目录下找到sql脚本，执行

2. nacos\conf目录下找到application.properties，增加数据库配置

   ```cnf
   ### Count of DB:
   db.num=1
   
   ### Connect URL of DB:
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
   db.user.0=root
   db.password.0=123456
   ```




## 集群部署配置

完成3333,4444,5555三个nacos的集群部署，通过nginx反向代理实现8848端口向外提供集群服务。



### 1.nacos的持久化配置

按上面持久化配置完成



### 2.nacos的集群配置cluster.conf修改

```cnd
123.57.159.15:3333
123.57.159.15:4444
123.57.159.15:5555
```



### 3.修改nacos启动脚本startup.sh，使其能够接收不同的启动端口

**第一处修改**

修改前

```sh
while getopts ":m:f:s:" opt
do
    case $opt in
        m)
            MODE=$OPTARG;;
        f)
            FUNCTION_MODE=$OPTARG;;
        s)
            SERVER=$OPTARG;;
        ?)
        echo "Unknown parameter"
        exit 1;;
    esac
done
```

修改后

```sh
while getopts ":m:f:s:p:" opt
do
    case $opt in
        m)
            MODE=$OPTARG;;
        f)
            FUNCTION_MODE=$OPTARG;;
        s)
            SERVER=$OPTARG;;
		p)
			PORT=$OPTARG;;
        ?)
        echo "Unknown parameter"
        exit 1;;
    esac
done
```



**第二处修改**

修改前

```sh
nohup $JAVA ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
```

修改后

```sh
nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
```



启动方式为`./startup.sh -p 3333`来指定端口号



### 4.nginx配置

```conf
upstream cluster {
	server 127.0.0.1:3333;
	server 127.0.0.1:4444;
	server 127.0.0.1:5555;
}

server {
	listen 8848;
	server_name localhost;
	location / {
		proxy_pass http://cluster;
	}
}

```



### 测试

1. 启动三个nacos服务
2. 启动nginx
3. 输入`123.57.159.15:8848/nacos`，出现nacos网址，即代表配置成功