## Can't create/write to file '/tmp/MYokuZAo' (OS errno 28 - No space left on device)

**问题**

1. 未指定**tmpdir**目录
2. 权限问题：/tmp 权限不够
3. **磁盘不足**：/tmp 文件夹的磁盘满了,文件写不进去了

**解决**

1. my.ini中 [mysqld]里面添加一行**tmpdir**="d:/mysql/temp/"
2. 给tmp文件夹授权(chmod -R 777 /tmp)
3. 清空tmp文件夹里的数据；或者删除这个文件

执行完操作，重启mysql 服务，最后执行命令：`show variables like '%dir%'`，查看设置是否正确生效。

## null, message from server: “Host ‘xx.xx.xx.xx‘ is not allowed to connect to this MySQL server“

**原因：没有权限**

**解决**

1. 登录mysql
   1. `mysql -u root -p`
2. 切换数据库
   1. `use mysql`
3. 修改root用户的权限
   1. `update user set host = '%' where user = 'root' ;`
4. 刷新权限
   1. `flush privileges`