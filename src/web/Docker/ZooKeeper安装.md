# Zookeeper Docker 安装

> 参考 https://zhuanlan.zhihu.com/p/306453731

## 拉去镜像

```bash | pure
docker pull zookeeper:3.4.9  # 拉取指定版本zk镜像
```

## 创建挂载目录

```bash | pure
mkdir -p /root/docker/zookeeper/data
```


## 挂载Docker

```bash | pure
docker run -d \
-p 2181:2181 \
-v /root/docker/zookeeper/data:/data/ \
--name zookeeper \
--privileged {image-ID} # 最后跟着 image ID
```
