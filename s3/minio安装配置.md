## 安装

```yaml
services:
  minio:
    image: quay.io/minio/minio:latest
    container_name: minio
    restart: unless-stopped
    ports:
      - "9000:9000"   # S3 API
      - "9001:9001"   # MinIO Console
    environment:
      MINIO_ROOT_USER: "user"
      MINIO_ROOT_PASSWORD: "password"
    volumes:
      - .data:/data
    command: server /data --console-address ":9001"
```

```shell
docker compose -f minio.yml up -d
```

## 配置

下载`mc`
```shell
curl https://dl.min.io/client/mc/release/linux-amd64/mc \
  --create-dirs \
  -o ~/minio-binaries/mc

chmod +x ~/minio-binaries/mc

sudo mv ~/minio-binaries/mc /usr/local/bin/mc
```


```shell
# 配置minio实例别名
mc alias set minio http://<host>:9000 <AK> <SK>

# 列出buckets
mc ls minio

# 删除buckets
mc rb minio/cloudreve
```