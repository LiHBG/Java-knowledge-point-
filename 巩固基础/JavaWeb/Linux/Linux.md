# Linux

一套免费使用和自由传播的操作系统。

## 常见命令

```bash
#关机
init 0
#重启
init 6
```

命令的格式：command [-options] [parameter]

说明：

- command :      命令名
- [-options]：   选项，可采用对命令进行控制，也可以省略(可选)
- [parameter]：参数，可以是零个、一个或多个(可选)

### 目录&文件操作命令

- ls 显示指定目录下的内容 语法：ls [-al] [dir]

  - -a:显示所有文件及目录(. 开头的隐藏文件也会列出)
  - -l:除文件名外，同时将文件类型(d表示目录，-表示文件)、权限、拥有者、文件大小等信息详细列出

  ```bash
  #如果不指定文件夹 那么就会显示当前目录下的内容
  ls /
  ls -a /
  ls -l /
  ls -al /
  ll -a == ls -al
  #Linux简化 ls -l 为 ll
  ```

- cd 用于切换当前工作目录，即进入指定目录 语法：cd [dirName]

  - .  表示目前所在的目录
  - .. 表示当前目录位置的上级目录
  - ~ 表示用户的home目录

  ```bash
  cd .. 返回上级目录
  cd ~ 进入表示用户的home目录
  cd - 切换到上次所在目录
  ```

- mkdir 创建目录 语法：mkdir [-p] dirName

  - -p 确保目录名称存在，不存在就创建一个。通过此选项可以实现多层目录创建

  ```bash
  #在当前目录下，建立一个mysql的子目录
  mkdir mysql 
  #在mysql目录下创建一个mysql_name的子目录，如果mysql不存在就创建一个
  mkdir -p mysql/mysql_name
  ```

- rm 删除文件或者目录 语法：rm [-rf] name

  - -r  将目录以及目录中所有文件(目录)逐一删除，即递归删除。
  - -f  无需确认，直接删除。

  ```bash
  rm -r mysql/ 删除名为mysql的目录以及其目录中的所有内容，删除前需要确认
  rm -rf mysql/ 同上，但是删除的时候不需要确认
  rm -f mysql_name/ 无需确认，直接删除
  ```

- cat  用于显示文件的所有内容 语法：cat [-n] fileName 查看小文件

  - -n  由1开始对所有输出的行数编号

  ```bash
  cat /etc/profile 查看etc目录下的profile文件内容
  ```

- more 以分页的形式显示文件内容 语法：more filename

  - 回车键       向下滚动一行
  - 空格键       向下滚动一屏
  - b                 返回上一屏
  - q或Ctrl+c退出more

  ```bash
  more /etc/profile 以分页方式显示/etc目录下的profile文件内容
  more -20 /etc/profile 以分页的方式显示，每页显示20行
  ```

- head 查看文件开头的内容 语法：head [-n] fileName

  - -n  输出文件开头的n行内容

  ```bash
  head 1.log 默认显示该文件的开头10行的内容
  head -20 1.log 显示该文件的开头20行的内容
  ```

- tail 查看文件末尾的内容 语法：tail [-nf] fileName

  - -n 输出文件末尾的n行内容
  - -f  动态读取文件末尾内容并显示，通常用于日志文件的内容输出

  ```bash
  tail 1.log     默认显示该文件末尾10行的内容
  tail -20 1.log 显示该文件末尾20行的内容
  tail -f 1.log  动态读取该文件末尾内容并显示（实时刷新）
  ```

### 拷贝移动&打包压缩

- cp 用于复杂文件或目录  语法：cp [-r] source dest

  - -r  如果复制的是目录需要使用此选项，此时将复制该目录下所有的子目录和文件

  ```bash
  cp hello.txt mysql/ 将hello.txt 复制到mysql目录中
  cp hello.txt ./hi.txt 将hello.txx 复制到当前目录，并改名为hi.txt
  cp -r mysql/ ./mysqlS/ 将mysql目录和目录下所有文件复制到mysqlS目录下
  cp -r mysql/* ./mysqlS/ 将mysql目录下的所有文件复制到mysqlS目录下
  ```

