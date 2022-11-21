# Docker安装minio

```Bash
docker pull minio/minio:RELEASE.2020-11-25T22-36-25Z
docker run -p 9000:9000 --name minio -d --restart=always -e "MINIO_ACCESS_KEY=root" -e "MINIO_SECRET_KEY=123456789" -v /home/docker/data:/data -v /home/docker/config:/root/.minio minio/minio:RELEASE.2020-11-25T22-36-25Z server /data
```

docker run -p 9000:9000 -p 9001:9001 --net=host --name minio -d --restart=always -e "MINIO_ACCESS_KEY=root" -e "MINIO_SECRET_KEY=123456789" -v /home/docker/minio/data:/data -v /home/docker/minio/config:/root/.minio minio/minio:RELEASE.2020-11-25T22-36-25Z server /data --console-address ":9001" -address ":9000"

```bash

docker run -p 9000:9000 --name minio -d --restart=always -e "MINIO_ACCESS_KEY=root" -e "MINIO_SECRET_KEY=123456789" -v D:/home/docker/data:/data -v D:/home/docker/config:/root/.minio minio/minio:RELEASE.2020-11-25T22-36-25Z server /data
```

