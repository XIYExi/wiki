# MySQL5.7 Docker安装

> 参考 https://www.jianshu.com/p/b26625736cc2


## 拉取镜像

```bash | pure
docker pull mysql:5.7
```

## 创建映像文件

```bash | pure
mkdir -p /docker/mysql/log   # 日志文件
mkdir -p /docker/mysql/data   # 数据文件
mkdir -p /docker/mysql/conf   # 配置文件
```

## 配置 conf 文件

```bash | pure
vim /docker/mysql/conf/my.cnf  # 新建配置文件
```

```bash | pure
[client]
default_character_set=utf8
[mysqld]
collation_server = utf8_general_ci
character_set_server = utf8
```

## 挂载 docker

```bash | pure
docker run -d \
--name mysql \
-p 3306:3306 \
--privileged=true \
-v /docker/mysql/log:/var/log/mysql \
-v /docker/mysql/data:/var/lib/mysql \
-v /docker/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:5.7
```


