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

