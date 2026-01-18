# Redis

* 特征
  * 键值型
  * 单线程、命令具备原子性
  * 低延迟、速度快（**基于内存**、IO多路复用、编码良好）
  * 支持数据持久化
  * 支持主从集群、分片集群
  * 多语言客户端

# Redis 命令行客户端

* 连接：`redis-cli [options] [commands]`
* 常见  options
  * `-h 127.0.0.1`：指定连接的 redis 节点 IP 地址，默认127.0.0.1
  * `-p 6379`：指定要连接的 redis 节点的端口，默认 6379
  * `-a 123456`：指定 redis 的访问密码
* commands：Redis 的操作命令
  * `ping`：心跳链接，正常则返回 pong
* `help @`：命令文档
* `help [command]`：查看命令具体用法

# Redis 命令

* 基础数据类型：String、Hash、List、Set、ZSet

## 通用命令

* KEYS ：查看符合模板的所有 key（会扫描所有 key 生产环境不要用）
* DEL：删除指定的 key
* EXISTS：判断 key 是否存在
* TYPE：查看 key 类型
* EXPIRE：给 key 设置有效期
* TTL：查看 key 的剩余有效期
* PERSIST：移除过期时间
* RENAME：重命名 key
