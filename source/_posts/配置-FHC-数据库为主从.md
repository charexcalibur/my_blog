---
title: 配置 FHC 数据库为主从
date: 2021-07-03 13:58:26
tags:
 - 技术
 - 数据库
categories:
  - 技术
thumbnail: https://cdn.axis-studio.org/cp28/cp28d1_64.jpg?imageMogr2/quality/50
---

# 配置 FHC 数据库为主从

## 准备

- docker mysql8.0 * 2

将现有的 mysql 作为主库，在之前新购入的机器上面安装 docker mysql 作为从库。

## 开始

1. 进入主库机器写一个配置

   ```bash
    vim mysql-master.cnf

    [mysqld]
    server_id=1
    binlog_format=ROW
    gtid_mode=ON
    enforce-gtid-consistency=true
   ```

2. 将配置文件放入 docker mysql 容器中

    ```bash
    docker cp mysql-master.cnf mysql8.0:/etc/mysql/conf.d
    ```

3. 重启 docker mysql

    ```bash
    docker restart mysql8.0
    ```

4. 进入 mysql，配置从库可访问账号

    ```bash

    docker exec -it master-mysql bash

    # in docker
    mysql -u root -p

    # in mysql
    create user 'slave' identified with mysql_native_password by 'your pwd';

    GRANT REPLICATION SLAVE ON *.* TO 'slave';

    flush privileges;

    show master status\G;

    *************************** 1. row ***************************
    File: binlog.000024
    Position: 2670
    Binlog_Do_DB:
    Binlog_Ignore_DB:
    Executed_Gtid_Set:
    1 row in set (0.00 sec)

    ERROR:
    No query specified
    ```

5. 由于主数据库中本来就有数据，需要先把数据同步到从数据库中保持数据一致。
   1. 首先最将主数据库锁库
   2. `mysqldump -u root > all_dbs.sql`
   3. 将 sql 文件复制到从数据库
   4. 在从数据库恢复数据，进入 mysql `source /tmp/all_dbs.sql`

6. 恢复完数据后，进入从库服务器, 写一个 slave.cnf 并复制到 docker mysql 中

    ```conf
    [mysqld]
    server-id=2
    ```
7. 在从库配置 master 信息
   
  ```bash
  CHANGE MASTER TO
  MASTER_HOST='your master mysql ip',
  MASTER_USER='username',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='inlog.000024',
  Master_Port=3306, # 注意 Master_Port 为数字
  MASTER_LOG_POS=155; # 注意 position 为数字
  ```
8. 执行从库配置 `start slave;`
9. 查看状态 `show slave status\G;`
10. 在主库中添加数据，然后在从中查询，发现数据会同步过来，主从配置成功

## 本文参考

- [Docker部署MySQL 8.0主从](https://blog.csdn.net/u012295261/article/details/88017068)
- 封图摄于 CP28
