# Ado.Net

SqlConnection

close、dispose 区别

close（）后，还可以open（）再打开连接

dispose（）后，清空连接字符串，需要重新设置连接字符串

## 连接池

数据库连接，耗时耗性能

建立物理通道，与服务器初次握手，身份验证，运行检查

重复使用已有的连接

容器，存放了一定数量的与数据库服务器的物理连接

取出空闲的连接，而非创建新的连接

减少连接数据库的开销，提高系统性能

已达到最大连接数，等待，直到有空闲的连接可用

Ado.Net 默认启用连接池   

**按连接字符串区分不同连接池**

## ExecuteScalar（）

必须保持连接，connection 不能关闭，返回object值

执行插入语句，返回生成的标识列的值

“insert into   ; select @@identity；”

## ExecuteReader（）

必须保持连接，返回 **SqlDataReader** 数据流

未读取到末尾时需先调用**SqlCommand**.**Cancel**(), 再调用 **SqlDataReader**.**Close**()

执行存储过程时，先调用**SqlDataReader**.**Close**(), 才能获取存储过程结果

## SqlDataReader

不要求随意读取，不修改，数据量小的情况下，速度快，占用内存小

## SqlDataAdapter

随意读取，可修改，数据量大的情况下，占用内存大

Fill（） 填充数据后断开连接

尽量选择ExecuteReader(), 效率更高

修改数据，注入 **SqlCommandBuilder**， Update（）

## 事务

### 原子性

要么完成，要么失败

### 一致性

事务开始前和结束后，数据库的完整性约束没有被破坏

### 隔离性

同一时间仅有一个请求用于统一数据

### 持久性

事务完成后，对数据库的更改持久保存在数据库中，不会被回滚

## SqlTransaction