- mv 为文件或目录重命名、或将文件或目录移动到其他位置(第二个参数是已存在的目录执行移动) 语法：mv source dest

  ```bash
  mv hello.txt hi.txt 将hello文件改名为hi
  mv hi.txt mysql/ 将文件移动到mysql目录中
  mv hi.txt mysql/hello.txt 将文件移动到mysql目录下并改名为hello
  mv mysql/ mysqlS/ 如果mysqlS目录不存在，将mysql目录改名为mysqlS
  mv mysql/ mysqlS/ 如果mysqlS目录存在，那么将mysql移动到mysqlS目录中
  ```

- tar   对文件进行打包、解包。压缩、解压  语法：tar [-zcxvf] fileName [files]

  - 包文件后缀为.tar表示只是完成了打包，并没有压缩

  - 包文件后缀为.tar.gz表示打包的同时还进行了压缩

  - -z  z代表的是gzip，通过gzip命令处理文件，gzip可以对文件压缩或者解压

  - -c  c代表的是create，即创建新的包文件

  - -x  x代表的是extract，实现从包文件中还原文件

  - -v  v代表的是verbose，显示命令的执行过程

  - -f  f代表file，用于指定包文件的名称

    - 打包：

      ```bash
      tar -cvf hello.tar hello将当前目录下hello文件打包，打包后的文件名字为hello.tar
      tar -zcvf hello.tar.gz hello 将当前目录下hello文件打包并压缩，打包并压缩后的文件名为 hello.tar.gz
      ```

    - 解包：

      ```bash
      tar -xvf hello.tar 将hello.tar打包文件解包，并将解包后的文件放置到当前目录
      tar -zxvf hello.tar.gz 将打包并压缩的文件解压并解包，并且其解压和解包后的文件放置到当前目录
      tar -zxvf hello.tar.gz -C /usr/local 将文件进行解压和解包并将解压和解包的文件放置到/usr/local目录中。
      ```

### 文本编辑命令&查找命令

- vi 是Linux系统提供的一个文本编辑工具，类似于记事本

  - vi filename

- vim 是从vi发展来的

  ```bash
  #安装vim 
  yum install vim
  ```

  - 底行模式

    ```bash
    设置行数 ： :set nu
    取消设置行数: :set nonu
    定位到第几行:  :需要的行数
保存并退出:  :wq
    不保存退出:  :q!
    ```
    
  - 命令模式
  
    ```bash
    定位到第一行:	   gg
    定位到最后一行:  G
    删除一行:       dd
    删除指定的行数:  ndd  --n代表的是要删除多少行
    撤销:          u
    ```
  
- find  在指定目录下查找文件 语法：find dirName -option fileName

  ```bash
  find . -name "*.log"  在当前目录及其子目录下查找.log结尾文件
  find /mysql -name "*.log" 在/mysql目录及其子目录下查找.log结尾文件
  ```

- grep 从指定文件中查找指定的文本内容 语法: grep [-inAB] word fileName

  - -i : 检索的关键字忽略大小写

  - -n : 显示关键字所在的这一行的行号

  - -A : 输出关键字所在行及其之后的几行数据(-A5 关键字所在行及其之后的5行记录)

  - -B : 输出关键字所在行及其之前的几行数据(-B5 关键字所在行及其之前的5行记录)

    ```bash
    grep Hello Hello.java 查找Hello.java文件中出现的Hello字符串的位置
    grep hello *.java     查找当前目录中所有.java结尾文件中包含hello字符串的位置
    ```

### 系统管理

- top 实时监控系统资源

  ```bash
  top
  ```

- ps 查看进程状态

  ```bash
  ps
  ```

- kill 终止进程 kill -9 强制终止进程

  ```bash
  kill 1234 / kill -9 1234
  ```

- df/du 查看磁盘空间

  ```bash
  df -h #磁盘使用情况
  du -sh nginx-1.20.2/ #查看目录大小
  ```

- free 查看内存使用

  ```bash
  free -h
  ```

- uname 显示系统信息

  ```bash
  uname -a
  ```

- shutdown/reboot 关机/重启

  ```bash
  shutdown -h now #立即关机
  ```

### 权限管理

- chmod 修改文件权限

  ```bash
  chmod 755 script.sh
  ```

