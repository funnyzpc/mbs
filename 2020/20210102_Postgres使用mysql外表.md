
### postgres使用mysql外表

> 转载请注明出处[https://www.cnblogs.com/funnyzpc/p/13952374.html](https://www.cnblogs.com/funnyzpc/p/13952374.html)

#### 浅谈
  &nbsp;&nbsp; `postgres`不知不觉已经升到了版本13,记得两年前还是版本10，当然这中间一直期望着哪天能在项目中使用postgresql，现在已实现哈～；
  顺带说一下：使用`postgresql` 的原因是它的生态完整，还有一个很重要的点儿是 `速度快` 这个在第10版的时 这么说也许还为时过早，
  但是在13这一版本下一点儿也不为过,真的太快了，我简单的用500w的数据做聚合，在不建立索引(主键除外)的情况下 执行一个聚合操作，postgres
  的速度是`mysql`的8倍，真的太快了～；好了，这一章节我就聊一聊我实际碰到的问题，就是：跨库查询，这里是用mysql_fdw实现的。
  
#### 环境准备
+ 一个`mysql`实例(5.7或8均可)
+ 一个`postgres`实例(这里使用源码编译安装的13，建议13，11或12也可)
+ 一台linux（以下内容使用的是`centos`,其它系统也可参考哈）

  以下内容仅仅为安装及使用mysql_fdw的教程，具体mysql及postgres怎么安装我就一并略去

#### 准备libmysqlclient

  注意：若mysql与postgresql在同一台linux机上，则无需安装mysql工具，请略过本段
+ `wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.22-linux-glibc2.17-x86_64-minimal.tar.xz`
+ `tar -xvJf mysql-8.0.22-linux-glibc2.17-x86_64-minimal.tar.xz`
+ ` chown -R mysql:mysql /usr/local/mysql/`
+ `cd mysql-8.0.22-linux-glibc2.17-x86_64-minimal`
+ `cp -r ./* /usr/local/mysql/`

#### 配置环境变量

+ 配置文件

  ```vi /etc/profile```

+ 添加mysql环境变量
  ```
  export MYSQL_HOME=/usr/local/mysql
  export PATH=$PATH:/usr/local/mysql/bin
  export LD_LIBRARY_PATH=MYSQL_HOME/lib:$LD_LIBRARY_PATH
  ```

+ 添加postgres环境变量
  ```
  export PG_HOME=/usr/local/pgsql
  export LD_LIBRARY_PATH=$PG_HOME/lib:$MYSQL_HOME/lib:/lib64:/usr/lib64:/usr/local/lib64:/lib:/usr/lib:/usr/local/lib
  export PATH=$PG_HOME/bin:$MYSQL_HOME/bin:$PATH:.
  ```

+ 刷新配置

  `  source  /etc/profile `

#### 下载并编译mysql_fdw
+ 下载地址: 
 [https://github.com/EnterpriseDB/mysql_fdw/releases](https://github.com/EnterpriseDB/mysql_fdw/releases)
 
+ 解压

  `tar -xzvf REL-2_5_5.tar.gz`
  
+ 进入

  `cd  mysql_fdw-REL-2_5_5`
 
+ 编译 

  `make USE_PGXS=1`
  
+ 安装 

  `make USE_PGXS=1 install`

#### 重启postgres
 
  安装mysql_fdw 并 配置完成环境变量必须重启postgresql,这个很重要
  
  ```
    su postgres
    /usr/local/pgsql/bin/pg_ctl -D /mnt/postgres/data -l logfile stop
    /usr/local/pgsql/bin/pg_ctl -D /mnt/postgres/data -l logfile start
    psql [ or /usr/local/pgsql/bin/psql]
  ```

#### 登录到postgres并配置mysql_server
+ 切换到指定数据库(很重要!!!): `\c YOUR_DB_NAME`
+ `CREATE EXTENSION mysql_fdw;`
+ `CREATE SERVER mysql_server FOREIGN DATA WRAPPER mysql_fdw OPTIONS (host 'HOST', port '3306');`
+ `CREATE USER MAPPING FOR YOUR_DB_NAME SERVER mysql_server OPTIONS  (username 'USERNAME', password 'PASSWORD');`
+ `GRANT USAGE ON FOREIGN SERVER mysql_server TO YOUR_DB_NAME;`
+ `GRANT ALL PRIVILEGES ON ods_tianmao_transaction TO YOUR_DB_NAME;`

#### 创建外表

  创建的外表必须在mysql中有对应的表，否则无法使用(也不会在DB工具中显示)
  
+ 样例

  ```
  CREATE FOREIGN TABLE YOUR_TABLE_NAME(
    id  numeric(22),
    date date ,
    name varchar(50),
    create_time timestamp 
  )SERVER mysql_server OPTIONS (dbname 'YOUR_DB_NAME', table_name 'MYSQL_TABLE_NAME');
  ```

#### 删除操作
+ 删除扩展 

  `DROP EXTENSION mysql_fdw CASCADE;`

+ 删除mysql_server 

  `DROP SERVER [mysql_server] CASCADE;`

+ 删除外表

  `DROP FOREIGN TABLE [YOUR_FOREIGN_TABLE_NAME] CASCADE;`

+ 修改user mapping
  ```
  ALTER USER MAPPING FOR YOUR_DB_USER SERVER mysql_server OPTIONS (SET password 'PASSWORD');
  ALTER USER MAPPING FOR YOUR_DB_USER SERVER mysql_server OPTIONS (SET username 'USERNAME');
  ```

#### 最后

  &nbsp;&nbsp;想说的是postgresql的外表功能实在是太好用了，建立mysql外表后可直接在posgresql中执行增删改查等操作
  更强大的是 还可以执行与postgresql表的连表查询，真香~，省去了应用配置数据源的麻烦。