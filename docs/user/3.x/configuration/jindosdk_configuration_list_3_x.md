# JindoSDK 高级参数配置

## 创建并使用配置文件

将下面环境变量添加到/etc/profile文件中
```
export B2SDK_CONF_DIR=/etc/jindo-conf
```
创建文件 /etc/jindo-conf/bigboot.cfg  包含以下基本内容
```
[bigboot]
logger.dir = /tmp/bigboot-log

[bigboot-client]
```

该文件会使用2个section，其中bigboot是日志相关配置，bigboot-client是客户端相关配置。

(注，如不便修改环境变量，可尝试在默认路径创建配置文件 `/usr/lib/b2smartdata-current/conf/bigboot.cfg`)

## SDK配置项列表

以下是重要的配置项列表
| 配置项                                          | section          | 默认值           | 说明                                                           |
| ---------------------------------------------- | ---------------- | ---------------- | --------------------------------------------------------------|
| logger.dir                                     | [bigboot]        | /tmp/bigboot-log | 日志目录                                                       |
| logger.sync                                    | [bigboot]        | false            | 是否同步flush日志文件                                           |
| client.storage.rpc.port                        | [bigboot-client] | 6101             | 本地storage服务进程端口号                                       |
| client.namespace.rpc.address                   | [bigboot-client] | localhost:8101   | 访问JindoFS集群的namespace服务地址<br />（block/cache模式必填）  |
| client.oss.retry                               | [bigboot-client] | 5                | 访问oss失败重试次数                                             |
| client.oss.timeout.millisecond                 | [bigboot-client] | 30000            | 请求oss超时时间（毫秒）                                         |
| client.oss.connection.timeout.millisecond      | [bigboot-client] | 3000             | 连接oss超时时间（毫秒）                                         |
| client.oss.upload.thread.concurrency           | [bigboot-client] | 5                | 进程内oss并发上传的最大线程数                                    |
| client.oss.upload.queue.size                   | [bigboot-client] | 5                | oss并发上传任务队列大小                                         |
| client.oss.download.thread.concurrency         | [bigboot-client] | 5                | 进程内oss并发下载的最大线程数                                   |
| client.oss.download.queue.size                 | [bigboot-client] | 5                | oss并发下载任务队列大小                                         |
| client.read.oss.readahead.buffer.size          | [bigboot-client] | 1048576          | 预读oss的buffer大小                                             |
| client.read.oss.readahead.buffer.count         | [bigboot-client] | 4                | 同时预读oss的buffer个数                                         |
| jfs.cache.data-cache.enable                    | [bigboot-client] | false            | (仅用于cache模式)cache模式开启缓存功能                          |
| client.temp-data-dirs                               | [bigboot-client] | /tmp            | 客户端写入时的临时文件目录，可配置多个（逗号隔开），会轮流写入  |

