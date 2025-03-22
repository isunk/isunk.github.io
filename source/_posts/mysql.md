---
title: MySQL
tags: mysql
categories: mysql
---

# MySQL

## 环境搭建

- Linux
    - ~~下载安装包并离线安装  
        `wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-server_5.7.20-1ubuntu14.04_amd64.deb-bundle.tar`  
        `tar -xvf mysql-server_5.7.20-1ubuntu14.04_amd64.deb-bundle.tar`  
        `dpkg -i *.deb`~~
    - 在线安装  
        `apt-get install mysql-server mysql-client libmysqlclient-dev`  
        <font color="grey">安装过程中根据提示输入 mysql 用户中的 root 用户的密码</font>
    > - 启动服务
    >     - 方法一：使用 mysqladmin 启动  
    >         `mysqladmin start`
    >     - 方法二：启动服务  
    >         `/etc/init.d/mysql start`
    > - 重启服务
    >     - 方法一：使用 mysqladmin 重启  
    >         `mysqladmin restart`
    >     - 方法二：<span id="restart-mysqld">重启服务</span>  
    >         `/etc/init.d/mysql restart`
    > - 关闭服务  
    >     `mysqladmin shutdown`

- Windows
    - 离线安装
        1. 下载 [mysql-8.0.13-winx64.zip](https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.13-winx64.zip) 并解压
        2. 初始化 data 目录  
            `./bin/mysqld --initialize-insecure`
        3. 独立（非服务模式）运行  
            `./bin/mysqld --standalone --console`
            > - 安装至系统服务  
            >     `./bin/mysqld --install`
            >    - 启动系统服务  
            >        `net start mysql`    
            > - 删除系统服务  
            >     `./bin/mysqld --remove`  
            >     ~~`./bin/mysqld --remove MySQL`~~
        4. 登录/连接 mysql 控制台  
            `mysql -uroot`

## 控制台（mysql）操作

1. 登录/连接 mysql 控制台环境
    - 场景一：指定的 mysql 用户 root 已设置密码，需以密钥的方式登录  
        ~~`mysql -uroot -p`~~  
        `mysql -u root -p`  
        使用 test 用户登录远程主机 192.168.1.101（使用默认端口号 3306）的 mysql 服务  
        `mysql -h 192.168.1.101 -u test -p`  
        根据提示输入密码，验证通过后进入 mysql 命令行控制台界面中
    - 场景二：登录的 mysql 用户（root）未设置密码  
        `mysql`
    > 参数 "-u" 指定登录用户名称，默认使用 mysql 用户中的 root 用户

