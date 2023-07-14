## 3. MongoDB 搭建

### 3.1 部署机器

| 主机名     | 内网IP           |
|:-------:|:--------------:|
| node121 | 192.168.12.121 |
| node122 | 192.168.12.122 |
| node123 | 192.168.12.123 |

### 3.2 安装

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
