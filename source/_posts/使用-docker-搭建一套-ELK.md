---
title: 使用 Docker 搭建一套 ELK
date: 2021-04-12 20:35:44
tags:
 - 技术
 - ELK
thumbnail: https://cdn.axis-studio.org/cp/cp26day231.jpg
---

# 使用 docker 搭建一套 ELK

本文记录从零搭建一套 ELK 的实践过程。

## 环境

 - Ubuntu 18.04.1 LTS 2c4g
 - Docker version 20.10.5

## 安装

整套 ELK 可以使用 [Elastic stack (ELK) on Docker](https://github.com/deviantony/docker-elk) 的 Docker Compose 来安装，非常方便。

```bash
git clone https://github.com/deviantony/docker-elk.git

cd docker-elk

docker-compose up
```
启动完，你可以执行 `docker logs -f <container_name>` 查看对应容器的日志，比如 `docker logs -f docker-elk_logstash_1`。不出意外的话，可以看到你的 kibana 服务已经跑在 `<your ip>:5601` 上了。

## 导入数据

服务已经起来了，这时候我们需要将数据导入到 es 里。这里我们使用 logstash 的 pipe 来导入数据。

以导入 `mysql` 单表数据作为例子。

首先需要下载 `mysql-connector-java-8.0.23.jar` 文件，放入到 `/usr/share/logstash/logstash-core/lib/jars/` 里。

然后修改 logstash.conf。

```bash
vim logstash/pipeline/logstash.conf
```

修改默认配置为：

```conf
input {
  beats {
    port => 5044
  }

  jdbc {
    type => "rds"
    jdbc_connection_string => "jdbc:mysql://<your_mysql_server_url>:<port>/<your_db>"
    jdbc_user => ""
    jdbc_password => ""
    jdbc_validate_connection => true
    jdbc_driver_library => "/usr/share/logstash/logstash-core/lib/jars/mysql-connector-java-8.0.23.jar"
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    statement => "SELECT * FROM <your_table>"
  }
}

output {
  elasticsearch {
    hosts => "elasticsearch:9200"
    index => "skdemo-test-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "changeme"
  }
}

```

重启 logstash 服务：

```
docker-compose restart logstash
```

查看 logstash 日志应该就能看到 sql 语句被执行了。

此时回到页面 `/app/management/kibana/indexPatterns` 为导入的数据创建索引模式。

然后到 `/app/visualize` 就可以创建可视化图表了。

到这里为止就是这两天搭建 ELK 的成果。成功起了服务，成功导入数据，成功创建一张图表。

接下来要尝试：

- 多数据库增量同步数据
- 原有前端业务接入 ELK 可视化图表


## 本文参考

- [Elastic stack (ELK) on Docker](https://github.com/deviantony/docker-elk)
- [Logstash：把 MySQL 数据导入到 Elasticsearch 中](https://blog.csdn.net/UbuntuTouch/article/details/101691238?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161820255716780274179719%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=161820255716780274179719&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_v2~rank_v29-3-101691238.pc_v2_rank_blog_default&utm_term=mysql)
- 封图摄于 cp26
