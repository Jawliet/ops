## MongoDB 搭建

### 部署机器

| 主机名     | 内网IP           |
|:-------:|:--------------:|
| node121 | 192.168.12.121 |
| node122 | 192.168.12.122 |
| node123 | 192.168.12.123 |

### 安装

1. 开放端口
   
   | 协议  | 端口    | 用途                                     |
   |:---:|:-----:|:--------------------------------------:|
   | TCP | 27017 | mongos-router:27017                    |
   | TCP | 27018 | config-server:27018                    |
   | TCP | 27019 | shard1:27019 shard2:27020 shard3:27021 |
   | TCP | 27020 | shard1:27019 shard2:27020 shard3:27021 |
   | TCP | 27021 | shard1:27019 shard2:27020 shard3:27021 |

2. 集群目录
   
   ```bash
   mkdir -p /home/mongodb/cluster/{config,mongos,shard1,shard2,shard3}/{data,logs}
   ```
   
   - `/home/mongodb` 安装目录
   
   - `/home/mongodb/cluster` 集群组件数据目录

3. [下载安装](https://www.mongodb.com/try/download/community?tck=docs_server)
   
   - 安装mongodb tar包所需依赖
     
     ```bash
     yum install -y libcurl openssl xz-libs
     ```
   
   - 下载 MongoDB 二进制包，并解压到 `/home/mongodb`
     
     ```bash
     wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.4.10.tgz
     tar -zxvf mongodb-linux-x86_64-* -C /home/mongodb --strip=1
     ```
   
   - 配置环境变量
     
     ```bash
     echo "PATH=$PATH:/home/mongodb/bin" > /etc/profile.d/mongodb.sh
     source /etc/profile.d/mongodb.sh
     
     mongo --version
     ```
   
   - 创建 MongoDB 用户
     
     ```bash
     groupadd mongod
     useradd -g mongod mongod
     ```
   
   - 配置目录权限
     
     ```bash
     chown -R mongod:mongod /home/mongodb
     ```
   
   - 所有节点安装 rsync，方便分发配置文件
     
     ```bash
     yum install -y rsync
     ```

4. 修改对应组件配置文件
   
   以下所有操作在第一个节点node121执行
   
   - 下载并分发配置文件模板到各个目录
     
     ```bash
     wget https://raw.githubusercontent.com/mongodb/mongo/master/rpm/mongod.conf -P /home/mongodb/
     realpath /home/mongodb/cluster/* | xargs -I {} cp /home/mongodb/mongod.conf {}
     ```
   
   - 修改configServer配置文件
     
     ```shell
     cat > /home/mongodb/cluster/config/mongod.conf <<EOF
     systemLog:
       destination: file
       logAppend: true
       path: /home/mongodb/cluster/config/logs/mongod.log
     
     # Where and how to store data.
     storage:
       dbPath: /home/mongodb/cluster/config/data
       journal:
         enabled: true
     
     # how the process runs
     processManagement:
       fork: true  # fork and run in background
       pidFilePath: /home/mongodb/cluster/config/mongod.pid  # location of pidfile
       timeZoneInfo: /usr/share/zoneinfo
     
     # network interfaces
     net:
       port: 27018
       bindIp: node121
     
     sharding:
       clusterRole: configsvr
     
     replication:
       replSetName: config
     EOF
     ```
   
   - 修改shard1分片配置文件
     
     ```shell
     cat > /home/mongodb/cluster/shard1/mongod.conf <<EOF
     systemLog:
       destination: file
       logAppend: true
       path: /home/mongodb/cluster/shard1/logs/mongod.log
     
     # Where and how to store data.
     storage:
       dbPath: /home/mongodb/cluster/shard1/data
       journal:
         enabled: true
     
     # how the process runs
     processManagement:
       fork: true  # fork and run in background
       pidFilePath: /home/mongodb/cluster/shard1/mongod.pid  # location of pidfile
       timeZoneInfo: /usr/share/zoneinfo
     
     # network interfaces
     net:
       port: 27019
       bindIp: node121
     
     sharding:
         clusterRole: shardsvr
     
     replication:
         replSetName: shard1
     EOF
     ```
   
   - 修改shard2分片配置文件
     
     ```bash
     cat > /home/mongodb/cluster/shard2/mongod.conf <<EOF
     systemLog:
       destination: file
       logAppend: true
       path: /home/mongodb/cluster/shard2/logs/mongod.log
     
     # Where and how to store data.
     storage:
       dbPath: /home/mongodb/cluster/shard2/data
       journal:
         enabled: true
     
     # how the process runs
     processManagement:
       fork: true  # fork and run in background
       pidFilePath: /home/mongodb/cluster/shard2/mongod.pid  # location of pidfile
       timeZoneInfo: /usr/share/zoneinfo
     
     # network interfaces
     net:
       port: 27020
       bindIp: node121
     
     sharding:
         clusterRole: shardsvr
     
     replication:
         replSetName: shard2
     EOF
     ```
   
   - 修改shard3分片配置文件
     
     ```shell
     cat > /home/mongodb/cluster/shard3/mongod.conf <<EOF
     systemLog:
       destination: file
       logAppend: true
       path: /home/mongodb/cluster/shard3/logs/mongod.log
     
     # Where and how to store data.
     storage:
       dbPath: /home/mongodb/cluster/shard3/data
       journal:
         enabled: true
     
     # how the process runs
     processManagement:
       fork: true  # fork and run in background
       pidFilePath: /home/mongodb/cluster/shard3/mongod.pid  # location of pidfile
       timeZoneInfo: /usr/share/zoneinfo
     
     # network interfaces
     net:
       port: 27021
       bindIp: node121
     
     sharding:
         clusterRole: shardsvr
     
     replication:
         replSetName: shard3
     EOF
     ```
   
   - 修改mongos-router配置文件
     
     ```shell
     cat > /home/mongodb/cluster/mongos/mongod.conf <<EOF
     systemLog:
       destination: file
       logAppend: true
       path: /home/mongodb/cluster/mongos/logs/mongod.log
     
     # how the process runs
     processManagement:
       fork: true  # fork and run in background
       pidFilePath: /home/mongodb/cluster/mongos/mongod.pid  # location of pidfile
       timeZoneInfo: /usr/share/zoneinfo
     
     # network interfaces
     net:
       port: 27017
       bindIp: node121
     
     sharding:
       configDB: config/node121:27018,node122:27018,node123:27018
     EOF
     ```
   
   - 分发配置到各个节点
     
     ```bash
     rsync -avzp /home/mongodb/cluster node122:/home/mongodb/
     rsync -avzp /home/mongodb/cluster node123:/home/mongodb/
     ```
   
   - 在 node122, node123 节点更新配置文件绑定地址
     
     ```bash
     grep -rl bindIp /home/mongodb/cluster/ | xargs sed -i 's#bindIp: node121#bindIp: node122#g'
     
     grep -rl bindIp /home/mongodb/cluster/ | xargs sed -i 's#bindIp: node121#bindIp: node123#g'
     ```

5. 启动并初始化 configserver
   
   - 3个节点执行以下命令分别启动 configServer
     
     ```bash
     mongod -f /home/mongodb/cluster/config/mongod.conf
     ```
   
   - 连接到第一个节点 node121
     
     ```bash
     mongo --host node121 --port 27018
     ```
   
   - 初始化 configServer 副本集
     
     ```bash
     rs.initiate(
       {
         _id: "config",
         configsvr: true,
         members: [
           { _id : 0, host : "node121:27018" },
           { _id : 1, host : "node122:27018" },
           { _id : 2, host : "node123:27018" }
         ]
       }
     )
     ```

6. 启动并初始化shard分片
   
   - 3个节点每个节点执行以下命令启动3个shard分片副本集
     
     ```bash
     mongod -f /home/mongodb/cluster/shard1/mongod.conf
     mongod -f /home/mongodb/cluster/shard2/mongod.conf
     mongod -f /home/mongodb/cluster/shard3/mongod.conf
     ```
   
   - 连接到第一个shard分片并初始化
     
     ```bash
     mongo --host node121 --port 27019
     
     rs.initiate(
       {
         _id: "shard1",
         members: [
           { _id : 0, host : "node121:27019" },
           { _id : 1, host : "node122:27019" },
           { _id : 2, host : "node123:27019" }
         ]
       }
     )
     ```
   
   - 连接到第二个shard分片并初始化
     
     ```bash
     mongo --host node121 --port 27020
     
     rs.initiate(
       {
         _id: "shard2",
         members: [
           { _id : 0, host : "node121:27020" },
           { _id : 1, host : "node122:27020" },
           { _id : 2, host : "node123:27020" }
         ]
       }
     )
     ```
   
   - 连接到第三个shard分片并初始化
     
     ```bash
     mongo --host node121 --port 27021
     
     rs.initiate(
       {
         _id: "shard3",
         members: [
           { _id : 0, host : "node121:27021" },
           { _id : 1, host : "node122:27021" },
           { _id : 2, host : "node123:27021" }
         ]
       }
     )
     ```

7. 启动并初始化 mongos
   
   - 3个节点启动 mongos-router
     
     ```bash
     mongos -f /home/mongodb/cluster/mongos/mongod.conf
     ```
   
   - 连接到第一个 mongos 节点 node121 并初始化
     
     ```bash
     mongo --host node121 --port 27017
     
     sh.addShard( "shard1/node121:27019,node122:27019,node123:27019")
     sh.addShard( "shard2/node121:27020,node122:27020,node123:27020")
     sh.addShard( "shard3/node121:27021,node122:27021,node123:27021")
     ```
### 启用认证
   对副本集执行访问控制需要配置两个方面:
   - 副本集和共享集群的各个节点成员之间使用内部身份验证，可以使用密钥文件或x.509证书。密钥文件比较简单，本文介绍的也是使用密钥文件，官方推荐如果是测试环境可以使用密钥文件，但是正是环境，官方推荐x.509证书。原理就是，集群中每一个实例彼此连接的时候都检验彼此使用的证书的内容是否相同。只有证书相同的实例彼此才可以访问
   - 使用客户端连接到mongodb集群时，开启访问授权。对于集群外部的访问。如通过可视化客户端，或者通过代码连接的时候，需要开启授权。

下面开始详细说明：

   1. 生成密钥文件
      在keyfile身份验证中，副本集中的每个mongod实例都使用keyfile的内容作为共享密码，只有具有正确密钥文件的mongod或者mongos实例可以连接到副本集。密钥文件的内容必须在6到1024个字符之间，并且在unix/linux系统中文件所有者必须有对文件至少有读的权限。 可以用任何方式生成密钥文件例如：
   ```bash
      openssl rand -base64 756 > /data/mongodb/testKeyFile.file
      chmod 400 /data/mongodb/keyfile/testKeyFile.file
      chmod -R mongod:mongod /data/mongodb/keyfile/
   ```
      第一条命令是生成密钥文件，第二条命令是使用chmod更改文件权限，为文件所有者提供读权限
   2. 将密钥复制到集群中的每台机器（82，83，86）的指定位置
   ```bash scp -P22 /data/mongodb/testKeyFile.file root@10.12.40.86:/data/mongodb ```
    > 一定要保证密钥文件一致。文件位置随便。但是为了方便查找，建议每台机器都放到一个固定的位置。我的配置文件都放在`/data/mongodb/testKeyFile.file`
   3. 预先创建好一个管理员账号和密码然后将集群中的所有mongod和mongos全部关闭账号可以在集群认开启认证以后添加。但是那时候添加比较谨慎。只能添加一次，如果忘记了就无法再连接到集群。建议在没开启集群认证的时候先添加好管理员用户名和密码然后再开启认证再重启
   - 连接任意一台机器的mongos
      ```bash mongo --port 23000```
   - 添加用户
      ```shell
      use admin   //注意一定要使用admin数据库 
      db.createUser({user:"your account",pwd:"your password",roles:[{role:"root",db:"admin"}]})
      ```
   - 然后依次连接到每一台机器上执行。
      ```bash
      killall mongod 
      killall mongos
      ```
   > 然后删除每个mongod实例存储数据存储路径下面的mongod.lock（如果后面启动不报错可以不处理）可以发现。集群多少有的节点都关闭了。没开启认证的集群如果开启认证需要集群宕机几分钟。当然也有热启动的方式，官方文档中有介绍说明：可以先开启认证重启后再添加用户。但是只能在admin库添加一次，所以如果忘记了，或者权限分配不恰当就无法再更改，所以建议先添加用户再开启认证重启，并且集群不建议在每个单节点添加用户，并且建议单节点关闭初始添加账号的权限，详情见enableLocalhostAuthBypass)

   3. 使用访问控制强制重新启动复制集的每个成员这个步骤比较重要。设置访问控制有两种方式。我选择在配置文件里面配置好。（也可以在启动命令时使用命令来指定）

   4. 依次在每台机器上的mongod（注意是所有的mongod不是mongos）的配置文件中加入下面一段配置。如我在10.12.40.83上的config server，shard1，shard2，shard3都加入下面的配置文件
   ```yaml
   security:   
     keyFile: /data/mongodb/testKeyFile.file   
     authorization: enabled
   ```
   依次在每台机器上的mongos配置文件中加入下面一段配置。如我在10.12.40.83上的mongos配置文件中加入上面的一段配置
   ```yaml
   security:   keyFile: /data/mongodb/testKeyFile.file
   ```
   > 解释：mongos比mongod少了authorization：enabled的配置。原因是，副本集加分片的安全认证需要配置两方面的，副本集各个节点之间使用内部身份验证，用于内部各个mongo实例的通信，只有相同keyfile才能相互访问。所以都要开启keyFile: `/data/mongodb/testKeyFile.file`.然而对于所有的mongod，才是真正的保存数据的分片。mongos只做路由，不保存数据。所以所有的mongod开启访问数据的授权authorization:enabled。这样用户只有账号密码正确才能访问到数据

   5. 重启每个mongo示例。因为我的认证配置在了配置文件里面，所以启动命令不需要再加认证的参数 (例如--auth等)
   ```bash
   mongod -f /data/mongodb/config/configs.config 
   mongod -f /data/mongodb/config/shard1.config 
   mongod -f /data/mongodb/config/shard2.config 
   mongod -f /data/mongodb/config/shard3.config 
   mongos -f /data/mongodb/config/mongos.config
   ```
   依次重启三台机器的mongod和mongos实例
