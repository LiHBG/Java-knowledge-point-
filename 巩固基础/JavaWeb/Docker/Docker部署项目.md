# Docker部署项目

利用docker安装应用是，docker会自动搜索并下载应用镜像(image)。镜像不仅包含应用本身，还包含应用运行所需要的环境、配置、系统函数库。docker会在运行镜像时创建一个隔离环境，称为容器。

## 安装Docker

1. 首先卸载存在的旧版本

   ```bash
   yum remove docker  \
       docker-client  \
       docker-client-latest  \
       docker-common  \
       docker-latest  \
       docker-latest-logrotate  \
       docker-logrotate  \
       docker-engine  \
       docker-selinux 
   ```

2. 安装yum工具

   ```bash
   yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

3. 配置docker的yum源

   ```bash
   yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
   
   sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
   ```

4. 更新yum，建立缓存

   ```bash
   yum makecache fast
   ```

5. 安装Docker

   ```shell
   yum install -y \
   	docker-ce \
   	docker-ce-cli \
   	containerd.io \
   	docker-buildx-plugin \
   	docker-compose-plugin
   ```

6. 启动docker    systemctl start docker

7. 关闭docker    systemctl stop docker

8. 重启docker    systemctl restart docker

9. 设置开机自启  systemctl enable docker

10. 配置镜像加速

    1. 创建文件 /etc/docker/daemon.json

    2. 配置镜像,需要查询最新的国内可用镜像

       ```bash
       tee /etc/docker/daemon.json <<-'EOF'
        {
         "registry-mirrors": [
           "https://docker.hpcloud.cloud",
           "https://docker.m.daocloud.io",
           "https://docker.unsee.tech",
           "https://docker.1panel.live",
           "http://mirrors.ustc.edu.cn",
           "https://docker.chenby.cn",
           "http://mirror.azure.cn",
           "https://dockerpull.org",
           "https://dockerhub.icu",
           "https://hub.rat.dev",
           "https://proxy.1panel.live",
           "https://docker.1panel.top",
           "https://docker.m.daocloud.io",
           "https://docker.1ms.run",
           "https://docker.ketches.cn"
         ]
       }
       EOF
       ```

11. 重新加载配置   systemctl daemon-reload

12. 重启docker  systemctl restart docker

## 安装MySQL

```bash
docker run -d \
  --name mysql \				#取名为mysql
  -p 3307:3306 \				#所需要的端口映射
  -e TZ=Asia/Shanghai \			#
  -e MYSQL_ROOT_PASSWORD=123 \  #指定登录的密码是123
  mysql:8
```

命令解读

- docker run : 创建并运行一个容器， -d 是让容器在后台运行
- --name mysql : 给容器取一个名字，在docker中容器的名字必须唯一
- -p 3307:3306 : 设置端口映射（前面写的是宿主机的端口，后面写的是docker容器中的端口）
- -e KEY=VALUE ： 设置环境变量
- -v 数据卷:容器内目录 可以完成数据卷挂载（数据卷不存咋会自动创建）
- mysql:8 ：指定运行的镜像的名字，版本（前者是镜像名，后者是镜像的版本），没有指定版本的时候是默认的最新版本

## Docker常用命令

镜像相关：

- docker pull                          从镜像仓库中拉取镜像到本地

- docker images                    查看本地仓库中的所有镜像

- docker rmi + 镜像名称       从本地镜像仓库中删除指定的镜像

- docker bulid                        自己制作一个docker镜像

- docker save                         打包一个docker镜像

  ```bash
  docker save -o nginx-1.20.2.tar nginx:1.20.2
  ```

- docker load                         将打包好的docker镜像加载到本地镜像仓库中

  ```bash
  docker load -i nginx-1.20.2.tar
  ```

  

- docker push                        将自己的镜像推送到镜像仓库中（需要权限）

容器相关：

- docker run                          将镜像布置到容器中并运行

- docker stop                         将运行的容器停止

- docker start                         将停止的容器重新运行

- docker ps                             查看当前正在运行的容器

  ```bash
  #查看全部的容器 
  docker ps -a
  ```

- docker rm                            删除容器

  ```bash
  #删除未运行的容器
  docker rm 容器名/容器id
  #强制删除正在运行的容器
  docker rm -f 容器名/容器id
  ```

  

- docker logs                          查看容器运行的日志

- docker exec                         进入容器的内部

  ```bash
  #进入容器并进行操作分配一个命令台指令为bash
  docker exec -it 容器名 bash
  ```

  

## 安装NGINX

1. 搜索NGINX镜像，查看镜像的名称

2. 拉取NGINX镜像

   ```bash
   docker pull nginx:版本号
   ```

3. 查看本地镜像列表

   ```bash
   docker images
   ```

4. 创建并运行NGINX容器

   ```bash
   docker run -d \
   	--name nginx \
   	-p 80:80 \
   	nginx:1.20.2
   ```

5. 查看容器

   ```bash
   docker ps
   ```

## 数据卷

是一个虚拟目录，是 **容器内目录**与 **宿主机目录**之间映射的桥梁。

创建数据卷的时候，会在默认的目录(/var/lib/docker/volumes/)下创建，会存储在数据卷下的/data内。

**操作指令**

- 创建数据卷 docker volume create 

  ```bash
  docker volume create html
  ```

- 查看所有数据卷 docker volume ls

- 删除指定数据卷 docker volume rm

- 查看某个数据卷的详情 docker volume inspect 

- 清楚所有未使用的数据卷 docker volume prune

```bash
#使用数据卷挂载
docker run -d \
	--name nginx \
	-p 80:80 \
	-v html:/usr/share/nginx/html \