- chown/chgrp 修改文件所有者/所属组

  ```bash
  chown root /u         将 /u 的属主更改为"root"。
  chown root:staff /u   和上面类似，但同时也将其属组更改为"staff"。
  chown -hR root /u     将 /u 及其子目录下所有文件的属主更为"root"。
  chgrp staff /u        将 /u 的属组更改为"staff"。
  chgrp -hR staff /u    将 /u 及其子目录下所有文件的属组为"staff"。
  ```

### 用户与组

- useradd/userdel 添加用户/删除用户

  ```bash
  useradd -m home #创建用户以及主目录
  userdel -r home #删除用户以及主目录和邮件池
  ```

- passwd 修改用户密码

  ```bash
  passwd 用户
  ```

- su/sudo 切换用户/管理员权限执行

### 网络相关

- ping 测试网络连通性

- curl/wget 下载文件或测试http请求

  ```bash
  curl -0 http://example.com/file.zip
  ```

- ssh 远程登录服务器

  ```bash
  ssh user@192.168.1.100
  ```

- netstat/ss 查看网络连接状态

  ```bash
  ss -tuln
  ```

- ip/ipconfig 配置/查看网络接口

### 软件包管理

- APT(Debian/Ubuntu)

  ```bash
  apt update         # 更新软件源
  apt install nginx  # 安装软件
  apt remove nginx   # 卸载软件
  ```

- YUM/DNF(RedHat/CentOs)

  ```bash
  dnf install nginx
  ```

- systemctl 管理系统服务

  ```bash
  systemctl start nginx    # 启动服务
  systemctl enable nginx   # 开机自启
  ```

### 其他

- history 查看历史命令

  ```bash
  history | grep ssh
  ```

- man 查看命令手册

  ```bash
  man ls
  ```

- alias 创建命令别名

  ```bash
  alias ll="ls -alh"
  ```

## Linux软件安装

**安装方式:**

- 二进制发布包安装
  - 软件已经针对具体平台编译打包发布，只要解压，修改配置即可
- rpm安装
  - 软件已经按照redhat的包管理规范进行打包，使用rpm命令进行安装，不能自行解决库依赖问题
- yum安装
  - 一种在线软件安装方式，本质上还是rpm安装，自动下载安装包并安装，安装过程中自动解决库依赖问题
- 源码编译安装
  - 软件以源码工程的形式发布，需要自己编译打包

### 安装JDK

1. 使用FinalShell自带的上传工具将jdk的二进制发布包上传到Linux

2. 解压安装包

   ```bash
   tar -zxvf jdk-17.0.10_linux-x64_bin.tar.gz -C /usr/local
   ```

3. 配置环境变量，使用vim命令修改/etc/profile文件，在文件末尾加入配置

   ```bash
   vim /etc/profile
   
   export JAVA_HOME=/usr/local/jdk-17.0.10
   export PATH=$JAVA_HOME/bin:$PATH
   ```

4. 重新加载profile文件，使更改的配置立即生效

   ```bash
   source /etc/profile
   ```

5. 检查安装是否成功

   ```bash
   java -version
   ```

### 安装MySQL

1. 准备工作：卸载Linux系统中自带的mysql/mariadb安装包，否则MySQL将安装失败

   ```bash
   rpm -qa | grep mariadb
   rpm -e --nodeps mariadb-libs-5.6.60-1.el7_5.x86_64
   
   rpm -qa | grep mysql
   rpm -e --nodeps mysql-xxxxxxxx
   ```

2. 下载并上传MySQL安装包

3. 解压安装包到当前目录，并将解压后的文件夹移动到/usr/local目录下，改名为mysql

   ```bash
   tar -xvf mysql-8.0.30-linux-glibc2.12-x86_64.tar.xz
   mv mysql-8.0.30-linux-glibc2.12-x86_64 /usr/lcoal/mysql
   ```

4. 配置环境变量

   ```bash
   export MYSQL_HOME=/usr/local/mysql
   export PATH=$MYSQL_HOME:$PATH
   
   source /etc/profile
   ```

5. 注册MySQL为系统服务

   ```bash
   cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
   chkconfig --add mysql
   ```

