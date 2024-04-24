# [TDengine 部署](https://www.taosdata.com/cn/documentation/cluster)

## 1.1 部署机器

| 主机名     | 内网IP           |
|:-------:|:--------------:|
| node121 | 192.168.12.121 |
| node122 | 192.168.12.122 |
| node123 | 192.168.12.123 |

## 2.2 安装

1. 开放端口
   
   ```bash
   firewall-cmd --add-port=6030-6049/tcp \
                --add-port=6030-6049/udp --permanent
   firewall-cmd --reload
   ```

2. 将`TDengine-server-2.4.0.18-Linux-x64-Lite.tar.gz`上传到`/home`

3. 安装
   
   ```bash
   tar -xzvf TDengine-server-2.4.0.18-Linux-x64-Lite.tar.gz -O > TDengine-server
   cd TDengine-server
   ./install.sh -e no
   ```

4. 配置
   
   分别则三个节点执行操作
   
   1. 创建存储路径
      
      ```bash
      mkdir -p /home/taos/{data,log} 
      ```
   
   2. 配置 firstEp 为 node121
      
      ```bash
      sed -i 's/# firstEp                   hostname:6030/firstEp                   node121:6030/' /etc/taos/taos.cfg
      ```
   
   3. 配置 fqdn 为当前节点
      
      ```bash
      sed -i 's/# fqdn                      hostname/fqdn                      node121/' /etc/taos/taos.cfg
      ```
   
   4. 配置存储路径
      
      ```bash
      # 日志
      sed -i 's/# logDir                    \/var\/log\/taos/logDir                    \/home\/taos\/log/taos/' /etc/taos/taos.cfg
      cp -rp /var/log/taos /home/taos/log/
      # 数据
      sed -i 's/# dataDir                   \/var\/lib\/taos/dataDir                   \/home\/taos\/data/taos/' /etc/taos/taos.cfg
      cp -rp /var/lib/taos /home/taos/data/
      ```

5. 集群配置
   
   1. 启动第一个节点，执行 taos, 启动 taos shell
      
      ```bash
      systemctl start taosd
      ```
   
   2. 启动其它两个节点，通过第一个节点的 taos shell 添加到集群
      
      ```shell
      CREATE DNODE "node122:6030";
      CREATE DNODE "node123:6030"; 
      ```
      
      可以通过`SHOW DNODES;`查看节点
