# Docker安装minio mc

```Bash
docker pull minio/mc
docker run -it --entrypoint=/bin/sh minio/mc
```

**Minio只提供了最高7天的分享链接，这里我们需要下载minio client来解决这个问题**

```Bash
sh-4.4# mc config host add minio http://192.168.1.107:9000 root 123456789 --api s3v4
mc: Configuration written to `/root/.mc/config.json`. Please update your access credentials.
mc: Successfully created `/root/.mc/share`.
mc: Initialized share uploads `/root/.mc/share/uploads.json` file.
mc: Initialized share downloads `/root/.mc/share/downloads.json` file.
Added `minio` successfully.
sh-4.4# mc policy set public minio/collection
Access permission for `minio/collection` is set to `public`
sh-4.4#
```