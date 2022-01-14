# 阿里云 OSS-HDFS 服务（JindoFS 服务）快速入门

JindoFS服务是阿里云推出新的存储空间类型，兼容HDFS接口, 支持目录以及目录层级，JindoSDK 为 Apache Hadoop的计算分析应用（例如MapReduce、Hive、Spark、Flink等）提供了访问JindoFS服务功能。

## 服务开通

### 1. 权限设置

#### 1.1. OSS 服务账号授权

首次使用 JindoFS 服务功能时，需要先在 [RAM 控制台](https://ram.console.aliyun.com/overview) 完成以下授权，以便 OSS 服务账号能够管理 Bucket 中的数据块。

#### a. 新建名为AliyunOSSDlsDefaultRole的角色

在创建角色对话框里面选择可信实体类型为“阿里云服务”，点击下一步；
角色类型选择“普通服务角色”，角色名称固定为“AliyunOSSDlsDefaultRole”，备注可以为“DLS默认使用此角色来访问用户数据”，受信服务选择“对象存储”。

#### b. 新建名为AliyunOSSDlsRolePolicy的权限策略

该权限策略的内容为：
```json
{
  "Version": "1",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "oss:ListObjects",
      "Resource": [
        "acs:oss:*:*:*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "oss:*",
      "Resource": [
        "acs:oss:*:*:*/.dlsdata",
        "acs:oss:*:*:*/.dlsdata*"
      ]
    }
  ]
}

```

#### c. 对新建的角色AliyunOSSDlsDefaultRole进行授权

对刚创建的角色AliyunOSSDlsDefaultRole进行“精确授权”，权限类型为自定义策略，策略名称为：“AliyunOSSDlsRolePolicy”

#### 1.2. 客户端授权访问 JindoFS服务

注意：如您使用 子账号 或者 服务角色（如 [EMR 服务角色](https://help.aliyun.com/document_detail/28072.html)）的方式访问 JindoFS服务，客户端 AK 或者 服务角色 需要以下策略，设置方式可参考[《通过RAM对OSS进行权限管理》](https://help.aliyun.com/document_detail/58905.html)。

```json
{
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "oss:*",
            "Resource": [
                "acs:oss:*:*:*/.dlsdata",
                "acs:oss:*:*:*/.dlsdata*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "oss:GetBucketInfo",
                "oss:PostDataLakeStorageFileOperation"
            ],
            "Resource": "*"
        }
    ],
    "Version": "1"
}
```

### 2. 下载 JindoSDK 包
下载最新的 tar.gz 包 jindosdk-x.x.x.tar.gz ([下载页面](/docs/user/4.x/jindodata_download.md))。

### 3. 配置环境变量
* 配置`JINDOSDK_HOME`

解压下载的安装包，以安装包内容解压在`/usr/lib/jindosdk-4.0.0`目录为例：
```bash
export JINDOSDK_HOME=/usr/lib/jindosdk-4.0.0
```
* 配置`HADOOP_CLASSPATH`

```bash
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:${JINDOSDK_HOME}/lib/*
```

### 4. 配置 JindoFS 服务实现类及 Access Key

将 JindoSDK OSS 实现类配置到 Hadoop 的`core-site.xml`中。

```xml
<configuration>
    <property>
        <name>fs.AbstractFileSystem.oss.impl</name>
        <value>com.aliyun.jindodata.oss.JindoOSS</value>
    </property>

    <property>
        <name>fs.oss.impl</name>
        <value>com.aliyun.jindodata.oss.JindoOssFileSystem</value>
    </property>
</configuration>
```
将已开启 HDFS 服务的 Bucket 对应的`Access Key ID`、`Access Key Secret`等预先配置在 Hadoop 的`core-site.xml`中。
```xml
<configuration>
    <property>
        <name>fs.oss.accessKeyId</name>
        <value>xxx</value>
    </property>

    <property>
        <name>fs.oss.accessKeySecret</name>
        <value>xxx</value>
    </property>
</configuration>
```
JindoSDK 还支持更多的 AccessKey 的配置方式，详情参考 [JindoSDK Credential Provider 配置](security/jindosdk_credential_provider.md)。

### 5. 配置 JindoFS 服务 Endpoint
访问 OSS Bucket 上 JindoFS 服务需要配置 Endpoint（`cn-xxx.oss-dls.aliyuncs.com`），与 OSS 对象接口的 Endpoint（`oss-cn-xxx.aliyuncs.com`）不同。JindoSDK 会根据配置的 Endpoint 访问 JindoFS 服务 或 OSS 对象接口。

使用 JindoFS 服务时，推荐访问路径格式为：`oss://<Bucket>.<Endpoint>/<Object>`

如: `oss://dls-chenshi-test.cn-shanghai.oss-dls.aliyuncs.com/Test`。

这种方式在访问路径中包含 JindoFS 服务的 Endpoint，JindoSDK 会根据路径中的 Endpoint 访问对应的 JindoFS 服务接口。 JindoSDK 还支持更多的 Endpoint 配置方式，详情参考[JindoFS 服务 Endpoint 配置](configuration/jindosdk_endpoint_configuration.md)。

# 基本操作示例
JindoFS服务创建以及配置完成后，可以通过hdfs dfs 命令进行相关文件/目录操作
### 新建目录
在JindoFS服务上创建目录
<br>用例: `hdfs dfs -mkdir oss://<Bucket>.<Endpoint>/Test/subdir`

```shell
[root@emr-header-1 ~]# hdfs dfs -mkdir oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/subdir
[root@emr-header-1 ~]# hdfs dfs -ls oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test
Found 1 items
drwxr-x--x   - root supergroup          0 2021-12-01 20:19 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/subdir

```

### 新建文件
利用hdfs dfs -put命令上传本地文件到JindoFS服务
<br> 用例：`hdfs dfs -put <localfile> oss://<Bucket>.<Endpoint>/Test`
```shell
[root@emr-header-1 ~]# hdfs dfs -put /etc/hosts oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/
[root@emr-header-1 ~]# hdfs dfs -ls oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test
Found 2 items
-rw-r-----   1 root supergroup       5824 2021-12-01 20:24 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/hosts
drwxr-x--x   - root supergroup          0 2021-12-01 20:19 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/subdir

```
### 查看文件或者目录信息
在文件或者目录创建完之后，可以查看指定路径下的文件/目录信息。hdfs dfs没有进入某个目录下的概念。在查看目录和文件的信息的时候需要给出文件/目录的绝对路径。
<br>指令：ls
<br>用例：`hdfs dfs -ls oss://<Bucket>.<Endpoint>/Test`
```shell
[root@emr-header-1 ~]# hdfs dfs -ls oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test
Found 2 items
-rw-r-----   1 root supergroup       5824 2021-12-01 20:24 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/hosts
drwxr-x--x   - root supergroup          0 2021-12-01 20:19 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/subdir

```

### 查看文件或者目录的大小
查看已有文件或者目录的大小
<br>用例：`hdfs dfs -du oss://<Bucket>.<Endpoint>/Test`
```shell
[root@emr-header-1 ~]# hdfs dfs -du oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test
5824  oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/hosts
0     oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/subdir

```

### 查看文件的内容
有时候我们需要查看一下在JindoFS服务文件的内容。hdfs dfs命令支持我们将文件内容打印在屏幕上。（请注意，文件内容将会以纯文本形式打印出来，如果文件进行了特定格式的编码，请使用HDFS的JavaAPI将文件内容读取并进行相应的解码获取文件内容）
<br>用例：`hdfs dfs -cat oss://<Bucket>.<Endpoint>/Test/helloworld.txt`

```shell
[root@emr-header-1 ~]# hdfs dfs -cat  oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/helloworld.txt
hello world!
```

### 复制目录/文件
有时候我们需要将JindoFS服务的一个文件/目录拷贝到另一个位置，并且保持源文件和目录结构和内容不变。
<br>用例：`hdfs dfs -cp oss://<Bucket>.<Endpoint>/Test/subdir oss://<Bucket>.<Endpoint>/TestTarget/sudir2`
```shell
[root@emr-header-1 ~]# hdfs dfs -cp oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/subdir oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/TestTarget/subdir1
[root@emr-header-1 ~]# hdfs dfs -ls  oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/TestTarget/
Found 1 items
drwxr-x--x   - root supergroup          0 2021-12-01 20:37 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/TestTarget/subdir1

```

### 移动目录/文件
在很多大数据处理的例子中，我们会将文件写入一个临时目录，然后将该目录移动到另一个位置作为最终结果。源文件和目录结构和内容不做保留。下面的命令可以完成这些操作。
<br>用例：`hdfs dfs -mv oss://<Bucket>.<Endpoint>/Test/subdir oss://<Bucket>.<Endpoint>/Test/subdir1`

```shell
[root@emr-header-1 ~]# hdfs dfs -mv  oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/subdir  oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/newdir
[root@emr-header-1 ~]# hdfs dfs -ls  oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test
Found 3 items
-rw-r-----   1 root supergroup         13 2021-12-01 20:33 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/helloworld.txt
-rw-r-----   1 root supergroup       5824 2021-12-01 20:24 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/hosts
drwxr-x--x   - root supergroup          0 2021-12-01 20:19 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/newdir

```

### 下载文件到本地文件系统
某些情况下，我们需要将OSS文件系统中的某些文件下载到本地，再进行处理或者查看内容。这个可以用下面的命令完成。
<br>用例：`hdfs dfs -get oss://<Bucket>.<Endpoint>/Test/helloworld.txt <localpath>`
```shell
[root@emr-header-1 ~]# hdfs dfs -get oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/helloworld.txt /tmp/
[root@emr-header-1 ~]# ll /tmp/helloworld.txt
-rw-r----- 1 root root 13 12月  1 20:44 /tmp/helloworld.txt

```

### 删除目录/文件
在很多情况下，我们在完成工作后，需要删除在JindoFS服务上的某些临时文件或者废弃文件。这些可以通过下面的命令完成。
<br>用例：`hdfs dfs -rm oss://<Bucket>.<Endpoint>/Test/helloworld.txt`
```shell
[root@emr-header-1 ~]# hdfs dfs -rm oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/helloworld.txt
21/12/01 20:46:44 INFO fs.TrashPolicyDefault: Moved: 'oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/helloworld.txt' to trash at: oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/user/root/.Trash/Current/Test/helloworld.txt

[root@emr-header-1 ~]# hdfs dfs -rm -r oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/newdir
21/12/01 20:47:16 INFO fs.TrashPolicyDefault: Moved: 'oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/newdir' to trash at: oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/user/root/.Trash/Current/Test/newdir

[root@emr-header-1 ~]# hdfs dfs -ls oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/
Found 1 items
-rw-r-----   1 root supergroup       5824 2021-12-01 20:24 oss://dls-chenshi-test.cn-xxx.oss-dls.aliyuncs.com/Test/hosts

```

除了上述基本操作之外，JindoFS服务还支持Acl，attr/xattr，snapshot, checksum等功能。