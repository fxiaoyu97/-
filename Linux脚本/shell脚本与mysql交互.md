## mysql命令参数详解：

+ `-u 用户名`
+ `-p 用户密码`
+ `-h 服务器IP地址`
+ `-D 连接的数据库`
+ `-N 不输出列信息`
+ `-B 使用tab键代替默认交互分隔符`
+ `-e 执行SQL语句`
+ `-E 垂直输出`
+ `-H 以HTML格式输出`
+ `-X 以XML格式输出`

```shell
mysql -uroot -p123456 -h192.168.0.112 -B -N -D school -e "select * FROM users"
```

插入数据脚本

```shell
#！/bin/bash
#

user="root"
password="123456"
host="192.168.0.112"

#要读取的文件的各字段之间的分隔符
IFS="|"

cat data-2.txt | while read id name birth sex
do
		# $id必须使用单引号，使用双引号的时，系统会默认进行替换，SQL语句执行时，插入的数值就没有引号
		mysql -u"$user" -p"$password" -h"$host" -e "INSERT INTO school.student values('$id','$name')"
done
```

## 备份数据库

`Mysqldump`

常用参数详解：

		+ `-u 用户名`
		+ `-p 密码`
  + `-h 服务器ip地址`
  + `-d 等价于--no-data` 	只导出表结构
  + `-t 等价于--no-create-info` 只导出数据，不导出建表语句
  + `—A 等价于--all-databases`
  + `-B 等价于--databases`  导出一个或多个数据库

FTP常用指令

| 参数 | 含义                              | 示例                 |
| ---- | --------------------------------- | -------------------- |
| open | 与FTP服务器建立连接               | `open 192.168.0.112` |
| user | 有权限登录FTP服务器的用户名和密码 | `user root 123456`   |

需求：将school中的score表备份，并且将备份数据通过FTP传输到192.168.0.112的/data/backup目录下

