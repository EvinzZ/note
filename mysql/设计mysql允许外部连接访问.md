登陆Mysql服务器，执行以下命令

```mysql
use mysql;

update user set host = '%' where user = 'root';

flush privileges;

exit;
```

