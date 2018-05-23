## windows 下安装zip版的mysql

1. 初始化mysql
~~~sql
bin/mysqld --defaults-file=D:\MySQL\MySQL Server 5.7\my.ini --initialize
~~~
2. 注册服务
~~~sql
bin/mysqld install
~~~
3. 查询mysql 的服务, 
~~~
sc qc mysql
~~~
4. win + R 输入 regedit ,打开注册列表，定位到
~~~
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\MySQL
~~~
修改ImagePath
~~~
"D:\MySQL\MySQL Server 5.7\bin\mysqld" --defaults-file="D:\MySQL\MySQL Server 5.7\my.ini" MySQL
~~~
5. 启动服务
~~~
net start mysql
~~~
6. 进入mysql
~~~ sh
shell> mysql -u root -p
Enter password: (enter the random root password here)
~~~
Look in the server error log if you do not know this password.
After connecting, assign a new root password:
~~~sql
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
~~~