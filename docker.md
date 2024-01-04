**docker 相关命令**

1. 构建镜像

​	docker build -t image_name:tag path_to_dockerfile

```
FROM ubuntu:22.04

ENV LANG=C.UTF-8 DEBIAN_FRONTEND=noninteractive
COPY sources.list /etc/apt/sources.list
RUN apt-get update \
    && apt-get install -y dialog apt-utils tzdata procps telnet iputils-ping lsof net-tools tcpdump curl wget unzip htop tree vim \
    && apt-get install -y nginx supervisor openssh-client openssh-service \
    && apt-get clean all \
    && rm -rf /var/lib/apt/lists/* \
    && useradd -M -s /sbin/nologin nginx \
    && rm -rf /var/www/html/* /etc/nginx/sites-enabled/default \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' >/etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata \
    && echo '"\e[A": history-search-backward' >>/etc/inputrc \
    && echo '"\e[B": history-search-forward' >>/etc/inputrc

COPY start.sh /usr/bin
RUN chmod +x /usr/bin/start.sh
EXPOSE 80 443 1101 1102
ENTRYPOINT ["/usr/bin/start.sh"]

# docker build -t test:v1 .
```



​	2.导入导出

​	镜像

```
docker save ubuntu:load>/root/ubuntu.tar
docker load<ubuntu.tar
```

​	容器

```
docker export 98ca36> ubuntu.tar
cat ubuntu.tar | sudo docker import - ubuntu:import
```

export 和 import 导出的是一个容器的快照, 不是镜像本身, 也就是说没有 layer。

dockerfile 里的 workdir, entrypoint 之类的所有东西都会丢失，commit 过的话也会丢失。

快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也更大。

-  docker save 保存的是镜像（image），docker export 保存的是容器（container）；
-  docker load 用来载入镜像包，docker import 用来载入容器包，但两者都会恢复为镜像；
-  docker load 不能对载入的镜像重命名，而 docker import 可以为镜像指定新名称



3. docker ps
   - -a 列出所有容器
   - -q 仅显示容器id
   - -l 显示最后创建的容器
   - -s 显示容器的统计信息，包括每个容器的磁盘使用情况
4. docker run
   - --name 指定容器名称
   - -d 后台运行
   - -t 分配伪终端
   - -i 交互模式
   - -p 端口映射 宿主机端口:容器端口
   - -v 挂载目录  宿主机目录:容器目录
   - -e 指定环境变量
   - -w 指定工作目录
   - --rm 容器退出时自动删除
   - --network 指定网络
   - --ip 指定ip --ip=192.168.0.11
   - --restart unless-stopped 容器退出时自动重启，手动停止除外
5. docker inspect  查看容器、网络、镜像，卷等的详细信息
   - -f '{{json .Config.Cmd}}'  格式化输出容器启动命令
6. docker logs -f  实时跟踪容器的日志输出
7. docker system 查看管理docker系统的信息与资源
   - df  显示 Docker 系统的磁盘使用情况
   - prune 清理不使用的资源
   - events 查看docker中日志
   - info 查看docker信息
8. docker start|restart|stop|rm 
9. docker rmi xxx 删除镜像



**docker-compose**

1. docker-compose up -d 
   - --force-recreate  重建并启动服务
   - -f 指定yml文件
   - service-name 启动文件中指定服务
   - --build 构建镜像并启动服务



**alpine/dfimage**

查看相关镜像的构建命令

```
docker pull alpine/dfimage
alias dfimage="docker run -v /var/run/docker.sock:/var/run/docker.sock --rm alpine/dfimage"
dfimage -sV=1.36 images:tag
```