6. 初始化数据库

   ```bash
   groupadd mysql
   useradd -r -g mysql \
   -s /bin/false mysql \
   mysqlld --initialize --user=mysql \
   --basedir=/usr/local/mysql \
   --datadir=/usr/local/mysql/data
   
   初始化完毕之后，日志会输出mysql的root用户的临时密码，记得复制
   ```

7. 启动服务登录MySQL

   ```bash
   systemctl start mysql
   mysql -uroot -p xxxx
   ```

8. 配置MySQL的root用户密码，授权远程访问

   ```SQL
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
   CREATE USER 'root'@'%' IDENTIFIED BY 'root';
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
   FLUSH PRIVILEGES;
   ```

9. **防火墙操作**

   1. 查看防火墙状态

      ```bash
      systemctl status firewalld /firewall-cmd --state
      ```

   2. 暂时关闭防火墙

      ```bash
      systemctl stop firewalld
      ```

   3. 永久关闭防火墙(禁用开机自启)

      ```bash
      systemctl disable firewalld
      ```

   4. 暂时开启防火墙

      ```bash
      systemctl start firewalld
      ```

   5. 永久开启防火墙(启用开机自启)

      ```bash
      systemctl enable firewalld
      ```

   6. **开放指定端口**

      ```bash
      firewall-cmd --zone=public --add-port=8080/tcp --permanent
      ```

   7. 关闭指定端口

      ```bash
      firewall-cmd --zone=public --remove-port=8080/tcp --permanent
      ```

   8. 立即生效 -重新加载

      ```bash
      firewall-cmd --reload
      ```

   9. 查看开放端口

      ```bash
      firewadll-cmd --zone=public --list-ports
      ```

### 安装Nginx

1. 安装niginx运行时需要的依赖

   ```bash
   yum install -y pcre pcre-devel zlib zlib-devel openssl openssl-devel
   ```

2. 上传NGINX压缩包

   ```bash
   tar -xvf nginx.xxxxxx
   ```

3. 进入解压目录，执行指令

   ```bash
   ./configure --prefix=/usr/local/nginx
   ```

4. 执行编译NGINX的指令

   ```bash
   make
   ```

5. 执行编译安装指令

   ```bash
   make install
   ```

6. 进入/usr/lcoal/nginx,启动nginx服务

   ```bash
   sbin/nginx
   ```

7. 开发访问端口

   ```bash
   firewall-cmd --zone=public --add-port=80/tcp permanent
   ```

8. 重新加载nginx的配置文件

   ```bash
   sbin/nginx -s reload
   ```

## 前端项目部署

1. 将前端项目的打包文件放入nginx的html目录中
2. 编写nginx/conf/nginx.conf文件
3. 重写加载nginx的配置文件  nginx -s reload

## 后端项目部署

1. 打包之前先测试连接Linux上的数据库，测试通过即可通过maven父工程中的package生命周期对项目进行打包
2. 在Linux服务器的/usr/local/目录下创建一个存放.jar包的目录
3. 执行命令 java -jar xxxx.jar

上述操作会占用窗口，且窗口关闭之后服务也就停止了。可以使用nohup指令，后台运行服务。

```bash
nohup java -jar xxxx.jar &> tlias.log &
```

查看进程

```bash
ps -ef | grep xxxx
```

终结进程

```bash
kill 进程ID
```

强制终结进程

```bash
kill -9 进程ID
```

## Linux目录特点

- / 是所有目录的顶点
  - bin 	  存放二进制可执行文件
  - boot     存放系统引导时使用的各种文件
  - dev       存放设备文件
  - **etc       存放系统配置文件**
  - home   存放系统用户的文件
  - lib         存放程序运行所需的共享库和内核模块
  - opt        额外按照的可选应用程序包所放置的位置
  - **root      超级用户目录**
  - sbin       存放二进制可执行文件，只有root用户才能访问
  - tmp       存放临时文件
  - **usr        存放系统应用程序**
  - var         存放运行时需要改变数据的文件，例如日志文件

**/usr 和 usr在 Linux中只有区别的，如果有/ 那么表示的是在根目录下去寻找后面的目录 是绝对路径；如果没有 / 那么表示在当前目录下寻找 是相对路径。**

