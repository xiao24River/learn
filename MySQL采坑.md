# Oracle转MySQL采坑

> [https://github.com/Snailclimb/JavaGuide/blob/master/docs/database/%E4%B8%80%E5%8D%83%E8%A1%8CMySQL%E5%91%BD%E4%BB%A4.md](https://github.com/Snailclimb/JavaGuide/blob/master/docs/database/一千行MySQL命令.md)



> 切换MySQL需要换方言，需要对自定义函数进行注册，否则hql中无法使用自定义函数:

```java
public class UseMySQL57Dialect extends MySQL57Dialect {
	public UseMySQL57Dialect() {
		super();
		registerHibernateType(Types.NULL, StandardBasicTypes.STRING.getName());
		/**
		 * 函数名必须是小写，试验大写出错 
		 * SQLFunctionTemplate函数第一个参数是函数的输出类型，
		 * varchar对应StandardBasicTypes.STRING 
		 * ?1代表第一个参数,?2代表第二个参数,所以写成my_bpm_task(?1,?2,?3,?4,?5,?6)
		 */
		registerFunction("my_bpm_task", 
                         new SQLFunctionTemplate(StandardBasicTypes.STRING, 		                              "my_bpm_task(?1,?2,?3,?4,?5,?6)"));
		registerFunction("fun_get_shuliang", 
                         new SQLFunctionTemplate(StandardBasicTypes.INTEGER,                                      "fun_get_shuliang(?1,?2)"));
		registerFunction("money_to_chinese", 
                         new SQLFunctionTemplate(StandardBasicTypes.STRING,                                        "money_to_chinese(?1)"));
	}
}
```



## 一、数据类型

> 在使用UTF8字符集的时候，MySQL手册上是这样描述的：
>
> + 基本拉丁字母、数字和标点符号使用一个字节；
> + 大多数的欧洲和中东手写字母适合两个字节序列：扩展的拉丁字母（包括发音符号、长音符号、重音符号、低音符号和其它音符）、西里尔字母、希腊语、亚美尼亚语、希伯来语、阿拉伯语、叙利亚语和其它语言；
> + 韩语、中文和日本象形文字使用三个字节序列。

### 1.1 整值类型

1. 整型

| 数据类型     | 范围                                 | 说明                                                         |
| ------------ | ------------------------------------ | ------------------------------------------------------------ |
| TINYINT(m)   | 1个字节 范围(-128~127)               | MySQL没有布尔型，常用TINYINT(1)表示布尔型，1表示true，0表示false |
| SMALLINT(m)  | 2个字节 范围(-32768~32767)           |                                                              |
| MEDIUMINT(m) | 3个字节 范围(-8388608~8388607)       |                                                              |
| INT(m)       | 4个字节 范围(-2147483648~2147483647) |                                                              |
| BIGINT(m)    | 8个字节 范围(+-9.22*10的18次方)      |                                                              |

2. 浮点型

| 数据类型    | 范围                                             | 说明 |
| ----------- | ------------------------------------------------ | ---- |
| FLOAT(m,d)  | 单精度浮点型  8位精度(4字节)   m总个数，d小数位  |      |
| DOUBLE(m,d) | 双精度浮点型  16位精度(8字节)   m总个数，d小数位 |      |

3. 定点数

| 数据类型      | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| DECIMAL(M, D) | M也表示总位数，D表示小数位数。保存一个精确的数值，不会发生数据的改变，不同于浮点数的四舍五入。将浮点数转换为字符串来保存，每9位数字保存为4个字节。 |

### 1.2 字符串类型

| 类型       | 范围                                                         | 说明                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| char(m)    | m表示能存储的最大长度，此长度是字符数，非字节数。最多255个字符，与编码无关。 | 定长字符串，速度快，必须在括号里定义长度，自动用空格填充，且在检索的时候后面的空格会隐藏掉，所以检索出来的数据需要记得用trim之类的函数去过滤空格。 |
| varchar(m) | m表示能存储的最大长度，此长度是字符数，非字节数。最多65535字符，与编码有关。utf8 最大为21844个字符，gbk 最大为32766个字符，latin1 最大为65532个字符 | 必须在括号里定义长度，不进行空格自动填充。varchar保存数据时第一个字节是空的，不存在任何数据，另外如果数据小于255个字节，则采用一个字节来保存长度，反之需要两个字节来保存。 |
| text       |                                                              | 非二进制字符串（字符字符串）。不需要定义长度，也不会计算总长度。 |
| blob       |                                                              | 二进制字符串（字节字符串），类似如text                       |
| binary     |                                                              | 二进制字符串（字节字符串），类似如char                       |
| varbinary  |                                                              | 二进制字符串（字节字符串），类似如varchar                    |

### 1.3 日期时间类型

| 类型      | 大小  | 说明                                                         | 范围                                       |
| --------- | ----- | ------------------------------------------------------------ | ------------------------------------------ |
| datetime  | 8字节 | 日期及时间：<br>YYYY-MM-DD hh:mm:ss                          | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 |
| date      | 3字节 | 日期：<br />YYYY-MM-DD<br />YY-MM-DD<br />YYYYMMDD <br />YYMMDD<br />YYYYMMDD<br />YYMMDD | 1000-01-01 到 9999-12-31                   |
| timestamp | 4字节 | 时间戳：<br />YY-MM-DD hh:mm:ss <br />YYYYMMDDhhmmss<br />YYMMDDhhmmss<br />YYYYMMDDhhmms<br />YYMMDDhhmmss | 19700101000000 到 2038-01-19 03:14:07      |
| time      | 3字节 | 时间：<br />hh:mm:ss <br />hhmmss <br />hhmmss               | -838:59:59 到 838:59:59                    |
| year      | 1字节 | 年份：<br />YYYY<br />YY                                     | 1901 - 2155                                |




## 二、Mysql常用函数

### 2.1 数学函数

```mysql
abs(x)                  	-- 返回x的绝对值
bin(x)                  	-- 返回x的二进制（oct返回八进制，hex返回十六进制）
ceiling(x)              	-- 返回大于x的最小整数值
exp(x)                  	-- 返回值e（自然对数的底）的x次方
floor(x)                	-- 返回小于x的最大整数值
greatest(x1,x2,...,xn)  	-- 返回集合中最大的值
least(x1,x2,...,xn)     	-- 返回集合中最小的值
ln(x)                   	-- 返回x的自然对数
log(x,y)                	-- 返回x的以y为底的对数
mod(x,y)                	-- 返回x/y的模（余数）
pi()                    	-- 返回pi的值（圆周率）
rand()                  	-- 返回０到１内的随机值,可以通过提供一个参数(种子)使rand()随机数生成器生成一个指定的值。
round(x,y)              	-- 返回参数x的四舍五入的有y位小数的值
sign(x)                 	-- 返回代表数字x的符号的值
sqrt(x)                 	-- 返回一个数的平方根
truncate(x,y)           	-- 返回数字x截短为y位小数的结果
```

### 2.2 字符串函数

```mysql
ascii(char) 				-- 返回字符的ascii码值
bit_length(str) 			-- 返回字符串的比特长度
concat(s1,s2...,sn) 		-- 将s1,s2...,sn连接成字符串
concat_ws(sep,s1,s2...,sn)  -- 将s1,s2...,sn连接成字符串，并用sep字符间隔
insert(str,x,y,instr)   	-- 将字符串str从第x位置开始，y个字符长的子串替换为字符串instr，返回结果
find_in_set(str,list)   	-- 分析逗号分隔的list列表，如果发现str，返回str在list中的位置
lcase(str)/lower(str) 		-- 返回将字符串str中所有字符改变为小写后的结果
left(str,x) 				-- 返回字符串str中最左边的x个字符
length(s)   				-- 返回字符串str中的字符数
ltrim(str)  				-- 从字符串str中切掉开头的空格
position(substr,str)    	-- 返回子串substr在字符串str中第一次出现的位置
quote(str)  				-- 用反斜杠转义str中的单引号
repeat(str,srchstr,rplcstr) -- 返回字符串str重复x次的结果
reverse(str) 				-- 返回颠倒字符串str的结果
right(str,x) 				-- 返回字符串str中最右边的x个字符
rtrim(str) 					-- 返回字符串str尾部的空格
strcmp(s1,s2)   			-- 比较字符串s1和s2
trim(str)   				-- 去除字符串首部和尾部的所有空格
ucase(str)/upper(str) 		-- 返回将字符串str中所有字符转变为大写后的结果
```

### 2.3 日期和时间函数

```mysql
now()  						 -- 返回当前的日期和时间 2020-05-22 23:05:16
curdate()或current_date() 	-- 返回当前的日期 2020-05-22
curtime()或current_time() 	-- 返回当前的时间 23:07:28
date_add(date,interval int keyword)  -- 返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：selectdate_add(current_date,interval 6 month);
date_format(date,fmt)  		-- 依照指定的fmt格式格式化日期date值
date_sub(date,interval int keyword)  -- 返回日期date加上间隔时间int的结果(int必须按照关键字进行格式化),如：select date_sub(current_date,interval 6 month);
dayofweek(date)   			-- 返回date所代表的一星期中的第几天(1~7)
dayofmonth(date)  			-- 返回date是一个月的第几天(1~31)
dayofyear(date)   			-- 返回date是一年的第几天(1~366)
dayname(date)   			-- 返回date的星期名，如：select dayname(current_date);
from_unixtime(ts,fmt)  		-- 根据指定的fmt格式，格式化unix时间戳ts
hour(time)   				-- 返回time的小时值(0~23)
minute(time)   				-- 返回time的分钟值(0~59)
month(date)   				-- 返回date的月份值(1~12)
monthname(date)   			-- 返回date的月份名，如：select monthname(current_date);
quarter(date)   			-- 返回date在一年中的季度(1~4)，如select quarter(current_date);
week(date)   				-- 返回日期date为一年中第几周(0~53)
year(date)   				-- 返回日期date的年份(1000~9999)
```

### 2.4 聚合函数(常用于group by从句的select查询中)

```mysql
avg(col)     				-- 返回指定列的平均值
count(col)   				-- 返回指定列中非null值的个数
min(col)     				-- 返回指定列的最小值
max(col)     				-- 返回指定列的最大值
sum(col)     				-- 返回指定列的所有值之和
group_concat(col) 			-- 返回由属于一组的列值连接组合而成的结果
```

### 2.5 加密函数

```mysql
aes_encrypt(str,key)  		-- 返回用密钥key对字符串str利用高级加密标准算法加密后的结果，调用aes_encrypt的结果是一个二进制字符串，以blob类型存储
aes_decrypt(str,key)  		-- 返回用密钥key对字符串str利用高级加密标准算法解密后的结果
decode(str,key)   			-- 使用key作为密钥解密加密字符串str
encrypt(str,salt)   		-- 使用unixcrypt()函数，用关键词salt(一个可以惟一确定口令的字符串，就像钥匙一样)加密字符串str
encode(str,key)   			-- 使用key作为密钥加密字符串str，调用encode()的结果是一个二进制字符串，它以blob类型存储
md5()    					-- 计算字符串str的md5校验和
password(str)   			-- 返回字符串str的加密版本，这个加密过程是不可逆转的，和unix密码加密过程使用不同的算法。
sha()    					-- 计算字符串str的安全散列算法(sha)校验和
```

### 2.6 格式化函数

```mysql
-- 其中最简单的是format()函数，它可以把大的数值格式化为以逗号间隔的易读的序列。
date_format(date,fmt)       -- 依照字符串fmt格式化日期date值
format(x,y)                 -- 把x格式化为以逗号隔开的数字序列，y是结果的小数位数
inet_aton(ip)   			-- 返回ip地址的数字表示
inet_ntoa(num)   			-- 返回数字所代表的ip地址
time_format(time,fmt)  		-- 依照字符串fmt格式化时间time值
```





## 三、Oracle函数变换MySQL函数

### 3.1 `decode`：条件判断

```mysql
-- oracle
decode(sign( s.comp_yxq - DATE_FORMAT( NOW(), '%Y-%m-%d' )), -1, 1, 0)
-- mysql
CASE sign( s.comp_yxq - DATE_FORMAT( NOW(), '%Y-%m-%d' )) WHEN - 1 THEN 1 ELSE 0 END
```

### 3.2 `wm_concat`：行转列功能

> 即将查询出的某一列值使用逗号进行隔开拼接，成为一条数据。

```mysql
-- oracle
SELECT wm_concat( b.yw_lx || ':' || b.zw_id ) FROM sys_user_zhiwu b
-- mysql
SELECT GROUP_CONCAT( b.yw_lx || ':' || b.zw_id ) FROM sys_user_zhiwu b
```

### 3.3 `trunc` ：时间转换

1. 格式化时间:

```mysql
-- oracle
trunc(sysdate,'dd')
-- mysql
DATE_FORMAT(NOW(),'%Y-%m-%d %H:%i:%s')
```

2. 时间操作

```mysql
-- oralce
select add_months(sysdate,-6) from dual;  -- 2019/11/7 9:36:22
-- mysql
SELECT ADDDATE(NOW(), INTERVAL -6 MONTH); -- 2019-11-07 09:36:22
SELECT ADDDATE(NOW(), INTERVAL -7 DAY);   -- 2020-04-30 09:41:13
```





### 3.4 `null`和空字符串

+ `Oracle`

  > `null`等同于空字符''；

+ MySQL

  > 1. `mysql`插入`null`显示为`null`，插入空字符串显示空；
  > 2. 空字符串不占空间，`null`占空间

### 3.5 `MySQL`字符串的处理

#### 3.5.1 字符串的拼接

1. `CONCAT(s1, s2, ...)`：返回连接参数产生的字符串，一个或多个待拼接的内容，任意一个为`NULL`则返回值为`NULL`。

```mysql
SELECT CONCAT('现在的时间：',NOW());  -- 输出结果：现在的时间：2020-04-28 11:27:58
```

2. `CONCAT_WS(x, s1, s2, ...)`：返回多个字符串拼接之后的字符串，每个字符串之间有一个`x`。

```mysql
SELECT CONCAT_WS(';','A','b','C'); -- 输出结果：A;b;C
```

#### 3.5.2 字符串的截取

1. `SUBSTRING(s, n, len)`，`MID(s, n, len)`函数：两个函数作用相同，从字符串s中返回一个第n个字符开始、长度为len的字符串。

```mysql
SELECT SUBSTRING('您好，欢迎使用MySQL',8,14); 
SELECT MID('您好，欢迎使用MySQL',8,14);       
```

2. `LEFT(s, n)`，`RIGHT(s, n)`函数：前者返回字符串s从最左边开始的n个字符，后者返回字符串s从最右边开始的n个字符。

```mysql
SELECT LEFT('您好，欢迎使用MySQL',7);   
SELECT RIGHT('您好，欢迎使用MySQL',14);
```

#### 3.5.3 字符串与数字







## 四、其他语句

1. 查询所有表的行数

   ```mysql
use information_schema;
select table_name,table_rows from tables where TABLE_SCHEMA = "数据库名称" order by table_rows asc;
   ```

2. 获取所有表的视图脚本

   ```mysql
   use information_schema;
   select t.table_name,t.table_comment,
   CONCAT('create or replace view v_yzb_' , lower(t.table_name) , ' as select * from zjyzb.' , lower(t.table_name) , ';')
   from TABLES t where t.TABLE_SCHEMA='zjyzb' and t.table_comment <> ''
   order by t.table_name;
   ```

## 五、自定义函数

### 5.1 declare声明

```mysql
begin 
   -- 定义变量,必须声明在begin...end块的最前面
	DECLARE taskKey varchar(50);
	set taskKey := 0;
end
```

