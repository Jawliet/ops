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
    
# 文件权限问题： minio.sh: line 270: /data/.root_user: Permission denied
# --user $(id -u):$(id -g)  解决文件权限问题 ,需要创建用户，所以不使用
# chown -R 1001:1001 /data/minio 或者先修改文件夹权限 ，再创建容器
# user & password 分别是 登陆用户名和密码
# web登陆： IP+9001
