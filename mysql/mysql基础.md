1 mysql 重置密码：
```
mysqld --skip-grant-tables    //无启动登录
update user set authentication_string=password('password') where user='root'; //重置密码
flush privileges;
```

2 ERROR</br>
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```
set password = password('password1')
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
flush privileges;
```
