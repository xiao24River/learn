# MySQL安装使用说明

## 一、MySQL的下载与安装

### 1.1 下载

### 1.2 安装

#### 1.2.1 免安装版

1. 下载完成后，解压到相应的目录：`D:\mysql5.7.29`;

2. 配置MySQL的配置文件,在`D:\mysql5.7.29`下，新建`my.ini`配置文件；

   ```
   [client]
   # 设置mysql客户端默认字符集
   default-character-set=utf8
    
   [mysqld]
   # 设置3306端口
   port = 3306
   # 设置mysql的安装目录
   basedir=D:\\mysql5.7.29
   # 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
   datadir=D:\\mysql5.7.29\\sqldata
   # 允许最大连接数
   max_connections=20
   # 服务端使用的字符集默认为8比特编码的latin1字符集
   character-set-server=utf8
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB
   ```

3. 需用管理员身份运行`cmd`命令，如果不用管理员运行，后面启动将会报错;

4. 切换目录到`D:\mysql5.7.29\bin`下:

   ![image-20200416230144561](C:\Users\28219\AppData\Roaming\Typora\typora-user-images\image-20200416230144561.png)

5. 初始化数据库：

   ```
   mysqld –initialize –console
   ```

   执行完成后，会输出`root`用户的初始默认密码：如`A temporary password is generated for root@localhost: APWCY5ws&hjQ`，其中`APWCY5ws&hjQ`就是`root`用户的初始默认密码，可以在登录后进行修改;

6. 输入安装命令：

   ```
   mysqld install
   ```

7. 如果`D:\mysql5.7.29`目录下没有`data`目录需要初始化：

   ```
   mysqld –initialize-insecure 
   ```

8. 启动服务命令：

   ```
   net start mysql
   ```

#### 1.2.2 安装版

## 二、MySQL的运行

### 2.1 登录MySQL

```mysql
mysql -h 主机名 -u 用户名 -p
```

参数说明：

- -h : 指定客户端所要登录的 MySQL 主机名, 登录本机(localhost 或 127.0.0.1)该参数可以省略;

- -u : 登录的用户名;

- -p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。

  登录本机：

  ```mysql
  mysql -u root -p
  ```

### 2.2 MySQL用户设置

1. 添加用户

   > MySQL 5.7之前

   ```mysql
   INSERT INTO user(host,user,password,select_priv,insert_priv,update_prive) values('%','wbwp_cas_zhejiang',MD5('ZJcas2019'),'Y','Y','Y')
   ```

   > MySQL 5.7之后

   ```mysql
   INSERT INTO user
   	(host,user,authentication_string, -- password在5.7后修改为authentication_string
    	select_priv,
    	insert_priv,
    	update_prive) 
   values('%',
          'wbwp_cas_zhejiang',
          MD5('ZJcas2019'), -- 密码需要用函数来进行加密 ，如`PASSWORD()`,`MD5()`，`PASSWORD()`在8.0.11中已被移除
          'Y','Y','Y')
   ```

   

   用户授权

   * 创建用户授权

   ```mysql
   -- 此命令执行后会重新载入授权表，若不使用，则新建用户无法连接MySQL服务器，除非重启MySQL服务器
   FLUSH PRIVILEGES;
   ```

   | `select_prive`    |      |
   | ----------------- | ---- |
   | `Insert_pri`      |      |
   | `Update_pri`      |      |
   | `Delete_pri`      |      |
   | `Create_pri`      |      |
   | `Drop_pri`        |      |
   | `Reload_pri`      |      |
   | `Shutdown_priv`   |      |
   | `Process_priv`    |      |
   | `File_priv`       |      |
   | `Grant_priv`      |      |
   | `References_priv` |      |
   | `Index_priv`      |      |
   | `Alter_priv`      |      |

   - Grant命令添加用户授权

   ```mysql
   GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP 
   	ON cbsd.* 
   	TO 'wbwp_cas_zhejiang'@'localhost'
   	IDENTIFIED BY 'ZJcas2019';
   ```


## 三、备份与还原

```mysql
/* 备份与还原 */ ------------------
备份，将数据的结构与表内数据保存起来。
利用 mysqldump 指令完成。
-- 导出
mysqldump [options] db_name [tables]
mysqldump [options] ---database DB1 [DB2 DB3...]
mysqldump [options] --all--database
1. 导出一张表
　　mysqldump -u用户名 -p密码 库名 表名 > 文件名(D:/a.sql)
2. 导出多张表
　　mysqldump -u用户名 -p密码 库名 表1 表2 表3 > 文件名(D:/a.sql)
3. 导出所有表
　　mysqldump -u用户名 -p密码 库名 > 文件名(D:/a.sql)
4. 导出一个库
　　mysqldump -u用户名 -p密码 --lock-all-tables --database 库名 > 文件名(D:/a.sql)
可以-w携带WHERE条件
-- 导入
1. 在登录mysql的情况下：
　　source  备份文件
2. 在不登录的情况下
　　mysql -u用户名 -p密码 库名 < 备份文件
```

