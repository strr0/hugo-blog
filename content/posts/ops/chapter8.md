---
title: "Linux 环境下 SeaTunnel 的安装与使用"
date: 2023-10-17T10:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

### 介绍

SeaTunnel 是一个非常易于使用，超高性能的分布式数据集成平台，支持实时性海量数据同步。在已部署 SeaTunnel 的环境下，只需一个作业配置即可根据配置执行作业。

### 1 SeaTunnel 部署

#### 1.1 环境需求

java 环境（java 8 或 java 11）

#### 1.2 下载 SeaTunnel

下载并解压
```bash
wget "https://archive.apache.org/dist/seatunnel/2.3.3/apache-seatunnel-2.3.3-bin.tar.gz"
tar -xzvf "apache-seatunnel-2.3.3-bin.tar.gz"
```

#### 1.3 安装连接器插件

配置所需要的插件，如 jdbc 插件
```sh
# config/plugin_config

--seatunnel-connectors--
connector-jdbc
--end--
```
执行命令安装插件
```bash
sh bin/install-plugin.sh 2.3.3
```

> jdbc 所依赖的驱动程序需要复制到 $SEATNUNNEL_HOME/lib/ 目录下，才能保证作业正常运行

### 2 运行作业

#### 2.1 环境依赖

- 已部署的SeaTunnel

#### 2.2 作业配置（Demo）

```sh
# config/v2.batch.config.template

env {
  execution.parallelism = 1
  job.mode = "BATCH"
}

source {
  FakeSource {
    result_table_name = "fake"
    row.num = 16
    schema = {
      fields {
        name = "string"
        age = "int"
      }
    }
  }
}

transform {
  FieldMapper {
    source_table_name = "fake"
    result_table_name = "fake1"
    field_mapper = {
      age = age
      name = new_name
    }
  }
}

sink {
  Console {
    source_table_name = "fake1"
  }
}
```

#### 2.3 执行作业

运行如下命令执行作业
```bash
cd apache-seatunnel-2.3.3
./bin/seatunnel.sh --config ./config/v2.batch.config.template -e local
```

### 3 验证

#### 3.1 环境准备

- 服务器配置 128 核 512g 内存
- Java 8
- SeaTunnel 2.3.3
- 达梦数据库及测试数据

#### 3.2 驱动配置

将达梦数据库驱动 DmJdbcDriver18.jar 复制到 $SEATNUNNEL_HOME/lib/ 目录下

#### 3.3 作业配置

```sh
# config/v2.batch.config

env {
  execution.parallelism = 10  # 并行数
  job.mode = "BATCH"
}

source {
  Jdbc {
    url = "jdbc:dm://localhost:5236/xxxx"
    driver = "dm.jdbc.driver.DmDriver"
    connection_check_timeout_sec = 100
    user = "xxxx"
    password = "********"
    query = "select * from xxxx.xxxx"
    fetch_size = 10000
    # partition_column = "PART_ID"  # 分区字段
    partition_num = 40  # 分区数
  }
}

#transform {
#}

sink {
  jdbc {
    url = "jdbc:dm://localhost:5236/xxxx"
    driver = "dm.jdbc.driver.DmDriver"
    user = "xxxx"
    password = "********"
    query = "insert into xxxx.xxxx(xxxx,xxxx,xxxx) values(:xxxx,:xxxx,:xxxx)"
  }
}
```

#### 3.4 执行作业

```bash
./bin/seatunnel.sh --config ./config/v2.batch.config -e local
```

#### 3.5 运行结果

| 数据量 | 并行数 | 分区数 |设置分区字段|  耗时  |
| :---: | :---: | :---: | :------: | :---: |
|  800w |    10 |    10 |    否    | 11分钟 |
|  800w |    20 |    20 |    否    | 11分钟 |
|  800w |     4 |     4 |    否    | 11分钟 |
|  800w |     4 |    40 |    否    | 11分钟 |
|  800w |    10 |   100 |    否    | 11分钟 |
|  800w |    10 |    40 |    否    | 10分钟 |
|  800w |    10 |    40 |    是    | 18分钟 |

### 4 SeaTunnel-Web 部署

#### 4.1 环境要求

- 已部署 SeaTunnel
- MySQL

#### 4.2 下载与安装

从 [https://seatunnel.apache.org/download](https://seatunnel.apache.org/download) 下载 seatunnel web，并解压
```bash
tar -zxvf apache-seatunnel-web-1.0.0-SNAPSHOT.tar.gz
```

#### 4.3 初始化数据库（MySQL）

修改 seatunnel_server_env.sh 配置
```sh
# script/seatunnel_server_env.sh

export HOSTNAME="localhost"
export PORT="3306"
export USERNAME="root"
export PASSWORD="123456"
```
运行命令初始化
```bash
sh script/init_sql.sh
```

#### 4.4 配置修改

修改 application.yml 数据库连接配置
```yml
# conf/application.yml

spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/seatunnel?useSSL=false&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&allowPublicKeyRetrieval=true
    username: root
    password: password  # 根据实际情况修改
```
复制 hazelcast-client.yaml 到 conf/
```bash
cp $SEATUNNEL_HOME/config/hazelcast-client.yaml conf/
```
复制 plugin-mapping.properties 到 conf/
```bash
cp $SEATUNNEL_HOME/connectors/plugin-mapping.properties conf/
```
启动 SeaTunnel-Web
```bash
sh bin/seatunnel-backend-daemon.sh start
```
访问 [http://127.0.0.1:8801/ui/](http://127.0.0.1:8801/ui/) 即可进入 web 端（admin/admin）

### 5 问题与修复

#### 5.1 空指针问题

checkpoint 超时导致，可增大 timeout 解决
```yml
# config/seatunnel.yaml

seatunnel:
  engine:
    checkpoint:
      interval: 100000
      timeout: 6000000
```

#### 5.2 数据重复问题

未配置 checkpoint 导致，checkpoint 用于记录数据中间状态，确保数据不会丢失和重复

#### 5.3 作业配置问题

① source 参数和 sink 参数完全一致的情况下可以不使用 transform，sink 中的 insert 语句可以直接使用 ? 占位符

② 使用 transform 导致 ? 占位符匹配参数错位，可以使用 :param（参数名称）来指定参数

③ 作业 sink 参数个数必须与 source 参数个数一致，否者作业会执行失败，可用 transform 过滤
```sh
transform {
  FieldMapper {
    source_table_name = "fake"
    result_table_name = "fake1"
    field_mapper = {
        id = id
        card = card
        name = new_name
    }
  }
}
```

#### 5.4 java.net.BindException: 地址已在使用 (Bind failed)

查看端口是否占用
```bash
netstat -alnp | grep 8801
```
如果端口被占用，执行命令杀死
```bash
kill -9 103436（进程号）
```

#### 5.5 Web 端 404 问题

查看运行日志，可以看到 user does not exists
```bash
tail -f nohup.out

...user does not exists
```
尝试通过 nginx 跳转进入
```sh
# nginx.conf

...
# SeaTunnel-Web
server {
    listen 8808;
    server_name localhost;

    location / {
        proxy_pass http://127.0.0.1:8801;
    }

    location /ui {
        root /linewell/seatunnel/apache-seatunnel-web-1.0.0-SNAPSHOT;
        index index.html index.htm;
    }
}

```
重启 nginx
```bash
/usr/local/nginx/sbin/nginx -s reload -c /usr/local/nginx/conf/nginx.conf
```