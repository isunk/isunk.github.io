---
title: Docker
tags: docker
categories: docker
---

# Docker

## 安装

- 从 [DaoCloud](http://get.daocloud.io/) 下载并安装（推荐，速度快）
    - 安装 docker
        ```bash
        curl -sSL https://get.daocloud.io/docker | sh
        ```
    - 安装 docker-compose
        ```bash
        curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
        chmod +x /usr/local/bin/docker-compose
        ```
    - 配置 docker 镜像站
        ```bash
        curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
        ```

- Debian/Ubuntu 环境安装 docker 社区版
    1. 新增 docker apt repository
        ```bash
        # curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
        # sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

        curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
        sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
        ```
    2. 安装 docker-ce  
        `sudo apt-get update`  
        `sudo apt-get install docker-ce`
        > 非官方源是不可信任的，可能会出现异常 “W: GPG error: https://download.docker.com trusty Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 7EA0A9C3F273FCD8”，解决办法是导入该源的公钥  
        >   ~~`gpg --keyserver download.docker.com --recv 7EA0A9C3F273FCD8 && gpg --export --armor 7EA0A9C3F273FCD8 | sudo apt-key add -`  
        >   或~~  
        >   `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`  
        >   `sudo apt-get update`

    > 启动 docker 服务  
    > `service docker start`  
    > 停止 docker 服务  
    > `service docker stop`

- [ubuntu 14.04 离线安装 docker 和 docker compose](http://www.cnblogs.com/xiaoff/p/8176819.html)

- [使用 registry 搭建 docker 私有 hub](http://blog.csdn.net/mideagroup/article/details/52052618)
    1. 下载 registry 镜像  
        `docker pull registry`
    2. 创建一个 registry 的容器并运行（"-p 5000:5000" 表示将容器内部的 5000 端口映射到宿主机的 5000 端口）
        ```bash
        docker run -d -p 5000:5000 registry
        ```
        ```bash
        docker run \
            -d \ # 作为 daemon 进程启动，即后台启动
            -v /myrepo:/var/lib/registry \ # 将容器 /var/lib/registry 目录映射到宿主机的 /myrepo，用于存放镜像数据
            -p 5000:5000 \ # 将容器的 5000 端口映射到 Host 的 5000 端口（5000 是 registry 服务端口）
            registry:latest
        ```
    3. 验证 registry 容器是否启动成功  
        `curl http://127.0.0.1:5000/v2/_catalog`
    4. 使用 registry 管理仓库和镜像
        - 上传镜像
            1. 使用 tag 命令让本地镜像 hello-world 指向到 registry 仓库中  
                `docker tag hello-world localhost:5000/hello-world:latest`
            2. 推送到 registry 仓库中  
                `docker push localhost:5000/hello-world`
        - 下载镜像  
            `docker pull localhost:5000/hello-world`
    5. 开启 TLS 认证
        1. 启动服务
            ```bash
            # 生成证书
            openssl req -newkey rsa:4096 -nodes -sha256 -keyout myhub.key -subj "/CN=myhub.com" -x509 -days 365 -out myhub.crt

            # 生成账号密码 zhangsan/123456
            docker run --rm --entrypoint htpasswd httpd:2 -Bbn zhangsan 123456 > htpasswd

            # 启动 registry 容器
            docker run -d -p 5000:5000 --restart=always --name registry \
                # -v /myrepo:/var/lib/registry \
                -v $(pwd):/certs \ # 将证书和私钥文件挂载进容器内 /certs 目录下
                -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/myhub.crt \ # 指定（容器内的）证书和私钥文件路径
                -e REGISTRY_HTTP_TLS_KEY=/certs/myhub.key \
                -v $(pwd):/auth \ # 将账号密码文件挂载进容器内 /auth 目录下
                -e "REGISTRY_AUTH=htpasswd" \
                -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
                -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \ # 指定（容器内的）账号密码文件路径
                registry
            ```
        2. 配置访问
            ```bash
            # 配置 host
            echo "127.0.0.1 myhub.com" >> /etc/hosts

            # 验证服务是否启动
            curl --cacert myhub.crt https://myhub.com:5000/v2/_catalog

            # 下发证书
            mkdir -p /etc/docker/certs.d/myhub.com\:5000
            cp myhub.crt /etc/docker/certs.d/myhub.com\:5000

            # 账号密码登录
            docker login myhub.com:5000
            # username: zhangsan
            # password: 123456

            # 上传镜像
            docker tag hello-world myhub.com:5000/hello-world:latest
            docker push myhub.com:5000/hello-world

            # 下载镜像
            docker pull myhub.com:5000/hello-world

            # 退出登录
            docker logout myhub.com:5000
            ```

- 使用 Harbor 搭建 docker 私有 hub
    1. 搭建
    2. 配置域名访问，如 https://www.mydockerhub.com
    3. 上传镜像
        1. 在 /etc/docker/certs.d 目录下配置 Harbor 服务的公钥证书
            ```bash
            cd /etc/docker/certs.d
            
            mkdir www.mydockerhub.com
            
            cd www.mydockerhub.com
            
            openssl s_client -showcerts -connect www.mydockerhub.com:443 < /dev/null 2> /dev/null | openssl x509 -outform PEM > www.mydockerhub.com.crt
            ```
        2. 登录 Harbor 服务  
            `docker login www.mydockerhub.com`
        3. 推送镜像  
            `docker push www.mydockerhub.com/hello-world:latest`

- [配置阿里云镜像加速器](https://cr.console.aliyun.com/?spm=a2c4g.11186623.2.3.LLTrnr#/accelerator)
    1. 修改 daemon 配置文件 /etc/docker/daemon.json 来使用加速器（其中 "https://0cx54rhx.mirror.aliyuncs.com" 为已申请的专属加速器地址）
        ```bash
        tee /etc/docker/daemon.json <<-'EOF'
        {
            "registry-mirrors": ["https://0cx54rhx.mirror.aliyuncs.com"]
        }
        EOF
        ```
    2. <span id="restart-docker-server">重启 docker 服务</span>  
        ```bash
        # ubuntu
        service docker restart

        # centos
        # systemctl restart docker
        
        # 如果是在容器中（如 docker:dind）重启 docker 服务，则直接在宿主机上执行 docker restart 重启容器即可
        ```

- 开启 docker 实验功能
    1. 修改配置文件 /etc/docker/daemon.json，在 `/` 节点下新增配置项
        ```json
        "experimental": true
        ```
    2. [重启 docker 服务](#restart-docker-server)

## 使用

- 查看 docker 版本信息  
    `docker version`  
    查看 docker 系统的信息  
    `docker info`

- 镜像（Image）管理
    ```bash
    # 列出远程仓库 Docker Hub 中所有名称为 ubuntu 的镜像
    docker search ubuntu
    # 从远程仓库 Docker Hub 中下载名称为 ubuntu 的镜像到本地
    docker pull ubuntu
    # 推送/发布镜像至远程仓库 Docker Hub 中
    docker push new_image_name

    # 列出本地仓库中所有的 docker 镜像
    docker images
    # 删除本地仓库中名称为 ubuntu 的镜像
    docker rmi ubuntu
    # # 删除本地仓库中所有镜像
    # docker rmi $(docker images -q)

    # 生成一个镜像 ubuntu 的实例（即容器），并执行该容器中的 /bin/bash，从而交互式进入到该容器中（键入 Ctrl + P + Q 键退出容器但不停止，键入 Ctrl + D 键或执行命令 exit 即可退出容器且停止容器）
    docker run -it ubuntu /bin/bash
    # # 创建一个镜像 ubuntu 的容器，新建的容器处于停止状态，可以使用 docker start 命令启动
    # docker create -it ubuntu
    # # 生成一个新的容器，并运行命令 ping www.google.com
    # docker run ubuntu ping www.google.com
    # # 通过 “-v” 参数可以支持把一个宿主机上的目录挂载到容器中
    # docker run -it -v /home/downloads:/usr/downloads ubuntu /bin/bash
    # 使用 "--entrypoint bash" 覆盖容器中 entrypoint 的默认启动命令，可用于启动因原 entrypoint 命令存在问题而导致无法启动容器的镜像
    # docker run --entrypoint bash -it nginx
    # 创建一个 redis 的只读容器
    # docker run -- read-only redis

    # 根据当前目录下的文件 Dockerfile 构建出一个名称为 IMAGE[:VERSION] 的镜像
    docker build -t IMAGE[:VERSION] .
    docker build --squash -t IMAGE[:VERSION] . # 参数 "--squash" 表示压缩镜像，需要开启 docker 实验功能

    # # 查看指定镜像的创建历史
    # docker history IMAGE[:VERSION]

    # 打包镜像 ubuntu，并导出到文件 ubuntu.tar 中
    docker save -o ubuntu.tar ubuntu
    # 从文件 ubuntu.ta 中导入镜像到本地仓库中
    docker load -i ubuntu.tar
    # docker load < ubuntu.tar
    ```

- 容器（Container）管理
    ```bash
    # 列出本地所有正在运行的容器
    docker ps
    # 列出本地所有的容器
    docker ps -a
    # # 列出本地所有的容器和体积大小
    # docker ps -as

    # 登录一个正在执行的容器
    docker attach [CONTAINER ID|NAMES]

    # 启动容器 # 其中 CONTAINER ID 为容器 ID、NAMES 为容器名称，可由 docker ps 查得。如启动 ID 为 5bf9b7645ed2 的容器：docker start 5bf9b7645ed2
    docker start [CONTAINER ID|NAMES]
    # 停止容器 # 如停止 NAMES 为 focused_bardeen 的容器：docker stop focused_bardeen
    docker stop [CONTAINER ID|NAMES]
    # 杀死容器
    docker kill [CONTAINER ID|NAMES]
    # docker kill $(docker ps -a -q) # 杀死所有正在运行的容器
    # 重启容器
    docker restart [CONTAINER ID|NAMES]
    # 删除容器
    docker rm [CONTAINER ID|NAMES]
    # docker rm $(docker ps -a -q) # 删除所有已经停止的容器

    # 从一个容器中取日志
    docker logs [CONTAINER ID|NAMES]

    # 列出一个容器里面被改变的文件或者目录，list 列表会显示出三种事件，A 增加的，D 删除的，C 被改变的
    docker diff [CONTAINER ID|NAMES]

    # 显示一个运行的容器里面的进程信息
    docker top [CONTAINER ID|NAMES]

    # 从容器里面拷贝文件/目录到本地一个路径
    # docker cp [CONTAINER ID|NAMES]:/container_path to_path
    docker cp /sbin/ifconfig 5bf9b7645ed2:/sbin

    # 使用已启动的容器，运行命令 ping www.google.com
    docker exec 5bf9b7645ed2 ping www.google.com
    # 使用已启动的容器，交互运行 /bin/sh
    docker exec -it b7542f84a6c3 /bin/sh

    # 导出容器到文件 /tmp/container01.tar 中
    docker export [CONTAINER ID|NAMES] > /tmp/container01.tar
    # 从文件 /tmp/container01.tar 中导入容器，并设置其 NAMES 为 focused_bardeen
    cat /tmp/container01.tar | docker import - focused_bardeen
    # # 导出容器并重新导入到镜像中，可用于压缩镜像。ref https://blog.csdn.net/qq_36763896/article/details/53293088
    # docker export <container id> | docker import - <image name>

    # 将容器 a404c6c174a2 保存为新的镜像 hello，并添加提交人信息和说明信息
    docker commit -a "author" -m "describe" a404c6c174a2  hello:v1
    # docker commit --change='ENTRYPOINT ["entrypoint.sh"]' a404c6c174a2  hello:v1
    # docker commit -c 'ENV JAVA_HOME=/opt/jdk' -c 'ENV PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:$JAVA_HOME/bin' -c 'ENTRYPOINT service ssh start && /bin/bash' a404c6c174a2 ubuntu-jdk-ssh

    # 查看容器端口映射
    docker port a404c6c174a2
    docker port a404c6c174a2 8080
    ```

- [网络管理](http://www.cnblogs.com/sammyliu/p/5894191.html)
    ```bash
    # 创建一个 overlay 网络 overlay1
    docker network create -d overlay --attachable overlay1

    # 查看网络 overlay1 的信息
    network inspect overlay1
    ```
    ```bash
    # 查看 NAT 表中的 PREROUTING 链（iptables 将满足条件的数据都转发到了 DOCKER 链上去了）
    iptables -t nat --list-rules PREROUTING

    # 查看 NAT 表中的 DOCKER 链
    iptables -t nat --list-rules DOCKER

    # 查看容器的 ip
    docker inspect <容器id> | grep '"IPAddress":' | grep -o -E [0-9.]+ | head -n 1

    # 端口映射
    iptables -t nat -A DOCKER -p tcp --dport <容器外部端口> -j DNAT --to-destination <容器ip>:<容器内部端口>
    # iptables -t nat -A POSTROUTING -j MASQUERADE -p tcp --source <容器ip> --destination <容器ip> --dport <容器内部端口>
    iptables -A DOCKER -j ACCEPT -p tcp --destination <容器ip> --dport <容器内部端口>

    # 取消端口映射规则
    iptables -t nat -D DOCKER -p tcp -d 0/0 --dport <容器外部端口> -j DNAT --to-destination <容器ip>:<容器内部端口>
    iptables -D DOCKER -j ACCEPT -p tcp --destination <容器ip> --dport <容器内部端口>
    ```

- 删除那些已停止的容器、dangling 镜像、未被容器引用的 network 和构建过程中的 cache  
    `docker system prune`  
    强制清理  
    `docker system prune --all --force --volumes`

- 资源限制
    ```bash
    docker run --rm -it \
        --cpu-period=100000 --cpu-quota=20000 \ # 每 100 毫秒(即 100000 微秒)内，运行进程使用的 cpu 时间最多为 20 毫秒
        --cpuset-cpus="1" \ # 指定运行容器编号为 1 的 cpu。https://m.jb51.net/article/135395.htm?from=singlemessage
        -m 50M --memory-swap 50M \ # 容器可以使用 50M 的物理内存，且不能使用 swap。https://www.cnblogs.com/sparkdev/p/8032330.html
        --device-write-bps /dev/sda:30MB \ # 限制容器写 /dev/sda 的速率为 30MB/s。http://m.hangge.com/news/cache/detail_2413.html
        u-stress:latest /bin/bash
    
    stress -c 4 # 启动压力测试
    ```

- 示例
    ```bash
    # 创建一个（虚拟的）网络接口名称为 docker1、网络 NAME 为 bridge1、子网网段为 10.0.1.* 的网桥
    docker network create -o "com.docker.network.bridge.name"="docker1" --subnet=10.0.1.0/24 bridge1

    # 停用网络接口 docker1
    # ifconfig docker1 down
    #apt-get install bridge-utils
    #brctl delbr docker1

    # 创建一个镜像为 ubuntu、网桥为 bridge1、ip 地址为 10.0.1.2、名称为 client1 的容器，并将宿主机目录 /sbin 挂载到容器中目录 /sbin 下，交互式启动容器中 /bin/bash
    docker run -it --network bridge1 --ip 10.0.1.2 --name client1 -v /sbin:/sbin ubuntu /bin/bash
    ```

- 构建镜像
    - [构建极简镜像（镜像中应用存在依赖关系）](https://www.jianshu.com/p/3a2a96ba6b44)  
        例如，基于一个空的基础镜像完成一个 /bin/hello 的镜像
        1. 查看 /bin/hello 的依赖库，并将 /bin/hello 以及其依赖的动态库，打包至文件 rootfs.tar.gz 中  
            `ldd /bin/hello | perl -p -e 's#^\s*(\S+ => )?(/\S+) \(0x[0-9a-z]+\)$#$2#g' | grep -v 'linux-vdso' | sed '$a/bin/hello' | xargs tar zcvf rootfs.tar.gz`
        2. 编辑文件 Dockerfile 构建镜像
            ```dockerfile
            FROM scratch 

            ADD rootfs.tar.gz / 
            
            # COPY redis.conf /etc/redis/redis.conf 
            # EXPOSE 6379

            # CMD ["redis-server"]
            CMD ["/bin/hello"]
            ```
        3. 构建镜像 hello  
            `docker build -t hello .`
    - 构建简单镜像（镜像中应用无依赖）
        1. 编辑文件 [Dockerfile](/Source/docker/hello.sh)
            ```dockerfile
            FROM scratch
            ADD hello /
            CMD ["/hello"]
            ```
            > Docker registry 中，有一个被称为 scratch 的使用空 tar 文件构建的特殊镜像  
            > `tar cv --files-from /dev/null | docker import - scratch`  
            > 基于该映像可构建新的无冗余的镜像

        2. 构建镜像 hello  
            `docker build -t hello .`
    - [多阶段构建](https://m.toutiaocdn.com/i6807657333934522891/?app=news_article&timestamp=1585054531&req_id=202003242055310101450260140F1D4D28&group_id=6807657333934522891&wxshare_count=1&tt_from=weixin&utm_source=weixin&utm_medium=toutiao_android&utm_campaign=client_share&from=singlemessage)  
        ```dockerfile
        FROM gcc AS mybuildstage
        COPY hello.c .
        RUN gcc -o hello hello.c
        FROM alpine
        COPY --from=mybuildstage hello .
        CMD ["./hello"]
        ```
<!--
    - 使用 [docker-squash](https://github.com/jwilder/docker-squash) 压缩镜像体积
        1. 下载  
            `curl -k -O -L https://github.com/jwilder/docker-squash/releases/download/v0.2.0/docker-squash-linux-amd64-v0.2.0.tar.gz --retry 3 --retry-delay 2`
        2. 解压并添加至 /usr/bin 目录下
            ```bash
            tar -C /usr/bin -xzvf docker-squash-linux-amd64-v0.2.0.tar.gz
            chown root:root /usr/bin/docker-squash
            chmod 755 /usr/bin/docker-squash
            ```
        3. 使用命令压缩镜像  
            ```bash
            # docker save IMAGE_ID | sudo docker-squash -t NEW_TAG | docker load

            docker save IMAGE_ID > image.tar
            sudo docker-squash -i image.tar -o squashed.tar -t NEW_TAG
            cat squashed.tar | docker load
            ```
    - 使用 [docker-slim](https://github.com/docker-slim/docker-slim) 压缩镜像体积（不推荐，实测新镜像在运行时出现异常）
        1. 下载  
            `curl -O https://downloads.dockerslim.com/releases/1.25.3/dist_linux.tar.gz`
        2. 解压并添加至 /usr/bin 目录下  
            ```bash
            tar zxvf dist_linux.tar.gz
            (cd ./dist_linux && {
                chown root:root ./docker-slim ./docker-slim-sensor
                chmod 755 ./docker-slim ./docker-slim-sensor
                mv ./docker-* /usr/bin/
            })
            rm -d dist_linux
            ```
        3. 使用命令压缩镜像  
            `docker-slim build --http-probe IMAGE[:VERSION]`
-->
    > [基础镜像包下载](http://openvz.org/Download/templates/precreated)

- [连接宿主机 docker 服务（daemon）](#docker_client_in_docker)

- 连接远程 docker 服务（daemon）
    1. 配置 docker.service 启用 remote api
        ```bash
        mkdir -p /etc/systemd/system/docker.service.d
        cat > /etc/systemd/system/docker.service.d/override.conf << EOF
        [Service]
        ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2376
        EOF
        ```
        或者配置 daemon.json
        ```bash
        vim /etc/docker/daemon.json
        {
            "hosts": ["tcp://0.0.0.0:2376", "unix:///var/run/docker.sock"]
        }
        ```
    2. 重启服务使配置生效
        ```bash
        systemctl daemon-reload && systemctl restart docker.service
        ```
    3. 查看 dockerd 进程
        ```bash
        ps aux | grep dockerd
        # root       2471  0.8  1.7 723016 67808 ?        Ssl  09:38   0:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2376
        ```
    4. 远程主机 docker 客户端执行命令验证
        ```bash
        docker -H tcp://192.168.93.9:2375 version
        ```
    5. curl 命令验证
        ```bash
        curl -XGET http://192.168.93.9:2376/version
        ```
    6. 变量方式使用示例
        ```bash
        docker run -it \
            -e DOCKER_HOST=tcp://192.168.93.9:2376 \
            docker version
        ```

## 故障解决

- 拉取镜像过程中，报 x509:certificate 错误
    - 方法一：设置 insecure-registries
        ```bash
        vim /etc/docker/daemon.json
        {
            "insecure-registries": ["myhub.com:443"]
        }

        # 重启 docker 服务
        ```
    - 方法二：添加信任证书
        ```bash
        # 把网站的 https 证书内容加载到系统 ca-bundle 中
        echo -n | openssl s_client -showcerts -connect myhub.com:443 2>/dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >> /etc/pki/tls/certs/ca-bundle.crt

        # 重启 docker 服务
        systemctl daemon-reload && systemctl restart docker
        ```
        > 当我们在用 docker pull 等命令时，docker 调用的是系统的 CA 证书，这些证书保存在 /etc/pki/certs/ca-bundle.crt 中（centos），所以我们把网站的 https 证书加入到这个文件，重启服务即可

## 其它

<details>
<summary>Docker Swarm</summary>

</details>

<details>
<summary>Docker Compose</summary>

[Docker 之 Compose 服务编排](https://www.cnblogs.com/52fhy/p/5991344.html)

1. 安装
    - 方法一：从 github 上下载 docker-compose 脚本
        ```bash
        # 下载
        sudo curl -L https://github.com/docker/compose/releases/download/1.20.1/docker-compose-Linux-x86_64 -o /usr/local/bin/docker-compose

        # 设置可执行权限
        chmod +x /usr/local/bin/docker-compose
        ```
    - 方法二：使用 pip 安装  
        `pip install docker-compose`  
2. 验证安装（查看 docker-compose 版本）  
    `docker-compose version`
3. 使用
    1. 创建并编辑配置文件 docker-compose.yml 如下
        ```yml
        version: '2'
        services:
          web:
            build: .
            ports:
              - "5000:5000"
          redis:
            image: "redis:alpine"
        ```
    2. 启动服务  
        `docker-compose up`  
        在后台启动服务  
        `docker-compose up -d`  
        查看启动的服务  
        `docker-compose ps`  
        停止服务  
        `docker-compose stop`  
        其他常用命令
        ```bash
        # 查看帮助
        docker-compose -h

        # -f 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。
        docker-compose -f docker-compose.yml up -d 

        # 启动所有容器，-d 将会在后台启动并运行所有的容器
        docker-compose up -d

        # 停用移除所有容器以及网络相关
        docker-compose down

        # 查看服务容器的输出
        docker-compose logs

        # 列出项目中目前的所有容器
        docker-compose ps

        # 构建（重新构建）项目中的服务容器。服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db。可以随时在项目目录下运行 docker-compose build 来重新构建服务
        docker-compose build

        # 拉取服务依赖的镜像
        docker-compose pull

        # 重启项目中的服务
        docker-compose restart

        # 删除所有（停止状态的）服务容器。推荐先执行 docker-compose stop 命令来停止容器。
        docker-compose rm 

        # 在指定服务上执行一个命令。
        docker-compose run ubuntu ping docker.com

        # 设置指定服务运行的容器个数。通过 service=num 的参数来设置数量
        docker-compose scale web=3 db=2

        # 启动已经存在的服务容器。
        docker-compose start

        # 停止已经处于运行状态的容器，但不删除它。通过 docker-compose start 可以再次启动这些容器。
        docker-compose stop
        ```

</details>

## 第三方镜像

- alpine（轻量级 linux 发行版，体积 < 5 MB，内置 apk 软件包管理工具，可通过 apk –help 命令查看完整的包管理命令）
    ```bash
    # 创建一个新的 alpine 容器并运行 sh 命令
    docker run -it alpine /bin/sh
    ```

- mysql
    ```bash
    docker run --name mysql01 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
    docker exec -it mysql01 /bin/bash
    # mysql -uroot -p123456
    ```

- java
    ```bash
    # jre 8
    docker run -it openjdk:8u292-jre-slim bash

    # jre 11
    docker run -it openjdk:11.0.12-jre-slim bash
    ```

- [eclipse che（在线 IDE 服务）](https://github.com/eclipse/che)
    ```bash
    # 创建并启动 che 服务
    docker run -it --rm -v /var/run/docker.sock:/var/run/docker.sock -v /home/data:/data eclipse/che start
    ```

- [Ubuntu Classic Desktop (GTK3-based Desktop Environment)](https://github.com/ghostplant/web-ubuntu-mate)  
    > Default VNC password: 123456, and you can update it via 'vncpasswd' command inside VNC X session.
    ```bash
    # Download/Update latest Ubuntu image
    docker pull ghostplant/flashback

    # Chioce 1 - Web as Client: Using web browser to login - http://localhost:8443/
    docker run -it --rm -p 8443:8443 -v /external:/root ghostplant/flashback

    # Chioce 2 - VNCViewer as Client: Using VNC client to login 'localhost:1'
    # docker run -it --rm -p 5901:5901 -v /external:/root ghostplant/flashback
    ```

- [code-server（在线 VS Code）](https://github.com/cdr/code-server)
    ```bash
    docker run --rm -it -p 8080:8080 -v "$PWD:/home/coder/project" codercom/code-server --auth none
    ```

- [minio](https://docs.min.io/cn/minio-docker-quickstart-guide.html)
    ```bash
    docker run -p 9000:9000 --name minio1 \
        -e "MINIO_ACCESS_KEY=AKIAIOSFODNN7EXAMPLE" \
        -e "MINIO_SECRET_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY" \
        -v /mnt/data:/data \
        -v /mnt/config:/root/.minio \
        minio/minio server /data
    ```
    
- [theia（在线 IDE）](https://github.com/theia-ide/theia-apps#theia-docker)
    ```bash
    docker run -it --init -p 3000:3000 -v "$(pwd):/home/project:cached" theiaide/theia:next
    ```
    
- clickhouse
    ```bash
    docker run -d \
        --name clickhouse-server \
        -p 9000:9000 \
        -p 8123:8123 \
        -p 9009:9009 \
        --ulimit nofile=262144:262144 \
        yandex/clickhouse-server
    ```

- [cloudreve](https://github.com/cloudreve/Cloudreve)
    ```bash
    docker run -d \
        --name cloudreve \
        -v /etc/cloudreve:/etc/cloudreve \
        -v /data/cloudreve:/data \
        -p 8081:8080 \
        littleplus/cloudreve-3.0.0-rc-1:sqlite
    ```
    > - 访问路径 http://127.0.0.1:8081，默认管理员账号 admin@cloudreve.org，密码见控制台启动日志
    > - 配置使用 Aria2 离线下载文件
    >     1. [安装、启动 Aria2 RPC 服务](linux.md#aria2)
    >     2. 管理员登录 cloudreve，进入 "管理面板" -> "参数设置" -> "离线下载"。配置 "RPC 服务器地址" 为 "http://127.0.0.1:6800/"，"RPC Secret" 为 "yourpassword"，"临时下载目录" 为 "/home/aria2/data"，点击 "保存" 按钮

- 轻量级搜索引擎 [MeiliSearch](https://github.com/meilisearch/MeiliSearch)  
    内置 webui，开箱即用，包含词干提取、停用词、同义词、排名、过滤器和分页等功能
    ```bash
    docker run \
        -p 7700:7700 \
        -v "$(pwd)/data.ms:/data.ms" \
        getmeili/meilisearch
    ```

- 基于 [distroless](https://github.com/GoogleContainerTools/distroless) 镜像（多阶段）构建精简镜像
    ```dockerfile
    # Start by building the application.
    FROM golang:1.13-buster as build
    WORKDIR /go/src/app
    ADD . /go/src/app
    RUN go get -d -v ./...
    RUN go build -o /go/bin/app

    # Now copy it into our base image.
    FROM gcr.io/distroless/base-debian10
    COPY --from=build /go/bin/app /
    CMD ["/app"]
    ```

- 使用 [nocodb](https://github.com/nocodb/nocodb) 将关系数据库转变为智能电子表格
    ```bash
    docker run -d --name nocodb -p 8080:8080 nocodb/nocodb
    ```

- 使用 [inbucket](https://github.com/inbucket/inbucket)（轻量级邮件接受平台，基于 golang 开发，占用内存 15 MB，仅支持 POP3 和 SMTP 服务）搭建邮件测试服务
    ```bash
    docker run \
        -d \
        --name inbucket \
        -p 9000:9000 \ # 控制台监听接口
        -p 2500:2500 \ # SMTP 服务监听接口
        -p 1100:1100 \ # POP3 服务监听接口
        inbucket/inbucket

    start http://127.0.0.1:9000
    ```

- 基于 Go 实现的搜索引擎 [zinc](https://github.com/prabhatsharma/zinc)
    ```bash
    docker run \
        -v /full/path/of/data:/data \
        -e DATA_PATH="/data" \
        -e FIRST_ADMIN_USER=admin \
        -e FIRST_ADMIN_PASSWORD=Admin_1234 \
        -p 4080:4080 \
        --name zinc \
        public.ecr.aws/m5j1b6u0/zinc:v0.1.1

    start http://127.0.0.1:4080
    ```

- [docker in docker](https://blog.csdn.net/networken/article/details/108218569)
    - <span id="docker_client_in_docker">客户端</span>
        1. 方法一：挂载主机上的 docker.sock 以及 docker 二进制文件
            ```bash
            docker run -it \
                -v /var/run/docker.sock:/var/run/docker.sock \ # 将宿主机的 docker 服务（daemon）挂载进容器内，使的容器内部 docker 客户端可以访问宿主机的 docker 服务
                -v $(which docker):/usr/bin/docker \
                -v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 \
                ubuntu:xenial docker version
            ```
        2. 方法二：使用官方 docker 客户端
            ```bash
            docker run -it \
                -v /var/run/docker.sock:/var/run/docker.sock docker:latest \ # docker:latest 只包含 docker 客户端
                docker version
            ```
        > 本质上还是连接到主机节点上的 docker 引擎执行 docker 命令，实际容器中执行 docker 操作与在主机节点直接执行 docker 命令没有区别。
    - 服务端 + 客户端
        ```bash
        docker run -it \
            --privileged \
            -d \
            --name docker-dind \
            docker:dind # docker:dind 包含 docker 客户端和 docker 服务端

        # 使用带有 docker 客户端的容器连接到 dind 容器
        docker run -it --link docker-dind:docker docker:latest version
        ```
        ```bash
        # 启用 TLS
        docker network create dind

        docker run \
            --privileged \
            --name docker-dind \
            -d \
            --network dind --network-alias docker \
            -e DOCKER_TLS_CERTDIR=/certs \
            -v docker-certs-ca:/certs/ca \
            -v docker-certs-client:/certs/client \
            docker:dind
        ```
        > docker dind 方式有所不同，所有的 docker 操作真正在容器内部进行，包括创建的容器、拉取的镜像都保留在容器内部。  
        > docker in docker 的原理是挂载 cgroup、tmpfs、securityfs、cgroup 的 SUBSYS、关掉不需要的文件描述符、最后启动 dockerd。

- [ZeroNet](https://github.com/HelloZeroNet/ZeroNet)
    ```bash
    docker run -d -e "ENABLE_TOR=true" -v <local_data_folder>:/root/data -p 15441:15441 -p 127.0.0.1:43110:43110 nofish/zeronet
    ```
