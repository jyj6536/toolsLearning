# zkCli.sh

zkCli.sh 是 ZooKeeper 官方提供的用于与 ZooKeeper 集群进行交互的命令行工具，主要用于开发调试。

zkCli.sh 可以实现以下功能

+ 创建 znode 节点
+ 获取数据
+ 观察 znode 变化
+ 设置数据
+ 创建 znode 的子节点
+ 列出 znode 的子节点
+ 检查状态
+ 移除/删除 znode

## 创建 znode

根据指定的路径创建 znode。参数 flag 指定创建的 znode 是临时节点、持久节点还是顺序节点。

+ 临时节点（-e）：会话超时或者客户端断开连接后会被自动删除
+ 顺序节点（-s）：确保 znode 路径是唯一的。zk 会在顺序节点后添加10个数字的序列号。例如，`/myapp` 会被转换为 `/myapp0000000000`，下一个顺序节点会被转换为 `/myapp0000000001`，可以与 -e 联合使用创建临时顺序节点
+ 不指定 flag：默认创建的是持续节点

**语法**

~~~
create [-es] /path data
~~~

**例子**

~~~
#创建数据为 test 的节点 /test
create /test "test"

#创建顺序节点
create -s /myapp

#创建临时节点（会话退出之后节点删除）
create -e /emnode
~~~

## 获取数据/添加观察点

获取指定 znode 关联的数据以及元数据/未指定 znode 添加观察点。

+ -s：显示统计信息
+ -w：添加观察点

**语法**

~~~
get [-sw] /path
~~~

**例子**

~~~
#获取节点数据以及统计信息
get -s /test

#为/test添加观察点
get -w /test
#执行delete /test之后输出
WATCHER::

WatchedEvent state:SyncConnected type:NodeDeleted path:/test
~~~

## **设置数据**

设置指定 znode 关联的数据。

**语法**

~~~
set /path /data
~~~

例子

~~~
#设置 /test 数据为2222
set /test 2222
~~~

## 列出子节点

列出指定 znode 的子节点。

+ -s：同时列出统计信息
+ -R：递归
+ -w：同时添加观察点

## 状态检查

列出指定 zndoe 的元信息。

**语法**

~~~
stat /path
~~~

## 删除节点

删除指定 znode。

**语法**

~~~
#删除指定 znode（不能有子节点）
delete /path
#删除指定 znode（递归删除）
daleteall /path
~~~

## ACL控制

[参考资料](https://it.cha138.com/android/show-43668.html)
