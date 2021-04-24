---

layout: post

title:  "后端组件环境搭建"

categories: back-end

tags:  Nginx MySQL Redis ZooKeeper Kafka

---

# 后端组件环境搭建

## Nginx-1.14.0

1. 安装

```
sudo apt install nginx -y
```

2. 启动

```
sudo /etc/init.d/nginx start
```

3. 打开配置文件

```
sudo vim /etc/nginx/nginx.conf
```

4. 配置文件关键参数

```
upstream mihooke {
    # 负载均衡的servers，默认是round robin
    server 192.168.8.243:8080;
    server 192.168.8.243:8090;
}

server {
    listen 80;
    server_name localhost;
    location / {
        proxy_pass http://mihooke;
    }
}
```

5. 检测配置文件和重启服务

```
sudo nginx -t -c /etc/nginx/nginx.conf
sudo /etc/init.d/nginx restart
```


## MySQL-5.7.32

1. 安装

```
sudo apt install -y mysql-client
sudo apt install -y mysql-server
```

2. 查看服务及启动服务

```
ps aux | grep mysql
systemctl start mysqld
systemctl status mysql.service
sudo /etc/init.d/mysql restart
```

3. 修改配置，其他主机也可访问

```
sudo vim /etc/mysql/mysql.conf.d/mysqld.conf
bind-address 改为 0.0.0.0
```

4. 修改权限

```
sudo mysql -u root -p
```

## Redis-5.05

1. 安装

```
sudo apt install -y redis-server
```

2. 修改配置

```
sudo vim /etc/redis/redis.conf
# 增加密码登录
requirepass 行去掉注释，改成自己的密码
# 解禁本机绑定
#bind 127.0.0.1 
```

3. 重启服务

```
sudo /etc/init.d/redis-server restart
```

4. client 登录

```
# -h 主机地址 -p 端口默认6379 -a 密码
redis-cli -h 127.0.0.1 -p 6379 -a lzali
```

> Redis和MySQL的配合使用：在MySQL中进行修改的时候，在此服务层将Redis缓存中的数据清除；用户访问时就会从MySQL中查询，查询结果再放入缓存中


## ZooKeeper-3.7.0

需要JDK环境1.15.0

1. 安装

```
从官网下载压缩文件
tar -zxvf zookeeper-3.7.0.tar.gz

新建zoo.cfg文件
cp conf/zoo_sample.cfg conf/zoo.cfg
```

2. 启动

```
sudo ./bin/zkServer.sh start

# 3.6.2版本依旧会提示JAVA_HOME未设置环境变量的错误
# 可能也会提示zkEnv.sh 语法错误，原因是系统的bash不对

sudo ln -sf bash /bin/sh

再次执行就可以了
```

3. 查看状态

```
sudo ./bin/zkServer.sh status
```


## Kafka-2.13-2.7.0

需要Java 8+环境
此版本已内置了ZooKeeper，不必再自己安装

1.安装
```
tar -xzf kafka_2.13-2.7.0.tgz
cd kafka_2.13-2.7.0
```

2.启动zk服务
```
bin/zookeeper-server-start.sh config/zookeeper.properties
执行完命令窗口需要保持
```

3.启动kafka server
```
新建终端窗口，运行
bin/kafka-server-start.sh config/server.properties
```

