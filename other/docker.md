# Docker使用教程



## 安装

1. 下载 https://docs.docker.com/docker-for-mac/install/#download-docker-for-mac
2. mac可视化安装即可。



## 常用命令

**image:**

```bash
# 显示镜像列表
docker image ls
# 拉取远程镜像
docker image pull [url]
# 删除镜像
docker image rm [image]
# 创建镜像
docker image build -t [imageName:tag] [DockerfileDir]
```

**container:**

```bash
# 运行容器-创建新容器
docker container run [imageName]
# 结束-直接杀死进程
docker container kill [container]
# 启动已有容器
docker container start [container]
# 停止容器-安全退出
docker container stop [container]
# 显示正在运行的容器列表
docker container ls
# 显示全部容器列表
docker container ls --all
# 删除容器
docker container rm [container]
# 进入容器
docker attach [container]
```

**compose:**

```bash
# 启动
docker-compose up
# 停止
docker-compose stop
# 删除
docker-compose rm
```

