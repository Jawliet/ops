```bash
mkdir -p /data/minio
chown -R 1001:1001 /data/minio 
docker run --name minio \
    --publish 9000:9000 \
    --publish 9001:9001 \
    --volume /data/minio:/data \
    --env MINIO_ROOT_USER="admin" \
    --env MINIO_ROOT_PASSWORD="123456" \
    --privileged=true \
    bitnami/minio:latest
```
> 文件权限问题： minio.sh: line 270: /data/.root_user: Permission denied
> ```bash
>  chown -R 1001:1001 /data/minio
> ```