2. 操作、命令
    - 执行 sql 语句（参考 [MySql 语法](#mysql-syntax)）
        ```sql
        -- 显示所有数据库
        show databases;

        -- 打开/使用 mysql 数据库（即切换当前所使用的数据库，也可通过 use 其他数据库来直接切换当前所使用的数据库）
        -- 打开之后，不指定数据库名如 "select * from user;" 默认使用当前打开的数据库中的表 user；也可在未打开任何数据库的情况下，直接指定数据库名如 "select * from mysql.user;"。）
        use mysql;
        
        -- 显示当前已打开的数据库中所有表
        show tables;

        -- 显示数据表 user 的结构
        describe user;
        -- 显示数据表 user 的列
        -- show columns from user;
        ```

    - 查询（mysql 系统）用户  
        `select Host, User, Password as Pass from mysql.user;`
        > 命令行中，以 `\G` 结尾，表示垂直显示查询结果，如  
        > `select top(3) * from mysql.user\G`

    - 执行 sql 脚本 data.sql  
        `source data.sql`

    - [将查询结果导出到文件 user.txt 中](https://www.cnblogs.com/xh831213/archive/2012/04/10/2439886.html)  
        `select Host, User, Password as Pass from mysql.user into outfile './user.txt';`  
        <!-- 导入文件  
        `load data infile './user.txt' into table table03` -->
        > 若导入导出文件失败，且抛出 "ERROR 1290" 异常，解决方法见 [导出文件时抛出异常 "ERROR 1290 (HY000): ...](#error-1290)  
        > 导出的文件见 /var/lib/mysql/user.txt

    - [导入导出 csv 文件](https://www.cnblogs.com/lvdongjie/p/6274392.html)
        ```sql
        -- 导出 users 表中数据至 users.csv 文件中
        select * from users into outfile '/tmp/users.csv' fields terminated by ',' optionally enclosed by '"' escaped by '"' lines terminated by '\r\n';

        -- 将 users.csv 文件中数据导入到 users 表中
        load data infile '/tmp/users.csv' into table users fields terminated by ','  optionally enclosed by '"' escaped by '"' lines terminated by '\r\n';
        ```

    - 将 table02 中的数据备份到 table02b 中  
        `create table table02b(select * from table02);`

    - <span id="console-status">查看当前所有连接信息</span>  
        `status`  
        查看当前所有连接进程、等待时间、所处状态、是否 locked  
        `show full processlist`

3. 退出控制台环境  
    `exit`

## 控制台（其他）操作

- 使用 mysqldump 备份还原数据库  
    - 备份数据库  
        `mysqldump -u root commerce > backup.sql`
    - 还原数据库  
        `mysql -u root < backup.sql`

## 配置

- 使用 mysqladmin 修改 mysql 用户密码  
    `mysqladmin -u[用户名] -p[旧密码] password [新密码]`

- 实时查看 mysql 当前连接数
    - 方法一
        - 查看当前所有连接的详细资料  
            `./mysqladmin -uroot -p -h192.168.1.101 processlist`
        - 只查看当前连接数（其中 Threads 即为连接数）  
            `./mysqladmin -uroot -p -h192.168.1.101 status`
    - [方法二](#console-status)
        > 或直接使用 mysql 控制台执行查询连接进程信息，并导出到文件 conns.txt 中  
        >     `mysql -e 'show full processlist;' > conns.txt`

- 配置 mysql 最大连接数
    1. 编辑 my.cnf（windows 环境下为 my.ini），修改如下
        ```ini
        max_connections = 100
        ```
<!--
- mysql 历史命令记录
    - 方法一：配置 mysql 日志，记录历史执行语句
        1. 编辑 my.cnf（windows 环境下为 my.ini），修改如下
            ```ini
            [mysqld]
            log = /tmp/mysql.log
            ```
        2. 重启 mysql 服务
    - 方法二：~/.mysql_history 文件中记录每个用户使用数据库的操作命令  
        `tail -20 ~/.mysql_history`
        > - 该文件会把所有操作记录下来，包括创建用户和修改用户的明文密码。因此出于安全考虑，需禁止该功能  
        >   <font color="grey">（该文件是 mysql 编译安装时默认配置好的，不容易修改。但是最好不要保存，仅仅删除是不行的，要直接将其软连接到垃圾箱）</font>  
        >   `ln -s /dev/null ~/.mysql_history`
-->

- 在 shell 中执行 mysql 命令
    ```bash
    mysql -h192.168.1.102 -P3306 -uroot -p123456 testdb < EOF | tee - > oper.log
    show tables;
    EOF
    ```

## <span id="mysql-syntax">MySql 语法</span>

- DQL（数据查询语言）
    - 简单查询
        ```sql
        -- 查询
        select * from users;

        select id, name, sex, round(datediff(curdate(), birth)/365.2422) as age from users;
        ```

    - 分页查询
        ```sql
        -- 返回 4 行，跳过前 9 条记录，即返回第 10、11、12、13 条记录
        select * from users limit 9,5;
        select * from users limit 4 offset 9;
        ```

    - 其他
        ```sql
        -- 查找 name 字段有重复的记录
        select name from users group by name having count(1) > 1;
        ```

- DML（数据操纵语言）
    - 简单增删改
        ```sql
        -- 新增
        insert into users(name, telephone) values('luffy', '17612345678');
        insert into users values(null, 'lina', 'female', '1993-11-23 00:00:00', '17612345679', null, now());
        insert into users(name, sex, birth, telephone, address) values('Haley', 'female', '1993-05-17', '18617382659', '河南开封朝阳区新华路23号');
        
        -- 修改
        update users set birth = now() where id = 1;
        update users set birth = '1993-11-24' where name = 'lina';

        -- 删除
        delete from users where name = 'lina';
        ```

    - 复杂增删改
        ```sql
        -- 插入一条数据，如果主键已存在，则修改该数据
        insert into users(id, name, telephone) values(1, 'zhangsan', '17612345678') on duplicate key update name = 'lisi', telephone = '17612335679';

        -- 复制数据到同一张表 users 中
        insert into users(name, sex, telephone) select name, sex, telephone from users;
        ```

    - 清空表数据
        ```sql
        truncate table users;
        ```

    - [锁](https://www.cnblogs.com/luyucheng/p/6297752.html)  
        - MySQL 各存储引擎使用了三种类型（级别）的锁定机制
            - 表级锁定（table-level）  
                开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。适合于以查询为主，只有少量按索引条件更新数据的应用。  
                使用表级锁定的主要是 MyISAM、MEMORY、CSV 等一些非事务性存储引擎。
            - 行级锁定（row-level）  
                开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。适合于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用。  
                使用行级锁定的主要是 InnoDB 存储引擎  
                InnoDB 行锁是通过给索引上的索引项加锁来实现的，只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁
                ```sql
                -- https://www.zhihu.com/tardis/sogou/art/143866444

                -- 悲观锁：每次获取商品时，对该商品加排他锁。也就是在用户A获取获取 id = 1 的商品信息时对该行记录加锁，期间其他用户（查询或修改）阻塞等待直至释放锁。悲观锁适合写入频繁的场景
                start transaction;
                select * from goods where id = 1 for update; -- for update 仅适用于 InnoDB，且必须在事务块 BEGIN/COMMIT 中才能生效。在进行事务操作时，通过 for update 语句，MySQL 会对查询结果集中每行数据都添加排他锁，其他线程对该记录的更新与删除操作都会阻塞。排他锁包含行锁、表锁。
                update goods set stock = stock - 1 where id = 1;
                commit;

                -- 乐观锁方案：每次获取商品时，不对该商品加锁。在更新数据的时候需要比较程序中的库存量与数据库中的库存量是否相等，如果相等则进行更新，反之程序重新获取库存量，再次进行比较，直到两个库存量的数值相等才进行数据更新。乐观锁适合读取频繁的场景
                select * from goods where id = 1;
                start transaction;
                update goods set stock = stock - 1 where id = 1 and stock = cur_stock;
                commit;

                -- 乐观锁方案：每次获取商品时，不对该商品加锁。在更新数据的时候需要比较程序中的库存量是否充足
                start transaction;
                update goods set stock = stock - 1 where id = 1 and stock >= 1; -- 返回成功修改行数应该为 1，否则判定失败，需要回滚
                commit;
                ```
            - 页级锁定（page-level） 
                开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。  
                使用页级锁定的主要是 BerkeleyDB 存储引擎
        - 共享锁（读锁）/ 排它锁（写锁）
            - 共享锁又叫读锁，是读取操作创建的锁。其他用户可以同步读取数据，但是不能够写数据。
            - 排它锁又叫写锁，是更新操作创建的锁。其他用户不可以同步读取数据更不能写数据。
        - MyIsam锁模式  
            MyIsam引擎在执行查询语句前，会自动给涉及到的表添加读锁；在更新操作前会自动给涉及到的表加写锁。
            1. 对MyIsam中的表进行读取操作，其他用户可以进行读取，但是不能修改；
            2. 对MyIsam中的表进行更新操作，其他用户不能进行读取，更不能进行操作。
        - InnoDb锁模式 
            对于insert、update、delete，InnoDB会自动给涉及的数据加排他锁（X）；对于一般的Select语句，InnoDB不会加任何锁，事务可以通过以下语句给显示加共享锁或排他锁。
            共享锁：SELECT ... LOCK IN SHARE MODE;
            排他锁：SELECT ... FOR UPDATE; SELECT ... FOR UPDATE NOWAIT;

    - Hint
        ```sql
        -- 强制索引 FORCE INDEX 
    	SELECT * FROM TABLE1 FORCE INDEX (FIELD1) …
        -- 以上的SQL语句只使用建立在FIELD1上的索引，而不使用其它字段上的索引。

        -- 忽略索引 IGNORE INDEX 
        SELECT * FROM TABLE1 IGNORE INDEX (FIELD1, FIELD2) …
        -- 在上面的SQL语句中，TABLE1表中FIELD1和FIELD2上的索引不被使用。 

        -- 关闭查询缓冲 SQL_NO_CACHE 
        SELECT SQL_NO_CACHE field1, field2 FROM TABLE1;
        -- 有一些SQL语句需要实时地查询数据，或者并不经常使用（可能一天就执行一两次）,这样就需要把缓冲关了,不管这条SQL语句是否被执行过，服务器都不会在缓冲区中查找，每次都会执行它。

        -- 强制查询缓冲 SQL_CACHE
        SELECT SQL_CALHE * FROM TABLE1;
        -- 如果在my.ini中的query_cache_type设成2，这样只有在使用了SQL_CACHE后，才使用查询缓冲。

        -- 优先操作 HIGH_PRIORITY
        -- HIGH_PRIORITY可以使用在select和insert操作中，让MYSQL知道，这个操作优先进行。
        SELECT HIGH_PRIORITY * FROM TABLE1;

        -- 滞后操作 LOW_PRIORITY
        -- LOW_PRIORITY可以使用在insert和update操作中，让mysql知道，这个操作滞后。
        update LOW_PRIORITY table1 set field1= where field1= …

        -- 延时插入 INSERT DELAYED
        INSERT DELAYED INTO table1 set field1= …
        -- INSERT DELAYED INTO，是客户端提交数据给MySQL，MySQL返回OK状态给客户端。而这是并不是已经将数据插入表，而是存储在内存里面等待排队。当mysql有空余时，再插入。另一个重要的好处是，来自许多客户端的插入被集中在一起，并被编写入一个块。这比执行许多独立的插入要快很多。坏处是，不能返回自动递增的ID，以及系统崩溃时，MySQL还没有来得及插入数据的话，这些数据将会丢失。

        -- 强制连接顺序 STRAIGHT_JOIN
        SELECT TABLE1.FIELD1, TABLE2.FIELD2 FROM TABLE1 STRAIGHT_JOIN TABLE2 WHERE …
        -- 由上面的SQL语句可知，通过STRAIGHT_JOIN强迫MySQL按TABLE1、TABLE2的顺序连接表。如果你认为按自己的顺序比MySQL推荐的顺序进行连接的效率高的话，就可以通过STRAIGHT_JOIN来确定连接顺序。

        -- 强制使用临时表 SQL_BUFFER_RESULT
        SELECT SQL_BUFFER_RESULT * FROM TABLE1 WHERE … 
        -- 当我们查询的结果集中的数据比较多时，可以通过SQL_BUFFER_RESULT.选项强制将结果集放到临时表中，这样就可以很快地释放MySQL的表锁（这样其它的SQL语句就可以对这些记录进行查询了），并且可以长时间地为客户端提供大记录集。
        ```

- DDL（数据定义语言）
    > MySQL 的 DDL 语句是非事务的，即不能对 DLL 语句进行回滚操作
    - 数据库管理
        - 创建数据库 commerce  
            `create database commerce;`  
        - 删除数据库 commerce  
            `drop database commerce;`

    - 数据表管理
        - 创建数据表 users
            ```sql
            use commerce;

            -- create table users (
            --     name varchar(20),
            --     sex char(1),
            --     birth date
            -- );

            create table if not exists users (
                id int not null auto_increment,
                name varchar(32) not null,
                -- age int not null,
                sex enum('male', 'female') not null default 'male',
                -- check(sex like '男' or sex like '女'), -- mysql 中 check 无效，使用 enum 替代
                birth date null,
                telephone char(11) not null,
                -- check(len(telephone) = 11),
                address varchar(255) null,
                updatetime datetime not null default now(),
                primary key(id)
                -- primary key(id, name) -- 联合主键
            ) engine = InnoDB default charset = utf8;
            ```
            > 引擎类型，多为 engine = InnoDB，如果省略了 engine = ... 语句，则使用默认的引擎 MyISAM
        - 删除数据表 users  
            `drop table users;`
        - 修改数据表
            ```sql
            -- 删除字段
            alter table users drop telephone;

            -- 添加字段
            alter table users add telephone char(11);
            
            -- 修改字段类型
            -- alter table users modify telephone varchar(20);

            -- 修改字段名称及类型
            -- alter table users change telephone telephone2 char(14) not null;

            -- 修改字段默认值
            -- alter table users alter telephone set default '13212345678';

            -- 修改表名
            -- alter table users rename to users2;
            ```

    - 创建索引
        ```sql
        -- 主键索引，一种特殊的唯一索引，不允许有空值。
        alter table `users` add primary key(`id`);

        -- 唯一索引，与“普通索引”类似，不同的就是：索引列的值必须唯一，但允许有空值。
        alter table `users` add unique(`id`);
        
        -- 普通索引，最基本的索引，没有任何限制
        alter table `users` add index index04(`id`);
        
        -- 全文索引，仅可用于 MyISAM 表，针对较大的数据，生成全文索引很耗时好空间。
        alter table `users` add fulltext(`id`);
        
        -- 组合索引，为了更多的提高 mysql 效率可建立组合索引，遵循“最左前缀”原则。
        alter table `users` add index index04(`id`, `name`);
        ```

    - 外键约束
        ```sql
        create table orders (
            ...
            user_id int not null,
            ...
            -- constraint constraint_name -- （可选）为外键约束定义约束名称，默认将自动生成一个名称
            foreign key fk_user(user_id) -- 指定子表（当前所在表）中引用父表中主键列的列
            references users(id) -- 指定父表及其子表中列的引用
            -- on update action -- （可选）指定在父表中的行更新时，子表中的行会怎样执行操作
            -- on delete action -- （可选）
        );

        alter orders drop foreign key fk_user;

        alter orders add constraint constraint_name foreign key fk_user(user_id) references users(id) on update action on delete action;

        -- 查看 orders 表的外键
        -- show create table orders;
        ```

    - 存储过程
        ```sql
        drop procedure if exists fake_users;

        delimiter $$
        create procedure fake_users(count int)
        begin
            declare i int default 0;
            -- declare r int default 0;
            declare rand_name varchar(32);
            declare rand_sex enum('male', 'female');
            declare rand_date date;
            declare rand_phone char(11);

            set @charset = 'abcdefghijklmnopqrstuvwxyz';
            set @numberset = '0123456789';

            while i < count
            do
                set @j = 0;
                set rand_name = '';
                while @j < floor(3 + rand() * 6) -- 随机生成值为 [3, 8] 的整数（FLOOR(min + RAND() * (max – min + 1))）
                do
                    set rand_name = concat(rand_name, substr(@charset, floor(rand() * length(@charset)) + 1, 1));
                    set @j = @j + 1;
                end while;

                if round(rand()) = 0 then
                    set rand_sex = 'male';
                else
                    set rand_sex = 'female';
                end if;

                set rand_date = date_add('1990-01-01', interval floor(1 + (rand() * 3650)) day); -- 随机生成起始日期为 1990-1-1，最高 10 年的日期

                set @j = 1;
                set rand_phone = '1';
                while @j < 11
                do
                    set rand_phone = concat(rand_phone, substring(@numberset, floor(1 + rand() * 10), 1));
                    set @j = @j + 1;
                end while;

                insert into users(name, sex, birth, telephone) values(rand_name, rand_sex, rand_date, rand_phone);

                set i = i + 1;
            end while;
        end
        $$
        delimiter ;

        call fake_users(20); -- 随机生成 20 条 users 数据
        ```

    - 触发器
        - 查看触发器  
            `show triggers;`
        - 创建触发器
            ```sql
            drop trigger if exists `users_trigger`;

            delimiter //
            create trigger users_trigger before update
            on users for each row
            begin
                set NEW.updatetime = now(); -- 当 users 表有数据更新时，修改该记录的 updatetime 为当前时间
            end //

            delimiter ;
            ```
            > 创建触发器格式如下
            > ```sql
            > CREATE TRIGGER [触发器名] [触发时机: BEFORE|AFTER] [触发事件: INSERT|DELETE|UPDATE]
            > ON [表名] FOR EACH ROW
            > BEGIN
            >     [执行语句列表]
            > END
            > ```

    - 视图
        - 创建视图
            ```sql
            create view v_users as select id, name, sex from users;
            -- create view v_users(ID, 姓名, 性别) as select id, name, sex from users;
            ```
        - 修改、删除视图
            ```sql
            -- 创建视图，已存在则替换
            create or replace view v_users as select id, name, sex from users;

            -- 修改视图
            alter view v_users as select * from users;

            -- 删除视图
            drop view v_users;
            ```
        - 查看视图
            ```sql
            -- 查看所有视图
            show table status where comment = 'view';

            -- 查看视图 v_users
            show create view v_users;
            ```
        - 使用视图
            ```sql
            select * from v_users;

            update v_users set name = '小红' WHERE id = 2;
            ```

    - 事务
        - 手动执行事务
            ```sql
            start transaction; -- 开启事务

            ...

            commit;
            -- rollback;
            ```
        - 事务管理
            ```sql
            -- 查看当前运行的事务
            select * from information_schema.innodb_trx;
            -- kill [trx_mysql_thread_id]; -- 杀掉指定事务线程，其中 trx_mysql_thread_id 来自上一步查询的事务线程 id

            -- 查看当前运行的事务的账户和事务开始的时间，及其事务语句
            select a.id, a.user, a.host, b.trx_started, b.trx_query from information_schema.processlist a right outer join information_schema.innodb_trx b on a.id = b.trx_mysql_thread_id;
            ```
        - 事务隔离级别设置
            ```sql
            -- 查看当前会话的事务隔离级别
            show variables like '%transaction_isolation%';

            -- 设置全局事务隔离级别，事务的隔离级别有：未提交读(read uncommitted)、已提交读(read committed)、可重复读(repeatable read)、串行化(serializable)
            set global transaction isolation level read committed;

            -- 设置当前会话的事务隔离级别
            set session transaction isolation level read committed;
            ```
        > - 嵌套事务：有多个 begin / commit / rollback 这样的事务块的事务，并且有父子关系。子事务的提交完成后不会真的提交，而是等到父事务提交才会真正的提交。
        > - MySQL 不支持事务嵌套：已经开启事务后，再开启事务（start transaction），会隐式的提交（commit）上一个事务（即第二个 begin 语句默认会变为 commit;begin;）。可以通过部分事务（savepoint）来实现嵌套。

    - 锁管理
        ```sql
        -- 查询是否锁表
        show OPEN TABLES where In_Use > 0;

        -- 查看所有进程
        show processlist; -- MySQL
        show full processlist; -- MariaDB

        -- 根据查询的进程，kill 指定连接的进程
        kill $pid

        -- 查看正在锁的事务
        select * from information_schema.innodb_locks;
        select * from performance_schema.data_locks; -- MySQL 版本 ≥ 8.0.13
        -- 查看等待锁的事务
        select * from information_schema.innodb_lock_waits;
        select * from performance_schema.data_lock_waits; -- MySQL 版本 ≥ 8.0.13

        -- 查看 innodb 引擎的运行时信息
        show engine innodb status\G;

        -- 查看服务器状态
        show status like '%lock%';

        -- 查看超时时间
        show variables like '%timeout%';
        ```

    - 日志  
        general_log：该日志用于记录 MySQL 中所有运行过的 SQL
        ```sql
        -- 查看 general_log 是否开启，及文件名
        show variables like '%general_log%';

        -- 开启 general_log
        set global general_log = 1;

        -- 关闭 general_log
        set global general_log = 0;
        ```

- DCL（数据控制语言）
    - 用户管理、授权
        - 添加用户
            - 方法一：新增一个用户 test01，允许任意远程主机可以登录，并设置其密码为 123456  
                `create user 'test01'@'%' identified by '123456';`
                > 新增用户命令的格式为  
                > `CREATE USER 'username'@'host' IDENTIFIED BY 'password';`  
                > 其中，  
                > - username 表示将创建的用户名；
                > - host 指定该用户在哪个主机上可以登录，如 "localhost" 只允许本地用户；"%" 允许任意远程主机。
                > - password 表示该用户的登录密码，密码可以为空,即无需密码可以直接登录。

            - 方法二：新增一个用户 test02，密码为 123，只允许本地访问  
                `insert into mysql.user(Host, User, Password) values("localhost", "test02", password("123"));`
            
            - 方法三：不使用以上方法创建用户，直接[对一个新用户授权](#grant-user)
        - 用户授权
            1. <span id="grant-user">授权</span>  
                授权用户 test03 拥有数据库 db01 的所有权限  
                `grant all privileges on db01.* to test03@localhost identified by '123';`  
                授权用户 test03 可以在任何主机上登录，并对所有数据库的表有查询、插入、修改、删除的权限  
                `grant select, insert, update, delete on *.* to "test03" Identified by "123";`
                > 授权命令的格式为  
                > `GRANT privileges ON database.table TO username@host IDENTIFIED BY "password";`  
                > 其中，
                > - 权限 privileges 可以为 select、delete、update、create、drop
                > - 数据库、数据表表示格式为 `database.table`，如 "*.*" 表示所有数据库的所有表，"db01.user" 表示数据库 db01 的 user 表
                > - 授权的用户（如 test03）如果不存在，则会自动创建该用户

            2. <span id="flush-privileges">刷新系统权限表</span>  
                `flush privileges;`
        - 删除用户
            1. 删除用户 test03
                - 方法一：  
                    `drop user test03@'localhost';`
                - 方法二：  
                    `delete from user where User = 'test03';`
            2. [刷新系统权限表](#flush-privileges)
        - 修改密码  
            设置（mysql 用户中的）root 用户的密码为 123456  
            `set password for root=password("123456");`

## Troubleshooting

- 远程访问连接拒绝
    - <span id="allow-root-remote">设置允许 mysql 用户中的 root 用户远程访问</span>  
        默认情况下，mysql 用户中的 root 用户只允许本地访问（Host 为 localhost），远程无法访问。  
        如需要允许 root 用户远程访问，则需修改数据库 mysql 下的表 user 中相关 Host 的值为 "%"  
        `mysql -u root –p`
        ```bash
        use mysql;

        update user set host = '%' where user = 'root';
        ```
    - mysql 可以用 localhost 连接，但不能用 ip 连接的问题解决方法  
        在本机上通过 "mysql -h locathost -P 3306 -u test -p123456" 可以访问，但是通过 "mysql -h 192.168.128.137 -P 3306 -u test -p123456" 无法访问，把mysql 里面 user 表里面的 Host 设置为了 % 也没有作用，最后通过修改配置文件 /etc/mysql/my.cnf，注释掉 bind-address 所在行即可  
        `sed -i 's/^bind-address/#bind-address/g' /etc/mysql/my.cnf`  
        [重启 mysql 服务](#restart-mysqld)

- <span id="error-1290">[导出文件时抛出异常 "ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement"](http://blog.csdn.net/HHTNAN/article/details/78520511)</span>  
    默认情况下，mysql 限制了导出文件的路径，执行以下命令查看当前 secure-file-priv 的值  
    `show variables like '%secure%';`  
    编辑配置文件 /etc/mysql/conf.d/mysql.cnf，在 "[mysqld]" 节点下添加/修改 secure-file-priv 配置项  
    `echo '[mysqld]' >> /etc/mysql/conf.d/mysql.cnf`  
    `echo 'secure_file_priv = "/"' >> /etc/mysql/conf.d/mysql.cnf`  
    [重启 mysql 服务](#restart-mysqld)

- 中文乱码  
    查看编码格式  
    `show variables like 'char%';`  
    设置编码格式  
    ~~`set character_set_database = utf8;`~~  
    ~~`set character_set_server = utf8;`~~  
    `set names utf8`

## [mysql 数据类型](http://www.cnblogs.com/JemBai/archive/2009/08/20/1550683.html)

| mysql 数据类型 | 说明 | java 数据类型 |
| :--- | :---- | :------ |
| `boolean` 或 `tinyint(1)` | insert value：true、false | `java.lang.Boolean` |

## 其他

<details>
<summary>在 docker 中运行 mysql</summary>

1. 启动 mysql 实例  
    ~~`docker run --name mysql01 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:tag`~~  
    `docker run --name mysql01 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql`
    > 其中，  
    > `mysql01` 为容器别名  
    > `123456` 为初始化设置的 root 用户的密码  
    > `tag` 为 mysql 的版本，不写默认使用最新版  
    > `-p 3306:3306` 表示在这个容器中使用 3306 端口（第二个）映射到本机的端口号也为 3306（第一个）
2. 连接到 mysql 实例
    - 连接到本地 mysql  
        `docker run -it --link mysql01:mysql --rm mysql sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'`
    - 连接其他地方的 mysql  
        `docker run -it --rm mysql mysql -hsome.mysql.host -umysql01-user -p`
3. 切换到容器 shell 中  
    `docker exec -it mysql01 bash`  
    退出输入  
    `exit`
4. 查看日志  
    `docker logs mysql01`

</details>

[SQL语言的分类（DQL、DML、DDL、DCL的概念与区别）](https://blog.csdn.net/weixin_39703170/article/details/79011704)

[MySQL 性能优化神器 Explain 使用分析](https://segmentfault.com/a/1190000008131735)