nginx:1.20.2
#使用本地宿主机目录挂载
docker run -d \
	--name niginx \
	-p 80:80 \
	-v /root/nginx/html:/usr/share/nginx/html \
	-v /root/nginx/conf:/user/share/negin/conf \
nginx:1.20.2
#使用本地宿主机目录进行挂载
docker run -d \
	--name mysql \
	-p 3306:3306 \
	-e TZ=Asia/Shanghai \
	-e MYSQL_ROOT_PASSWORD= root \
	-v /root/mysql/data:/var/lib/mysql \
	-v /root/mysql/conf:/etc/mysql/conf.d \
	-v /root/mysql/init:/docker-entrypoint-initdb.d \
mysql:8
```

## 自定义镜像

构建Java镜像步骤：每次操作都会形成一个层，最底层被称为基础镜像（系统函数库、环境配置、文件等），最上层被称为入口（一般是程序启动的脚本或者参数）

1. 准备一个Linux运行环境
2. 安装JDK并配置环境变量
3. 拷贝jar包
4. 编写运行脚本，运行jar包

### Dockerfile

dockerfile就是一个文件，其中包含一个个指令，用指令来说明要执行什么操作来构建镜像。将来docker可以根据dockerfile帮助我们构建。

**常见指令**

- 指定基础镜像 **FROM**

  - ```dcokerfile
    FROM centos:7
    ```

- 设置环境变量,可在后面指令中使用 **ENV**

  - ```dockerfile
    ENV key=value
    ```

- 拷贝指定文件到镜像的指定目录 **COPY**

  - ```dockerfile
    COPY ./jdk-17.tar.gz /tmp
    ```

- 执行Linux的shell命令，一般是安装过程的命令 **RUN**

  - ```dockerfile
    RUN tar -zxf /tmp/jdk-17.tar.gz 
    ```

- 指定容器运行时监听的端口，是给镜像使用者看的 **EXPOSE**

  - ```dockerfile
    EXPOSE 8080
    ```

- 镜像中应用启动的命名，容器运行时调用 **ENTRYPOINT**

  - ```dockerfile
    ENTRYPOINT java -jar xxx.jar
    ```

**制作一个Java程序的docker镜像**

```dockerfile
#指定基础镜像
FORM centos:7
#添加jdk到镜像中
COPY ./jdk-17.0.10_linux-x64_bin.tar.gz /usr/local/
#解压并删除
RUN tar -zxf /usr/local/jdk-17.0.10_linux-x64_bin.tar.gz -C /usr/local/ && rm -f /usr/lcaol/jdk-17.0.10_linux-x64_bin.tar.gz 
#设置环境变量
ENV JAVA_HOME=/usr/local/jdk-17.0.10
ENV PATH=$JAVA_HOME/bin:$PATH
#创建应用目录
RUN mkdir -p /app
#工作目录切换
WORKDIR /app
#复制jar包到容器中
COPY ./XXXXX.jar ./XXXXX.jar
#暴露端口
EXPOSE 8080
#运行命令
ENTRYPOINT ["java","-jar","/app/xxxx.jar"]
```

编写好之后

```bash
docker bulid -t myImage:1.0 .
```

- **-t** 给镜像起名，格式依旧是repository:tag 不指定tag，默认是latest
- **.** 指定dockerfile所在目录，如果就在当前目录则指定为 **“ . ”**

## 部署项目

- **自定义docker镜像**

  - ```bash
    FROM centos:7	
    COPY ./jdk-17.0.10_linux-x64_bin.tar.gz /usr/local/
    RUN tar -zxf /usr/local/jdk-17.0.10_linux-x64_bin.tar.gz -C /usr/local/ && rm /usr/local/jdk-17.0.10_linux-x64_bin.tar.gz
    ENV JAVA_HOME=/usr/local/jdk-17.0.10
    ENV PATH=$JAVA_HOME/bin:$PATH
    RUN mkdir -p /app
    WORKDIR /app
    COPY  TliasTeachingManagementSystem-1.0.0.jar  TliasTeachingManagementSystem-1.0.0.jar
    EXPOSE 8080
    ENTRYPOINT ["java","-jar","TliasTeachingManagementSystem-1.0.0.jar"]
    ```

- **启动容器**

  - ```bash
    docker run -d \
    	--name tlias \
    	-p 8080:8080 \
    tlias:1.0
    ```

- **网络**

  - 默认情况下，所有容器都是以bridge方式连接到Docker的一个虚拟网桥上的。

  - 常见指令：

    - 创建一个网络 docker network create 
    - 查看所有网络 docker network ls
    - 删除指定网络 docker network rm
    - 清除未使用网络 docker network prune
    - 使指定容器连接加入某网络 docker network connect
    - 使指定容器连接离开某网络 docker network disconnect
    - 查看网络详细信息 docker network inspect

    ```bash
    docker run -d \
    	--name nginx \
    	-p 80:80 \
    	-v /root/nginx/html:/usr/share/nginx/html \
    	-v /root/nginx/conf:/user/share/negin/conf \
    	--network tlias \
    nginx:1.20.2
    ```

## DockerCompose

是通过一个单独的docker-componse.yml 模板文件来定义一组想关联的应用容器，帮助我们实现多个互相关联的docker容器的快速部署。

- 准备资源（sql脚本、服务端的jdk、jar包、dockerfile文件、前端项目打包文件、nginx.conf）

- 准备dockercompose.yml配置文件

- 给予dockercompose快速构建项目

  ```yml
  service :
   mysql :
    image : mysql:8
    container_name : mysql
    ports :
     - "3306:3306"
    environment : 
     TZ : Asia/Shanghai
     MYSQL_ROOT_NAME : 123
    volumes :
     - "/usr/local/app/mysql/conf:/etc/mysql/conf.d"
     - "/usr/local/app/mysql/data:/var/lib/mysql"
     - "/usr/local/app/mysql/init:/docker-entrypoint-initdb.d"
    networks :
     - tlias-net
   tlias : 
    build : 
     contaxt: .
     dockerfile: Dockerfile
    container_name: tlias-server
    ports:
     - "80:80"
    networks:
     - tlias-net
    depends_on:
     - mysql #告诉容器依赖的关系，按照依赖的关系进行部署
   nginx:
    image: nginx:1.20.2
    container_name: nginx-tlias
    ports:
     - "80:80"
    volumes:
     - "/usr/local/app/nginx/conf/nginx.conf:/etc/nginx/nginx.conf"
     - "/usr/local/app/nginx/html:/usr/share/nginx/html"
    depends_on:
     - tlias
    network:
     - tlias-net
  networks:
   tlias-net:
    name: tlias
  ```

- 指令 docker compose [options] [command]

  - options
    - -f 指定compose文件的路径名称
    - -p 指定project名称
  - command
    - up 创建并启动所有service容器
    - down 停止并移除所有容器网络
    - ps 列出所有启动的容器
    - logs 查看指定容器的日志
    - stop 停止容器
    - start 启动容器
    - restart 重启容器
    - top 查看运行进度



