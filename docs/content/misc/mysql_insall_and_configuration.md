# Ubuntu20.04 中 MySQL8 的安装和配置

### 1. 安装 MySQL 服务

```
# 安装 mysql 服务
sudo apt install mysql-server

# 安装成功后 mysql 将会自动运行, 查看 mysql 状态
sudo service mysql status

/** * /usr/bin/mysqladmin  Ver 8.0.27-0ubuntu0.20.04.1 for Linux on x86_64 ((Ubuntu))
  * Copyright (c) 2000, 2021, Oracle and/or its affiliates.
  * 
  * Oracle is a registered trademark of Oracle Corporation and/or its
  * affiliates. Other names may be trademarks of their respective
  * owners.
  * 
  * Server version          8.0.27-0ubuntu0.20.04.1
  * Protocol version        10
  * Connection              Localhost via UNIX socket
  * UNIX socket             /var/run/mysqld/mysqld.sock
  * Uptime:                 1 hour 9 min 24 sec
  * 
  * Threads: 2  Questions: 150  Slow queries: 0  Opens: 637  Flush tables: 3  Open tables: 150  Queries per second avg: 0.036
  **/

# 若未运行则手动执行
sudo service mysql start

```

### 2. MySQL 安全配置

#### I. 执行此项来配置 mysql root 用户的密码，及其他信息

```
# 启动配置
sudo mysql_secure_installation
```

#### II. 是否执行高等级的密码安全策略

```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?

Press y|Y for Yes, any other key for No:
```

如果不想设置则直接回车跳过该项设置

#### III. 设置 root 账号密码

```
Please set the password for root here.

New password:

Re-enter new password:
```

linux 系统并不会像网页在输入密码时展示 `*`，需要注意别弄错了

#### IV. 移除匿名用户

```
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) :
```

输入 `Y` 并回车可删除匿名用户。不想删除则直接回车

#### V. 禁止 root 用户远程登录

```
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) :
```

为了安全起见，你可以输入 `Y` 并回车，禁止 root 用户远程登录，如果有远程连接的需求则需要添加一个新的用户，步骤见 3

#### VI. 移除 test 数据库

```
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) :
```

一般直接移除，输入 `Y` 并回车。不想删除则直接回车

#### VII. 重新加载权限配置

```
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) :
```

输入 `Y` 并回车以确保上面的修改生效

### 3. 新增用户

#### I. 登录mysql

```
sudo mysql
```

#### II. 创建用户

```
mysql> CREATE USER '{user}'@'{host}' IDENTIFIED BY '{password}';
```

> 修改 `{}` 中的内容为你的信息 例： -`{user}` => `test_user` -`{host}` => `localhost` -`{password}` => `123456` 最终： `CREATE USER 'test_user'@'localhost' IDENTIFIED BY '123456';` 注意1： `{host}` 的内容可以是某个固定的 IP，如：`localhost`或`192.168.8.2`，也可以时任意IP，如： `%` 注意2： 此时的用户并不能远程访问 mysql，需要执行第三步

#### III. 添加远程访问权限

```
grant all on {database}.{table} to '{user}'@'{localhost}';
```

> 修改 `{}` 中的内容为你的信息 例： - `{database}` => `test_db` -`{table}` => `*` -`{user}` => `test_user` -`{localhost}` => `%` 最终： `grant all on test_db.* to 'test_user'@'%';` 注意： `{database}` 和 `{table}` 的内容可以是某个固定值，也可以时任意值 `*`