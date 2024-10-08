# MySQL

# 1.MySQL体系结构
### 数据库产品的架构一般可以分为应用层、逻辑层、物理层

<img src="./Images/MySQL.png" alt="无法显示该图片"/>

## 1.1 连接层
## 1.1.1 Connectors
### MySQL首先是一个网络程序，其在TCP之上定义了自己的应用层协议。所以要使用MySQL，我们可以编写代码，跟MySQL Server建立TCP连接，之后按照其定义好的协议进行交互。当然这样比较麻烦，比较方便的办法是调用SDK，比如Native C API、JDBC、PHP等各语言MySQL Connector，或者通过ODBC。但通过SDK来访问MySQL，本质上还是在TCP连接上通过MySQL协议跟MySQL进行交互。

## 1.2 应用层
## 1.2.1 Connection Management
### 每一个基于TCP的网络服务都需要管理客户端链接，MySQL也不例外。MySQL会为每一个连接绑定一个线程，之后这个连接上的所有查询都在这个线程中执行。为了避免频繁创建和销毁线程带来开销，MySQL通常会缓存线程或者使用线程池，从而避免频繁的创建和销毁线程。客户端连接到MySQL后，在使用MySQL的功能之前，需要进行认证，认证基于用户名、主机名、密码。如果用了SSL或者TLS的方式进行连接，还会进行证书认证。

## 1.3 逻辑层
## 1.3.1 SQL Interface
### MySQL支持DML（数据操作语言）、DDL（数据定义语言）、存储过程、视图、触发器、自定义函数等多种SQL语言接口。

## 1.3.2 Parser
### MySQL会解析SQL查询，并为其创建语法树，并根据数据字典丰富查询语法树，会验证该客户端是否具有执行该查询的权限。创建好语法树后，MySQL还会对SQl查询进行语法上的优化，进行查询重写。

## 1.3.3 Optimizer
### 语法解析和查询重写之后，MySQL会根据语法树和数据的统计信息对SQL进行优化，包括决定表的读取顺序、选择合适的索引等，最终生成SQL的具体执行步骤。这些具体的执行步骤里真正的数据操作都是通过预先定义好的存储引擎API来进行的，与具体的存储引擎实现无关。

## 1.3.4 Caches & Buffers
### MySQL内部维持着一些Cache和Buffer，比如Query Cache用来缓存一条Select语句的执行结果，如果能够在其中找到对应的查询结果，那么就不必再进行查询解析、优化和执行的整个过程了。

## 1.4 物理层
## 1.4.1 Pluggable Storage Engine可插拔存储引擎
### 存储引擎的具体实现，这些存储引擎都实现了MySQl定义好的存储引擎API的部分或者全部。MySQL可以动态安装或移除存储引擎，可以有多种存储引擎同时存在，可以为每个Table设置不同的存储引擎。存储引擎负责在文件系统之上，管理表的数据、索引的实际内容，同时也会管理运行时的Cache、Buffer、事务、Log等数据和功能。

## 1.4.2 File System
### 所有的数据，数据库、表的定义，表的每一行的内容，索引，都是存在文件系统上，以文件的方式存在的。当然有些存储引擎比如InnoDB，也支持不使用文件系统直接管理裸设备，但现代文件系统的实现使得这样做没有必要了。在文件系统之下，可以使用本地磁盘，可以使用DAS、NAS、SAN等各种存储系统。

# 2.MySQL访问路径
### 当客户端链接上MySQL服务端时，系统为其分配一个链接描述符thd，用以描述客户端的所有信息，将作为参数在各个模块之间传递。
<img src="./Images/MySQL_module.jpg" alt="无法显示该图片"/>

# 3.MySQL高级特性
## 3.1 存储过程
- 变量定义顺序必须是：存储函数变量、游标定义、游标异常、程序主体
- 定义务必加上授权： CREATE DEFINER=‘root’@‘localhost’ PROCEDURE sp();
- 存储过程相关权限：被操作表的相关权限及EXECUTE(执行权限)，ALTER ROUTINE(修改权限)，CREATE ROUTINE(创建权限)
- 不能使用动态游标，CURSOR中不能有动态的表名
- 查看创建存储过程/函数的语句：SHOW CREATE PROCEDURE/FUNCTION ps;
- 查看所有存储过程/函数：SHOW PROCEDURE/FUNCTION STATUS [LIKE ps];
- 调用存储过程：CALL sp();
- 调用存储函数：SELECT sp();

## 3.2 触发器
- 定义务必加上授权： CREATE DEFINER=`root`@`localhost` Trigger tgr();
- 定义语句：CREATE DEFINER=`root`@`localhost` Trigger tgr() AFTER/BEFORE INSERT/UPDATE/DELETE ON table FOR EACH ROW;
- 数据调用：NEW.*（更新后数据） OLD.*（更新前数据）
- 行级触发器，每一行都会触发动作
- 内部可以调用存储过程和函数
- 每种类型的Trigger在一张表上只能建立一个

## 3.3 分区表
### 分区类型：
- RANGE分区：基于属于一个给定连续区间的列值，把多行分配给分区。
- LIST分区：类似于按RANGE分区，区别在于LIST分区是基于列值匹配一个离散值集合中的某个值来进行选择。
- HASH分区：基于用户定义的表达式的返回值来进行选择的分区，该表达式使用将要插入到表中的这些行的列值进行计算。这个函数可以包含MySQL中有效的、产生非负整数值的任何表达式。
- KEY分区：类似于按HASH分区，区别在于KEY分区只支持计算一列或多列，且MySQL服务器提供其自身的哈希函数。必须有一列或多列包含整数值。
- 子分区：子分区是分区表中每个分区的再次分割。

# 4.MySQL数据文件解析
## 4.1 InnoDB表：有2种存储方式
### 默认方式：每表有1个独立文件和一个多表共享的文件
- tb_name.frm：表结构定义文件，位于数据库目录中
- ibdata：共享的表空间文件，默认位于数据目录(datadir指向的目录)中

### 自定义方式：独立的表空间
- tb_name.frm：表结构定义文件，位于数据库目录中
- tb_name.ibd：独有的表空间文件
	>在MySQL初始化中打开独立表空间功能的方法：
	
	>vi /etc/my.cnf (在[mysqld]段下添加)

	>innodb_file_per_table = ON

	>注：表空间：table space，是由InnoDB管理的特有格式的数据文件，内部可同时存储数据和索引

## 4.2 MyISAM表：每表有3个文件，都位于数据库目录中
- tb_name.frm：表结构定义文件
- tb_name.MYD：数据文件
- tb_name.MYI：索引文件


