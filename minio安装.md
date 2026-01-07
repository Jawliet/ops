# MinIO 单节点单硬盘部署（完整简化版）

## 1. 下载 MinIO

### Intel/AMD 64位

```bash
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```

### ARM 64位

```bash
wget https://dl.min.io/server/minio/release/linux-arm64/minio
chmod +x minio
sudo mv minio /usr/local/bin/
```

## 2. 创建用户和目录

```bash
sudo groupadd -r minio-user
sudo useradd -M -r -g minio-user minio-user
sudo mkdir -p /mnt/data
sudo chown minio-user:minio-user /mnt/data
```

## 3. 创建 systemd 服务文件

创建 `/etc/systemd/system/minio.service`：

```ini
[Unit]
Description=MinIO
Documentation=https://min.io/docs/minio/linux/index.html
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio

[Service]
WorkingDirectory=/usr/local

User=minio-user
Group=minio-user
ProtectProc=invisible

EnvironmentFile=-/etc/default/minio
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /etc/default/minio\"; exit 1; fi"
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

Restart=always
LimitNOFILE=65536
TasksMax=infinity
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```

## 4. 配置环境变量

创建 `/etc/default/minio`：

```bash
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=password
MINIO_VOLUMES="/mnt/data"
MINIO_OPTS="--console-address :9001"
```

## 5. 启动服务

```bash
sudo systemctl daemon-reload
sudo systemctl start minio.service
sudo systemctl enable minio.service
sudo systemctl status minio.service
```

## 6. 访问 MinIO

- **控制台**：[http://localhost:9001](http://localhost:9001/)

- **API 地址**：[http://localhost:9000](http://localhost:9000/)

- **用户名/密码**：admin/password

## 7. mc 配置

### 安装 mc

将 **mc** 命令行工具安装到主机上。 点击 与主机操作系统或环境对应的选项卡：

Linux

以下命令将为您的系统 PATH 添加一个 *临时* 扩展，以运行 `mc` 实用程序。如果您想对系统 PATH 进行永久性修改， 请参考您的操作系统说明。

或者，您可以导航到父文件夹并运行 `./mc --help` 来执行 `mc` 。

**64-bit Intel**

```bash
curl https://dl.minio.org.cn/client/mc/release/linux-amd64/mc \
 --create-dirs \
 -o $HOME/minio-binaries/mc
chmod +x $HOME/minio-binaries/mc
export PATH=$PATH:$HOME/minio-binaries/
mc --help
```

**64-bit PPC**

```bash
curl https://dl.minio.org.cn/client/mc/release/linux-ppc64le/mc \
 --create-dirs \
 -o ~/minio-binaries/mc
chmod +x $HOME/minio-binaries/mc
export PATH=$PATH:$HOME/minio-binaries/
```

**ARM64**

```bash
curl https://dl.minio.org.cn/client/mc/release/linux-arm64/mc \
 --create-dirs \
 -o ~/minio-binaries/mc
chmod +x $HOME/minio-binaries/mc
export PATH=$PATH:$HOME/minio-binaries/
```

### 添加远端 alias（一次性配置）

```bash
# 语法
mc alias set <别名> <HOST:PORT> <ACCESS_KEY> <SECRET_KEY> [--api s3v4]

# 例子：公网 HTTPS 端口 9000
mc alias set minio https://minio.example.com:9000 AKIAIOSFODNN7EXAMPLE wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# 例子：内网 HTTP 端口 9000
mc alias set minio http://192.168.1.50:9000 minioadmin minioadmin
```

成功后会写入 `~/.mc/config.json`，之后在任何机器上只要把该文件拷过去就能复用别名。

### 常见“匿名”权限对照表

| 原 policy 关键字 | 新 anonymous 命令              | 说明                |
| ------------ | --------------------------- | ----------------- |
| `none`       | `mc anonymous set none`     | 完全私有，匿名 403       |
| `download`   | `mc anonymous set download` | 允许匿名读文件，但禁止列表     |
| `upload`     | `mc anonymous set upload`   | 允许匿名上传，不可下载       |
| `public`     | `mc anonymous set public`   | 匿名可列表 + 读 + 写，最开放 |

### “拒绝 ListBucket，允许 GetObject”的自定义匿名策略

1. 本地新建文件 `readonly-nolist.json`

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAnonymousGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::monitor/*"
    },
    {
      "Sid": "DenyAnonymousListBucket",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::monitor"
    }
  ]
}
```

2. 贴到桶上

```bash
mc anonymous set-json readonly-nolist.json minio/monitor
```

要点说明

- **Deny 优先**：Allow 放上面，Deny 放下面，显式拒绝列表。

- **Resource 区分**
  
  - `arn:aws:s3:::monitor/*` 代表“对象”
  
  - `arn:aws:s3:::monitor` 代表“桶本身（列表接口）”

- 这条策略只影响匿名请求；带 ak/sk 或预签名的调用完全不受限。

