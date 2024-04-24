# yum 安装 mongodb

## 安装 mongodb

1. 配置 yum Repository

   ```ini
   [mongodb-org-7.0]
   name=MongoDB Repository
   baseurl=https://repo.mongodb.org/yum/redhat/7/mongodb-org/7.0/x86_64/
   gpgcheck=1
   enabled=1
   gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
   ```

   或者

   ```bash
   yum install -y https://repo.mongodb.org/yum/redhat/7/mongodb-org/7.0/x86_64/RPMS/mongodb-org-server-7.0.5-1.el7.x86_64.rpm
   ```

2. 安装

   ```bash
   install -y mongodb-org
   ```

## 修改默认存储目录

1. 创建目录

   ```bash
   mkdir -p /mnt/mongodb/{data,log}
   chown -R mongod:mongod /mnt/mongodb
   ```

2. 配置 SELinux

   ```bash
   yum install policycoreutils-python
   
   semanage fcontext -a -t mongod_var_lib_t /mnt/mongodb/data.*
   semanage fcontext -a -t mongod_log_t /mnt/mongodb/log.*
   
   
   chcon -Rv -u system_u -t mongod_var_lib_t /mnt/mongodb/data
   chcon -Rv -u system_u -t mongod_log_t /mnt/mongodb/log
   
   
   restorecon -R -v /data/mongodb/data
   restorecon -R -v /data/mongodb/log
   ```

3. 修改 `/etc/mongod.conf`中的`storage.dbPath`和`systemLog.path`

   ```yaml
   # mongod.conf
   
   # for documentation of all options, see:
   #   http://docs.mongodb.org/manual/reference/configuration-options/
   
   # where to write logging data.
   systemLog:
     destination: file
     logAppend: true
   #  path: /var/log/mongodb/mongod.log # 修改前
     path: /mnt/mongodb/log/mongod.log  # 修改后
   
   # Where and how to store data.
   storage:
   #  dbPath: /var/lib/mongo # 修改前
     dbPath: /mnt/mongodb/data # 修改后
   
   # how the process runs
   processManagement:
     timeZoneInfo: /usr/share/zoneinfo
   
   # network interfaces
   net:
     port: 27017
     bindIp: 127.0.0.1  # Enter 0.0.0.0,:: to bind to all IPv4 and IPv6 addresses or, alternatively, use the net.bindIpAll setting.
   
   
   #security:
   
   #operationProfiling:
   
   #replication:
   
   #sharding:
   
   ## Enterprise-Only Options
   
   #auditLog:
   
   ```

4. 启动 mongodb

   ```bash
   systemctl start mongod
   ```
