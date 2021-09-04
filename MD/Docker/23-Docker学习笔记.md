# docker安装部署

```
yum remove docker docker-common docker-selinux docker-engine
yum install -y yum-utils device-mapper-persistent-data lvm2
wget -O /etc/yum.repos.d/docker-ce.repo https://download.docker.com/linux/centos/docker-ce.repo
sed -i 's+download.docker.com+mirrors.tuna.tsinghua.edu.cn/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum makecache fast
yum install docker-ce -y
systemctl start docker
```

使用国内阿里云镜像加速

```
https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
```

```
touch /etc/docker/daemon.json
cat > /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": ["https://9igar9xy.mirror.aliyuncs.com"]
}
EOF
```

# docker 镜像相关命令

### 搜索镜像

```
docker search centos  
```

### 拉取镜像 镜像名称:版本号

```
docker pull centos:7
```

### 查看镜像

```
docker images 
```

### 导入镜像

```
docker load -i centos_7.tar
```

### 导入镜像

```
docker load < centos_7.tar
```

### 更新镜像标签

```
docker tag centos:7 centos:7.1
```

### 只显示镜像的id号

```
docker images -q
```

### 删除镜像

```
docker rmi centos:7
```

### 删除镜像

```
docker rmi 8652b9f0cb4c
```

### 批量删除镜像

```
docker rmi $(docker images -q)	
```

### 导出镜像

```
docker save -o centos_7.tar centos:7
```

### 导出镜像

```
docker save centos:7 > docker_71.tar
```

### 批量导出镜像版本

```
docker images|awk 'NR>1{print "docker save",$1":"$2,">",$1"_"$2".tar"}'|bash
```

### 使用命令搜索镜像版本

```
yum install jq -y
curl -s https://registry.hub.docker.com/v1/repositories/centos/tags|jq|grep name
```

### 配置镜像加速

```
mkdir -p /etc/docker
cat > /etc/docker/daemon.json <<'EOF'
{
  "registry-mirrors": ["https://ig2l319y.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
```

批量生成镜像

```
for i in {1..100};do docker run -d -it centos:7 /bin/bash;done
```



# docker容器相关命令



## 容器的一些理念

```
1)容器运行的前提条件是进程号PID为1的进程不退出
2)一个容器内最好只干一件事
3)容器一旦创建，就不要再更改了
```

## 容器命令

### 查看正在运行的容器

```
docker ps
```

### 查看所有的容器，包含正在运行和已经退出的

```
docker ps -a
```

### 只显示正在运行的容器的ID号

```
docker ps -q
```

### 显示所有状态的容器的ID号

```
docker ps -aq
```

### 启动一个容器打开bash窗口并且放在后台运行

```
docker run --name linux6 -d -it centos:7 /bin/bash
```

### 分配一个bash终端，并且进入一个容器内，接收用户的输入

```
docker exec -it 容器ID /bin/bash
```

### 停止一个容器

```
docker stop 容器名称
```

### 批量停止容器

```sh
docker stop $(docker ps -q)
```

### 删除容器

```
docker rm cd1ecf35acd6
```

### 批量删除容器

```
docker rm $(docker ps -aq)
```

### 将宿主机的文件发送给容器内部

```
docker cp 宿主机文件 容器ID:/容器内路径
```

### 将容器内的文件拉取到宿主机里

```
docker cp 容器ID:/容器内路径 宿主机路径
```

### 参数解释

```
run 创建并运行一个容器
--name 给容器起个名字，但是不能重复
-i  Keep STDIN open even if not attached	提供交互式输入
-t  Allocate a pseudo-TTY					打开一个新的终端
-d  Run container in background and print container ID	在后台运行一个容器并输出ID


```

### 保持后台容器启动

```
docker run -it -d centos:7
```



### 查看docker对象的底层基础信息

```
docker inspect
```



### Docker端口映射

```
docker run -p 8080:80 -d nginx
docker run -P -d nginx

参数解释：
-p 8080:80   小P 指定端口映射  宿主机端口:容器内端口
-P           大P 随机端口映射到容器内暴露的端口
-d 
```

### Docker数据映射

```
docker run -p 8080:80 -d nginx
docker run -P -d nginx

参数解释：
-p 8080:80   小P 指定端口映射  宿主机端口:容器内端口
-P           大P 随机端口映射到容器内暴露的端口
```

#### 映射目录

```
docker run -d \
-p 80:80 \
-v /code/xiaoniao/:/usr/share/nginx/html/ \
nginx:latest
```

#### 映射配置文件和目录

```yaml
docker run -d \
-p 80:80 \
-v /root/conf/default.conf:/etc/nginx/conf.d/default.conf \
-v /code/:/code/ \
nginx:latest


参数解释：
-v 宿主机目录:容器内目录
```

# nginx部署的game小项目

端口规划：

```sh
访问8080 --> xiaoniaofeifei
访问8090 --> shenjingmao
```

宿主机目录规范：

```sh
mkdir /game/{code,conf}
```

容器里目录规划：

```shell
/code/xiaoniao 
/code/sjm
```

配置文件：

```
xiaoniao.conf 
sjm.conf
```



操作步骤：

```sh
1.创建目录
mkdir game/{code,conf}

2.准备代码文件
unzip xiaoniaofeifei.zip -d game/code/xiaoniao
unzip ShengJinMao.zip -d game/code/
cd game/code/
mv ShengJinMao sjm

3.准备配置文件
cat > game/conf/xiaoniao.conf << EOF
server {
    listen       8080;
    server_name  localhost;
    location / {
        root   /code/xiaoniao;
        index  index.html index.htm;
    }
}
EOF

cat > game/conf/sjm.conf << EOF
server {
    listen       8080;
    server_name  localhost;
    location / {
        root   /game/code/xiaoniao;
        index  index.html index.htm;
    }
}
EOF

4.检查目录和配置文件
[root@docker-11 ~]# tree -L 2 game/
game/
├── code
│   ├── sjm
│   └── xiaoniao
└── conf
    ├── sjm.conf
    └── xiaoniao.conf
	
5.启动容器
docker run -p 8080:8080 -p 8090:8090 \
-v /game/conf/:/etc/nginx/conf.d/ \
-v /game/code/:/game/code/ \
-d nginx
```

# docker容器相关参数

## docker参数--restart=always的作用

```

docker container update --restart=always 容器名字
```



# docker镜像手动构建

先启动一个空的nginx容器

```
docker run -d nginx 
```

docker cp 把代码文件复制到conf指定的目录



相当于把系统部署好 然后再commit

```
docker commit ab046b4bee7e me:v1
```

scp到第二台机器

```
docker run -d -p 8080:8080 -p 8090:8090 game:v1
```

项目部署完成！！！



# 容器的私有仓库部署搭建

## 搭建步骤

```
启动一台全新的centos7 容器
容器里面 vi /etc/yum.conf  修改为  keepcache=1，修改/etc/nginx/conf.d/yum.conf 配置文件
把容器的/var/cache/yum/文件复制到宿主机
宿主机 createrepo --update /data/yum/
```



```
ifconfig eth0:1 10.0.0.100 up
```



## 启动全新一台容器

```
#!/bin/bash

ifconfig eth0:1 10.0.0.100 up
tar zxf yum.tar.gz -C /
cp yum.conf /data/yum/
yum install createrepo -y
createrepo --update /data/yum/

docker run \
--name nginx_yum \
-p 10.0.0.100:80:80 \
-v /data/yum/yum.conf:/etc/nginx/conf.d/yum.conf \
-v /data/yum:/data/yum \
-d nginx:latest

docker exec -it nginx_yum rm -rf /etc/nginx/conf.d/default.conf
docker restart nginx_yum
```

## 宿主机操作:更新私有仓库yum源

```
docker exec 容器id -it  /bin/bash
docker cp 容器id:/var/cache/yum/* /data/yum/
```

```
停止容器后打开宿主机的nginx
打开网站访问内网仓库
```

# 基于CentOS7镜像自己构建一个nginx镜像

## 制作步骤

```
docker pull centos:7
docker run -d -it centos:7 /bin/bash
docker exec -it 87801cc3a821 /bin/bash
rm -rf /etc/yum.repos.d/*
cd /etc/yum.repos.d/
curl -O http://mirrors.aliyun.com/repo/Centos-7.repo
curl -O http://mirrors.aliyun.com/repo/epel-7.repo
sed -i '/aliyuncs/d' /etc/yum.repos.d/CentOS-Base.repo

```

```
cat > /etc/yum.repos.d/nginx.repo << 'EOF' 
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF
```

```
yum makecache fast
yum install nginx -y
```

## 封装新镜像

```
docker commit -m "add nginx" 65a651010c2d centos7_nginx:1
```



## 启动新容器

```
docker run -it -p 80:80 -d centos7_nginx:1 nginx -g 'daemon off;'
```

# 基于centos_nginx镜像自己构建云盘镜像



## 安装php命令

```
yum install php-fpm php-mbstring php-gd -y
sed -i '/^user/c user = nginx' /etc/php-fpm.d/www.conf
sed -i '/^group/c group = nginx' /etc/php-fpm.d/www.conf
sed -i '/daemonize/s#no#yes#g' /etc/php-fpm.conf
php-fpm -c /etc/php.ini -y /etc/php-fpm.conf
```

## 编写启动脚本

```
cat > start.sh << 'EOF'
#!/bin/bash
php-fpm -c /etc/php.ini -y /etc/php-fpm.conf
nginx -g 'daemon off;'
EOF
chmod +x start.sh 
```

## 编写nginx配置文件

```
cat > /etc/nginx/conf.d/cloud.conf <<'EOF'
server {
    listen 80;
    server_name localhost;
    root /code;
    index index.php index.html;

    location ~ \.php$ {
        root /code;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
EOF
```

```
rm -rf /etc/nginx/conf.d/default.conf
```

## 准备代码并测试

```
docker cp code 容器id:/code/
chown -R nginx:nginx /code/
```

## 启动测试

```
docker restart 
nginx  #nginx未启动的话重新启动
php-fpm -c /etc/php.ini -y /etc/php-fpm.conf #启动php-fpm
ps -ef
curl -I 127.0.0.1


###显示一下内容
[root@d92ca9ec3e7a code]# curl -I 127.0.0.1
HTTP/1.1 302 Moved Temporarily
Server: nginx/1.20.1
Date: Thu, 22 Jul 2021 11:59:51 GMT
Content-Type: text/html; charset=utf-8
Connection: keep-alive
X-Powered-By: PHP/5.4.16
Set-Cookie: KOD_SESSION_ID_9d6d9=92qct1rca7a2g8nv8elqt6v323; path=/
Set-Cookie: KOD_SESSION_ID_9d6d9=92qct1rca7a2g8nv8elqt6v323; path=/
Set-Cookie: KOD_SESSION_ID_9d6d9=92qct1rca7a2g8nv8elqt6v323; path=/
Set-Cookie: KOD_SESSION_SSO=9880rf65o78tc77u9rlsltgv14; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Set-Cookie: KOD_SESSION_ID_9d6d9=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT; path=/
Set-Cookie: kod_name=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT
Set-Cookie: kodToken=deleted; expires=Thu, 01-Jan-1970 00:00:01 GMT
Set-Cookie: X-CSRF-TOKEN=deleted; expires=Thu, 01-Jan-1970 00:00:01 GM
```

## 封装新镜像

```
docker commit -m "add kod" d92ca9ec3e7a centos7_kod:v1
```

## 启动新容器

```
docker run --name kod_v1 -p 80:80 -d centos7_kod:v1 /bin/bash /root/start.sh
```

# supervisor的使用

## 安装软件

```
yum install nginx php-fpm php-mbstring php-gd supervisor -y
```

## 编写进程配置文件

```
[root@b7757c17bf36 ~]# cat > /etc/supervisord.d/nginx_php.ini << 'EOF'
[program:nginx]
command=nginx -g 'daemon off;'
autostart=true
startsecs = 5
redirect_stderr = true
stdout_logfile_maxbytes = 20MB
stdout_logfile_backups = 20
stdout_logfile = /var/log/supervisor/nginx.log

[program:php-fpm]
command=php-fpm -c /etc/php.ini -y /etc/php-fpm.conf
autostart=true
startsecs = 5
redirect_stderr = true
stdout_logfile_maxbytes = 20MB
stdout_logfile_backups = 20
stdout_logfile = /var/log/supervisor/php-fpm.log
EOF
```



## 修改supervisord配置文件放在前台启动

```
sed -i "s#nodaemon=false#nodaemon=true#g" /etc/supervisord.conf
```



## 启动supervisord程序

```
supervisord -c /etc/supervisord.conf 
```

## 使用命令

```
supervisorctl update
supervisorctl status
supervisorctl start nginx
supervisorctl restart nginx
supervisorctl stop nginx
```

# Dockerfile自动构建Docker镜像  

## Dockerfile操作命令说明  

Docker通过对于在Dockerfile中的一系列指令的顺序解析实现自动的image的构建
通过使用build命令，根据Dockerfiel的描述来构建镜像
通过源代码路径的方式
通过标准输入流的方式

## Dockerfile指令

```
只支持Docker自己定义的一套指令，不支持自定义
大小写不敏感，但是建议全部使用大写
根据Dockerfile的内容顺序执行
```

## FROM：

```
FROM {base镜像}
必须放在DOckerfile的第一行，表示从哪个baseimage开始构建
```

## MAINTAINER：

```
可选的，用来标识image作者的地方
```

## RUN：

```
每一个RUN指令都会是在一个新的container里面运行，并提交为一个image作为下一个RUN的
```

## base

```
一个Dockerfile中可以包含多个RUN，按定义顺序执行
RUN支持两种运行方式：
RUN <cmd> 这个会当作/bin/sh -c “cmd” 运行
RUN [“executable”，“arg1”，。。]，Docker把他当作json的顺序来解析，因此必须使
用双引号，而且executable需要是完整路径
RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 RUN command1 的执行
仅仅是当前进程，一个内存上的变化而已，其结果不会造成任何文件。而到第二层的时候，启动的是一
个全新的容器，跟第一层的容器更完全没关系，自然不可能继承前一层构建过程中的内存变化。而如果
需要将两条命令或者多条命令联合起来执行需要加上&&。如：cd /usr/local/src && wget
xxxxxxx
```

## CMD：

```
CMD的作用是作为执行container时候的默认行为（容器默认的启动命令）
当运行container的时候声明了command，则不再用image中的CMD默认所定义的命令
一个Dockerfile中只能有一个有效的CMD，当定义多个CMD的时候，只有最后一个才会起作用
CMD定义的三种方式：
CMD <cmd> 这个会当作/bin/sh -c "cmd"来执行
CMD ["executable","arg1",....]
CMD ["arg1","arg2"]，这个时候CMD作为ENTRYPOINT的参数
```

## EXPOSE 声明端口

```
格式为 EXPOSE <端口1> [<端口2>...]。
EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明
应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用
者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也
就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。
```

## ENTRYPOINT：

```
entrypoint的作用是，把整个container变成了一个可执行的文件，这样不能够通过替换CMD的
方法来改变创建container的方式。但是可以通过参数传递的方法影响到container内部
每个Dockerfile只能够包含一个entrypoint，多个entrypoint只有最后一个有效
当定义了entrypoint以后，CMD只能够作为参数进行传递
entrypoint定义方式：
entrypoint ["executable","arg1","arg2"]，这种定义方式下，CMD可以通过json的方式
来定义entrypoint的参数，可以通过在运行container的时候通过指定command的方式传递参数
entrypoint <cmd>，当作/bin/bash -c "cmd"运行命令
```

## ADD & COPY：

```
当在源代码构建的方式下，可以通过ADD和COPY的方式，把host上的文件或者目录复制到image
中
ADD和COPY的源必须在context路径下
当src为网络URL的情况下，ADD指令可以把它下载到dest的指定位置，这个在任何build的方式
下都可以work
ADD相对COPY还有一个多的功能，能够进行自动解压压缩包
```

## ENV：

```
ENV key value
用来设置环境变量，后续的RUN可以使用它所创建的环境变量
当创建基于该镜像的container的时候，会自动拥有设置的环境变量
```

## WORKDIR：

```
用来指定当前工作目录（或者称为当前目录）
当使用相对目录的情况下，采用上一个WORKDIR指定的目录作为基准
```

## USER：

```
指定UID或者username，来决定运行RUN指令的用户
```

## ONBUILD：

```
ONBUILD作为一个trigger的标记，可以用来trigger任何Dockerfile中的指令
可以定义多个ONBUILD指令
当下一个镜像B使用镜像A作为base的时候，在FROM A指令前，会先按照顺序执行在构建A时候定
义的ONBUILD指令
ONBUILD <DOCKERFILE 指令> <content>
```

## VOLUME：

```
用来创建一个在image之外的mount point，用来在多个container之间实现数据共享
运行使用json array的方式定义多个volume
VOLUME ["/var/data1","/var/data2"]
或者plain text的情况下定义多个VOLUME指令
```

# 基于Dockerfile制作nginx镜像

## 创建目录

```
[root@docker-11 ~]# mkdir dockerfile
[root@docker-11 ~]# cd dockerfile/
[root@docker-11 ~/dockerfile]# mkdir nginx_base
[root@docker-11 ~/dockerfile]# cd nginx_base/
```

## 准备文件

```
[root@docker-11 ~/dockerfile/nginx_base]# cat > local.repo << 'EOF'
[local]
name=local
enable=1
gpgcheck=0
baseurl=http://10.0.0.100
EOF
[root@docker-11 ~/dockerfile/nginx_base]# ll
total 8
-rw-r--r-- 1 root root 292 Jul 22 15:01 Dockerfile
-rw-r--r-- 1 root root  65 Jul 22 15:47 local.repo
```

## 编写Dockerfile

```
[root@docker-11 ~/dockerfile/nginx_base]# cat > Dockerfile << 'EOF'
FROM centos:7
RUN rm -rf /etc/yum.repos.d/*
ADD local.repo /etc/yum.repos.d/local.repo
RUN yum makecache fast
RUN yum install nginx -y
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
EOF
```

## 构建镜像

```
[root@docker-11 ~/dockerfile/nginx_base]# cat build.sh 
#!/bin/bash
docker build -t centos7_nginx:1.20 .
[root@docker-11 ~/dockerfile/nginx_base]# bash build.sh 
```

## 启动测试

```
[root@docker-11 ~/dockerfile/nginx_base]# docker run -d -p 10.0.0.11:80:80 centos7_nginx:1.20
[root@docker-11 ~/dockerfile/nginx_base]# curl -I 10.0.0.11
```

# 基于Dockerfile构建云盘镜像

## 先创建目录

```
[root@docker-11 ~]# cd dockerfile/
[root@docker-11 ~/dockerfile]# ls
nginx_base
[root@docker-11 ~/dockerfile]# mkdir kod
[root@docker-11 ~/dockerfile]# cd kod/
[root@docker-11 ~/dockerfile/kod]# 
```

## 收集配置文件

```
mkdir conf
cd conf
cat > local.repo << EOF
[local]
name=local
enable=1
gpgcheck=0
baseurl=http://10.0.0.100
EOF
docker cp b7757c17bf36:/etc/php-fpm.d/www.conf  .
docker cp b7757c17bf36:/etc/nginx/conf.d/cloud.conf  .
docker cp b7757c17bf36:/etc/supervisord.conf  .
docker cp b7757c17bf36:/etc/supervisord.d/nginx_php.ini  .
```

## 准备代码目录

```
mkdir code/
cd code
docker cp b7757c17bf36:/code/ .
tar zcvf code.tar.gz code
```



## 编写Dockerfile

```
FROM centos:7
RUN rm -rf /etc/yum.repos.d/*
ADD conf/local.repo /etc/yum.repos.d/local.repo
RUN yum install nginx php-fpm php-mbstring php-gd supervisor -y
ADD conf/www.conf /etc/php-fpm.d/www.conf
ADD conf/cloud.conf /etc/nginx/conf.d/cloud.conf
ADD conf/supervisord.conf /etc/supervisord.conf
ADD conf/nginx_php.ini /etc/supervisord.d/nginx_php.ini
ADD code/code.tar.gz /
RUN chown -R nginx:nginx /code/
EXPOSE 80
CMD ["supervisord","-c","/etc/supervisord.conf"]
```

## 构建镜像

```
docker build -t kod:v2 .
```

## 启动容器测试

```
docker run -d --name kod_v1 -p 8080:80 kod:v1 
```

# mysql容器的使用

## mysql容器

## 官方镜像地址

```
https://hub.docker.com/_/mysql?tab=description&page=1&ordering=last_updated
```

## 涉及到的环境变量

```
MYSQL_ROOT_PASSWORD		#必选项，设置root账号密码
MYSQL_DATABASE			#创建数据库
MYSQL_USER				#创建普通用户
MYSQL_PASSWORD			#给创建的普通用户设置密码
```

## 使用命令

```
docker run --name mysql_v1 \
-v /data/docker_mysql:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123 \
-d mysql:5.7
```

## 宿主机用户ID和容器内保持一致

```
[root@docker-11 ~]# groupadd -g 1000 mysql
[root@docker-11 ~]# useradd -u 1000 -g 1000 -M -s /sbin/nologin mysql
[root@docker-11 ~]# id mysql
uid=1000(mysql) gid=1000(mysql) groups=1000(mysql)
[root@docker-11 ~]# chown -R mysql:mysql /data/docker_mysql/

docker run --name mysql_v1 \
-v /data/docker_mysql:/var/lib/mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7
```

## 验证数据持久化不会因为容器消失而消失

```
docker run --name mysql_v1 \
-v /data/docker_mysql:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7
```

## mysql容器使用结论

```
官方的mysql容器启动脚本可以判断是空的数据目录还是已经存在数据。
如果是空的数据目录，就按照用户传递的变量初始化数据库
如果数据目录已经有数据了，直接使用原来的数据，用户传递的变量失效了
```

**案例：运行zabbix数据库**

```
docker run --name mysql_v1 \
-p 3306:3306 \
-v /data/docker_mysql:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-e MYSQL_DATABASE=zabbix \
-e MYSQL_USER=zabbix \
-e MYSQL_PASSWORD=zabbix \
-d mysql:5.7
```

# zabbix容器化

## 容器命令

```
docker run --name mysql \
-p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=123 \
-e MYSQL_DATABASE=zabbix \
-e MYSQL_USER=zabbix \
-e MYSQL_PASSWORD=zabbix \
-v /data/docker_mysql:/var/lib/mysql \
-d mysql:5.7 \
--character-set-server=utf8 --collation-server=utf8_bin

docker run --name zabbix-server-mysql \
--link mysql \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-p 10051:10051 \
-d zabbix/zabbix-server-mysql

docker run --name zabbix-web-nginx-mysql \
--link mysql \
--link zabbix-server-mysql \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-e ZBX_SERVER_HOST="zabbix-server-mysql" \
-e PHP_TZ="Asia/Shanghai" \
-p 80:8080 \
-d zabbix/zabbix-web-nginx-mysql
```

## 注释详解

```
docker run \
#mysql服务
--name mysql \

#映射端口 宿主机端口:容器内端口	
-p 3306:3306 \	

#启动Mysql初始化的时候设置root的密码			
-e MYSQL_ROOT_PASSWORD=123 \

#初始化时创建的数据库名称		
-e MYSQL_DATABASE=zabbix \

#初始化时创建的的普通用户，此用户对刚才创建的数据库拥有所有权限			
-e MYSQL_USER=zabbix \	

#普通用户的密码			
-e MYSQL_PASSWORD=zabbix \			

#将容器内的数据持久化到宿主机，如果已经有数据了，用户传递的变量就失效了
-v /data/docker_mysql:/var/lib/mysql \	

#后台启动，使用镜像名称
-d mysql:5.7 \

#设置数据库的字符集
--character-set-server=utf8 --collation-server=utf8_bin

--------------------------------------------------------------------

docker run \
#给zabbix服务端容器起个名字
--name zabbix-server-mysql \

#连接到mysql容器，连接后可以直接使用容器名进行通讯
--link mysql \

#后端mysql的连接地址，这里因为Link了mysql容器，所以可以直接使用容器名通讯
-e DB_SERVER_HOST="mysql" \

#告诉zabbix-server连接mysql使用什么用户
-e MYSQL_USER="zabbix" \

#告诉zabbix-server连接mysql的用户的密码
-e MYSQL_PASSWORD="zabbix" \

#将zabbix-server的服务端口暴露出来，方便客户端连接
-p 10051:10051 \

#使用镜像名称
-d zabbix/zabbix-server-mysql

--------------------------------------------------------------------

docker run \
--name zabbix-web-nginx-mysql \
--link mysql \
--link zabbix-server-mysql \

#连接数据库的地址，因为link了mysql容器，所以可以直接使用容器名通讯
-e DB_SERVER_HOST="mysql" \

#mysql连接的用户
-e MYSQL_USER="zabbix" \

#mysql连接的用户密码
-e MYSQL_PASSWORD="zabbix" \

#zabbix-server的地址，因为link了zabbix-server的容器，所以可以直接使用容器名通讯
-e ZBX_SERVER_HOST="zabbix-server-mysql" \

#修改php的时区为上海
-e PHP_TZ="Asia/Shanghai" \

#映射web网页端口，通过查看镜像信息得知容器内是8080
-p 80:8080 \

#镜像名称
-d zabbix/zabbix-web-nginx-mysql
```

# 基于Dockfile构建LNMP

```
LNMP项目需求：
1.使用Dockerfile制作基于centos7镜像创建nginx+php的容器，使用本地yum源,php的依赖包和软件包私有仓库已经提供好，不用担心
2.基于官方的mysql镜像启动mysql容器
3.创建一个名字叫wordpress的的自定义网络，IP地址为192.168.100.0/24
4.nginx+php和mysql之间使用自定义的wordpress网段
5.将制作好的镜像发送到harbor仓库里
6.docker-12可以从harbor上下载镜像
7.以上需求可以自带笔记，基本环境只提供docker和本地yum仓库



启动一个mysql_wordpress容器
docker run --name mysql_wordpress \
-p 3306:3306 \
-v /data/docker_mysql:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123 \
-e MYSQL_DATABASE=wordpress \
-e MYSQL_USER=wordpress \
-e MYSQL_PASSWORD=wordpress \
-d 10.0.0.253/bighouse/mysql:5.7 \
--character-set-server=utf8 --collation-server=utf8_bin


进入centos7到容器

docker run -it --name nginx_php -p 80:80 --link mysql_wordpress -d 10.0.0.253/bighouse/centos:7

cd /etc/yum.repos.d/
rm -rf *
curl -O http://mirrors.aliyun.com/repo/Centos-7.repo
curl -O http://mirrors.aliyun.com/repo/epel-7.repo
sed -i '/aliyuncs/d' /etc/yum.repos.d/Centos-7.repo

cat > /etc/yum.repos.d/local.repo << 'EOF'

[local]
name=local
enable=1
gpgcheck=0
baseurl=http://10.0.0.11:8080

EOF

cat > /etc/yum.repos.d/nginx.repo << 'EOF' 
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF


yum makecache fast
yum install nginx -y

useradd -u 1009 -M -s /sbin/nologin www


yum install php-fpm php-mbstring php-gd php-mysql -y
sed -i '/^user/c user = nginx' /etc/php-fpm.d/www.conf
sed -i '/^group/c group = nginx' /etc/php-fpm.d/www.conf
sed -i '/daemonize/s#no#yes#g' /etc/php-fpm.conf
php-fpm -c /etc/php.ini -y /etc/php-fpm.conf
nginx
cat > start.sh << 'EOF'
#!/bin/bash
php-fpm -c /etc/php.ini -y /etc/php-fpm.conf
nginx -g 'daemon off;'
EOF
chmod +x start.sh 

cat > /etc/nginx/conf.d/wordpress.conf <<'EOF'
server {
    listen 80;
    server_name localhost;
    root /code/wordpress;
    index index.php index.html;

    location ~ \.php$ {
        root /code;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
EOF

docker cp wordpress  eab328ed08a1:/code


chown -R nginx:nginx /code

重启nginx php-fpm






```



```
FROM 10.0.0.253/bighouse/centos:7


RUN cd /etc/yum.repos.d/
RUN rm -rf *.rpm

ADD repo/local.repo /etc/yum.repos.d

RUN yum makecache fast
RUN yum install nginx -y
RUN yum install php-fpm php-mbstring php-gd php-mysql -y


RUN sed -i '/^user/c user = nginx' /etc/php-fpm.d/www.conf
RUN sed -i '/^group/c group = nginx' /etc/php-fpm.d/www.conf
RUN sed -i '/daemonize/s#no#yes#g' /etc/php-fpm.conf

RUN rm -rf /etc/nginx/conf.d/*
ADD conf/wordpress.conf /etc/nginx/conf.d
RUN useradd -u 1009 -M -s /sbin/nologin www



ADD scripts/start.sh /root
RUN chmod +x /root/start.sh

ADD code/wordpress-4.9.4-zh_CN.tar.gz /code
RUN chown -R nginx:nginx /code

EXPOSE 80
CMD ["/bin/bash","/root/start.sh"]
```

```
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
	user: 2000:2000
    environment:
      - "MYSQL_ROOT_PASSWORD=123"
      - "MYSQL_DATABASE=wordpress"
	  - "MYSQL_USER=wordpress"
	  - "MYSQL_PASSWORD=wordpress"
    volumes:
      - "/data/wordpress:/var/lib/mysql"
    ports:
      - "3306:3306"
    command:
      --character-set-server=utf8 
      --collation-server=utf8_bin
	  
  nginx_php:
    image: nginx_php:v1
    container_name: nginx_php
    ports:
      - "80:80"
	  
networks:
  default:
    external: true
    name: wordpress	  
```



# 运行在host模式下的zabbix容器

```
docker run --name mysql \
--network host \
-e MYSQL_ROOT_PASSWORD=123 \
-e MYSQL_DATABASE=zabbix \
-e MYSQL_USER=zabbix \
-e MYSQL_PASSWORD=zabbix \
-v /data/docker_mysql:/var/lib/mysql \
-d mysql:5.7 \
--character-set-server=utf8 --collation-server=utf8_bin

docker run --name zabbix-server-mysql \
--network host \
-e DB_SERVER_HOST="127.0.0.1" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-d zabbix/zabbix-server-mysql

docker run --name zabbix-web-nginx-mysql \
--network host \
-e DB_SERVER_HOST="127.0.0.1" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-e ZBX_SERVER_HOST="127.0.0.1" \
-e PHP_TZ="Asia/Shanghai" \
-d zabbix/zabbix-web-nginx-mysql
```

# 运行在自定义模式下的zabbix容器

```
docker network create -d bridge --subnet 192.168.200.0/24 --gateway 192.168.200.1 zabbix-net

docker inspect zabbix-net

docker run --name mysql \
--network zabbix-net \
-e MYSQL_ROOT_PASSWORD=123 \
-e MYSQL_DATABASE=zabbix \
-e MYSQL_USER=zabbix \
-e MYSQL_PASSWORD=zabbix \
-v /data/docker_mysql:/var/lib/mysql \
-d mysql:5.7 \
--character-set-server=utf8 --collation-server=utf8_bin

docker run --name zabbix-server-mysql \
--network zabbix-net \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-p 10051:10051 \
-d zabbix/zabbix-server-mysql

docker run --name zabbix-web-nginx-mysql \
--network zabbix-net \
-e DB_SERVER_HOST="mysql" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-e ZBX_SERVER_HOST="zabbix-server-mysql" \
-e PHP_TZ="Asia/Shanghai" \
-p 80:8080 \
-d zabbix/zabbix-web-nginx-mysql
```

# 使用私有镜像仓库Harbor

## 安装harbor

### docker和docker-compose

```
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### 下载harbor上传到/opt,并解压

```
tar zxf harbor-offline-installer-v1.9.0-rc1.tgz -C /opt/
```

### 修改harbor.yml配置文件

```
hostname = 10.0.0.11
harbor_admin_password = 123
```

```
cd /opt/harbor
vim harbor.yml 
5:hostname: 10.0.0.11
27:harbor_admin_password: 123
```

### 执行install.sh安装harbor

```
bash install.sh
```

## 上传镜像到harbor（客户端-需要上传下载镜像的机器）

### 上传镜像到harbor的命名规则

```
harbor地址/项目名称/镜像名称:版本号
10.0.0.11/linux6/mysql:5.7
```

## 修改docker配置文件，添加harbor仓库地址到白名单

```
[root@docker-11 ~]# cat /etc/docker/daemon.json 
{
  "bip": "192.168.2.1/24",
  "registry-mirrors": ["https://ig2l319y.mirror.aliyuncs.com"],
  "insecure-registries": ["http://10.0.0.11"]
}
[root@docker-11 ~]# systemctl restart docker
```

### 登录harbor

```
[root@docker-11 ~]# docker login 10.0.0.11
Username: admin
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

## 修改镜像名称

```
docker tag mysql:5.7 10.0.0.11/linux6/mysql:5.7
```

## 上传镜像到harbor

```
[root@docker-11 ~]# docker push 10.0.0.11/linux6/mysql:5.7
```

## 下载镜像

```
docker pull 10.0.0.11/linux6/mysql:5.7
```

## 其他台机器拉取镜像

```
cat /etc/docker/daemon.json 
{
  "bip": "192.168.2.1/24",
  "registry-mirrors": ["https://ig2l319y.mirror.aliyuncs.com"],
  "insecure-registries": ["http://10.0.0.11"]
}
systemct restart docker

拉取镜像:
docker pull 10.0.0.11/linux6/mysql:5.7
```

## ###重启harbor（可选）

```
cd /opt/ && docker-compose restart
```



## harbor登陆一直提示密码错误

现象

```
docker login 10.0.0.11 提示gateway错误 或者登陆的时候一直提示密码错误
```



解决方法

```
把harbor持久化数据的目录 /data/ 里面 1000uid的目录删除掉
```



# Docker网络模式

## docker网络的四种模式

```
Host 容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。
Bridge 此模式会为每一个容器分配、设置IP等，并将容器连接到一个docker0虚拟网桥，通过
docker0网桥以及Iptables nat表配置与宿主机通信。
None 此模式关闭了容器的网络功能。
Container 创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口
范围。
```

### Bridge模式

### Host模式

### Container模式

### None模式

### 自定义网络模式

语法

```
#创建自定义网络
docker network create -d <mode> --subnet <CIDR> --gateway <网关> <自定义网络名称
> #
引用自定义网络
docker run --network <自定义网络名称> <镜像名称>
#删除自定义网络
doccker network rm <自定义网络名称或网络ID>


```

\#创建自定义网络  

```
docker network create -d bridge --subnet 192.168.200.0/24 --gateway 192.168.200.1 zabbix-net
```

#查看信息

```
docker inspect my-net
```

#查看网桥

```
brctl show
```

# docker容器单机编排工具

## docker-compose介绍  

```
Compose 是用于定义和运行多容器 Docker 应用程序的工具。
通过Compose，您可以使用YML文件来配置应用程序需要的所有服务。
写好yaml文件之后，只需要运行一条命令，就会按照资源清单里的配置运行相应的容器服务。
```



## docker-compose命令格式  

```
build #构建镜像
bundle #从当前docker compose 文件生成一个以<当前目录>为名称的json格式的Docker
Bundle 备 份文件
config -q #查看当前配置，没有错误不输出任何信息
create #创建服务，较少使用
down #停止和删除所有容器、网络、镜像和卷
events #从容器接收实时事件，可以指定json 日志格式，较少使用
exec #进入指定容器进行操作
help #显示帮助细信息
images #显示镜像信息
kill #强制终止运行中的容器
logs #查看容器的日志
pause #暂停服务
port #查看端口
ps #列出容器
pull #重新拉取镜像，镜像发生变化后，需要重新拉取镜像，较少使用
push #上传镜像
restart #重启服务
rm #删除已经停止的服务
run #一次性运行容器
scale #设置指定服务运行的容器个数
start #启动服务
stop #停止服务
top #显示容器运行状态
unpause #取消暂定
up #创建并启动容器
```

## docker-compose语法介绍  

官方英文参考文档：  

```
https://github.com/compose-spec/compose-spec/blob/master/spec.md
```

菜鸟教程翻译文档:  

```
https://github.com/compose-spec/compose-spec/blob/master/spec.md  
```

```
version: '版本号'
services:
服务名称1:
image: 容器镜像
container_name: 容器名称
environment:
- 环境变量1=值1
- 环境变量2=值2
volumes:
- 存储驱动1:容器内的数据目录路径
- 宿主机目录路径:容器内的数据目录路径
ports:
- 宿主机端口:映射到容器内的端口
networks:
- 自定义网络的名称
links:
- namenode
服务名称2:
image: 容器镜像
container_name: 容器名称
environment:
- 环境变量1=值1
- 环境变量2=值2
volumes:
- 存储驱动2:对应容器内的数据目录路径
ports:
- 宿主机端口:映射到容器内的端口
networks:
- 自定义网络的名称
links:
- namenode
networks:
default:
external: true
name: 自定义网络名称
```



## docker-compose部署zabbix

官方文档：  

```
https://www.zabbix.com/documentation/5.0/zh/manual/installation/containers
```

使用自定义网络

```
docker network create -d bridge --subnet 172.16.1.0/24 --gateway 172.16.1.1
zabbix-net
```

修改后的docker-compose文件

```
#创建网段
docker network create -d bridge --subnet 192.168.200.0/24 --gateway 192.168.200.1 zabbix-net

#docker-compose文件
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
	user: 2000:2000
    environment:
      - "MYSQL_ROOT_PASSWORD=123"
      - "MYSQL_DATABASE=zabbix"
	  - "MYSQL_USER=zabbix"
	  - "MYSQL_PASSWORD=zabbix"
    volumes:
      - "/data/docker_mysql:/var/lib/mysql"
    ports:
      - "3306:3306"
    command:
      --character-set-server=utf8 
      --collation-server=utf8_bin
	networks:
      - zabbix-net 
	 
  zabbix-server-mysql:
    image: zabbix/zabbix-server-mysql
    container_name: zabbix-server-mysql
    environment:
      - "DB_SERVER_HOST=mysql"
      - "MYSQL_USER=zabbix"
      - "MYSQL_PASSWORD=zabbix"
    ports:
      - "10051:10051"
    depends_on:
      - mysql
	networks:
      - zabbix-net 
	  
  zabbix-web-nginx-mysql:
    image: zabbix/zabbix-web-nginx-mysql
    container_name: zabbix-web-nginx-mysql
    environment:
      - "DB_SERVER_HOST=mysql"
      - "MYSQL_USER=zabbix"
      - "MYSQL_PASSWORD=zabbix"
      - "ZBX_SERVER_HOST=zabbix-server-mysql"
      - "PHP_TZ=Asia/Shanghai"
    ports:
      - "80:8080"
	networks:
      - zabbix-net 
	  
networks:
  default:
    external: true
    name: zabbix-net
```



# Docker容器跨主机通信

## 跨主机通信类型

```
静态路由
flannel
Overlay
macvlan
calico
```

### 静态路由模式

### 跨主机通信-flannel实现  

### macvlan模式  

### 跨主机通信-Consul实现  



# 基于容器化后的代码上线

## Docker in Docker

### 如何在jenkins容器里里运行docker命令

#### 方法1：更新源并安装docker-ce

#更新源 

```
cat > /etc/apt/sources.list << 'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free
EOF
```

```
apt update
```



安装docker

```
apt -y install apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg |apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian \
   $(lsb_release -cs) \
   stable"
apt update
apt install docker-ce -y
```



#启动docker

```
/usr/bin/dockerd
```



#### 方法2

```
#启动容器的时候加上以下参数:
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/bin/docker:/usr/bin/docker \
```





### 如何指定root用户运行jenkins容器

使用root用户运行容器，进入容器的时候才能以root身份运行，才能正常使用

```
docker run \
--name jenkins \
-p 8080:8080 -p 50000:50000 \
--privileged=true \
--user root \
-v /data/jenkins:/var/jenkins_home \
-d jenkins/jenkins
```



### jenkins容器化需要特别注意的点

```
ssh认证:把密钥的文件目录持久化到宿主机
harbor私有镜像仓库:/root/.docker和/etc/docker/daemon.jason
```



### jenkins容器化完整的配置文件：

```yaml
version: '3'
services:
  gitlab:
    image: 'jenkins/jenkins:latest'
    container_name: jenkins 
    restart: always
    privileged: true
    user: root
    ports:
      - '8080:8080'
      - '50000:50000'
    volumes:
      - '/data/jenkins:/var/jenkins_home'
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/usr/bin/docker:/usr/bin/docker'
      - '/root/.ssh:/root/.ssh'
      - '/root/.docker:/root/.docker'
      - '/etc/docker/daemon.jason:/etc/docker/daemon.jason'

```





## docker部署gitlab





### 容器化gitlab遇到的坑

1.默认密码不知道，进入容器修改root密码

```
docker exec -it gitlab-ce /bin/bash
gitlab-rails console
user = User.where(username: 'root').first
user.password = 'admin-123'
user.save!
exit
```



2.即使添加了公钥，克隆代码需要密码，需要修改配置文件

```
解决方法1：直接修改配置文件然后重新启动
external_url 'http://10.0.0.11'
gitlab_rails['gitlab_shell_ssh_port'] = 2222
alertmanager['enable'] = false
grafana['enable'] = false
prometheus['enable'] = false
node_exporter['enable'] = false
redis_exporter['enable'] = false
postgres_exporter['enable'] = false
pgbouncer_exporter['enable'] = false
gitlab_exporter['enable'] = false


解决方法2：docker-compose文件直接添加变量
version: '3'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.example.com'
	container_name: gitlab 
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://10.0.0.201'
        gitlab_rails['gitlab_shell_ssh_port'] = 2222
	    alertmanager['enable'] = false
        grafana['enable'] = false
        prometheus['enable'] = false
        node_exporter['enable'] = false
        redis_exporter['enable'] = false
        postgres_exporter['enable'] = false
        pgbouncer_exporter['enable'] = false
        gitlab_exporter['enable'] = false
    ports:
      - '9090:80'
      - '2222:22'
    volumes:
      - '/data/gitlab/config:/etc/gitlab'
      - '/data/gitlab/logs:/var/log/gitlab'
      - '/data/gitlab/data:/var/opt/gitlab'
```

3.优化不需要的启动服务，需要修改配置文件

```
参考上面的docker-compose
```

4.公钥key写谁的？

```
version: '3'
services:
  jenkins:
    image: 'jenkins/jenkins:latest'
    container_name: jenkins 
    restart: always
    privileged: true
    user: root
    ports:
      - '8080:8080'
      - '50000:50000'
    volumes:
      - '/data/jenkins:/var/jenkins_home'
      - '/var/run/docker.sock:/var/run/docker.sock'
      - '/usr/bin/docker:/usr/bin/docker'
      - '/root/.ssh:/root/.ssh'
```

### gitlab容器化完整的配置文件：

```yaml
version: '3'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://10.0.0.11'
        gitlab_rails['gitlab_shell_ssh_port'] = 2222
        alertmanager['enable'] = false
        grafana['enable'] = false
        prometheus['enable'] = false
        node_exporter['enable'] = false
        redis_exporter['enable'] = false
        postgres_exporter['enable'] = false
        pgbouncer_exporter['enable'] = false
        gitlab_exporter['enable'] = false
    ports:
      - '9090:80'
      - '2222:22'
    volumes:
      - '/data/gitlab/config:/etc/gitlab'
      - '/data/gitlab/logs:/var/log/gitlab'
      - '/data/gitlab/data:/var/opt/gitlab'  

```



## 容器化后的jenkins参数化构建

### 最简单化版本

```
git clone ssh://git@10.0.0.201:2222/root/docker.git


tar zcvf code.tar.gz * 

cat > Dockerfile << EOF
FROM nginx:latest
ADD code.tar.gz /usr/share/nginx/html/
EOF

docker build -t 10.0.0.201/bighouse/nginx:v1 .
docker push 10.0.0.253/bighouse/nginx:v1
ssh 10.0.0.202 docker pull 10.0.0.201/bighouse/nginx:v1
ssh 10.0.0.202 docker stop app
ssh 10.0.0.202 docker rm app
ssh 10.0.0.202 docker run --name app -it -p 80:80 -d 10.0.0.201/bighouse/nginx:v1

```

### 参数化构建版本

```sh
#1.打包代码
code_tar(){
  tar zcvf code.tar.gz *
}

#2.编写dockerfile
docker_file(){
cat > Dockerfile << EOF
FROM nginx:latest
ADD code.tar.gz /usr/share/nginx/html/
EOF
}


#3.构建镜像
docker_build(){
  docker build -t 10.0.0.11/linux6/nginx:$git_version .
}

#4.推送镜像
docker_push(){
  docker push 10.0.0.11/linux6/nginx:$git_version
}

#5.远程拉取镜像
docker_pull(){
  ssh 10.0.0.12 docker pull 10.0.0.11/linux6/nginx:$git_version
}

#6.停止容器
docker_stop(){
  ssh 10.0.0.12 docker stop app
}

#7.删除容器
docker_rm(){
  ssh 10.0.0.12 docker rm app
}

#9.启动新容器
docker_run(){
  ssh 10.0.0.12 docker run --name app -it -p 80:80 -d 10.0.0.11/linux6/nginx:$git_version
}

#发布逻辑
if [ "$deploy_env" == "deploy" ]
then
  code_tar
  docker_file
  docker_build
  docker_push
  docker_pull
  docker_stop
  docker_rm
  docker_run
else
  docker_stop
  docker_rm
  docker_run
fi

```



## 容器化后的pipeline流水线

### 未优化版本

```sh
harbor准备操作：
创建以下两个仓库
base_image
image

docker-11准备操作：
docker login 10.0.0.11
docker pull nginx:1.17
docker pull nginx:1.18
docker tag nginx:1.17 10.0.0.11/base_image/nginx:1.17
docker tag nginx:1.18 10.0.0.11/base_image/nginx:1.18
docker push 10.0.0.11/base_image/nginx:1.17
docker push 10.0.0.11/base_image/nginx:1.18

docker-12准备操作:
docker login 10.0.0.11
docker push 10.0.0.11/base_image/nginx:1.17
docker run --name game -d 10.0.0.11/base_image/nginx:1.17



流水线脚本：
pipeline{ 
    agent any 
    parameters {
        gitParameter name: 'TAG', 
                     type: 'PT_TAG',
                     defaultValue: 'main'
        choice(name: 'base_image', choices: ['nginx:1.17','nginx:1.18'],description: '请选择基础镜像版本')
        choice(name: 'deploy_env', choices: ['deploy','rollback'],description: 'deploy: 发布版本\nrollback: 回滚版本') 
    }

    stages{
        stage('Example') {
            steps {
                checkout([$class: 'GitSCM', 
                          branches: [[name: "${params.TAG}"]], 
                          doGenerateSubmoduleConfigurations: false, 
                          extensions: [], 
                          gitTool: 'Default', 
                          submoduleCfg: [], 
                          userRemoteConfigs: [[url: 'ssh://git@10.0.0.11:2222/root/demo.git']]
                        ])
            }
        }

        stage("编译镜像"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }
            steps{
                writeFile file: "Dockerfile", text: """FROM 10.0.0.11/base_image/${params.base_image}\nADD game /usr/share/nginx/html/"""
                sh "docker build -t 10.0.0.11/image/game:${params.git_version} . && docker push 10.0.0.11/image/game:${params.git_version}"               
            } 
        }

        stage("推送镜像"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }
            steps{
                sh "docker build -t 10.0.0.11/image/game:${params.git_version} . && docker push 10.0.0.11/image/game:${params.git_version}"               
            }         
        }

        stage("部署容器"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }         
            steps{
                sh 'ssh 10.0.0.12 "docker stop game && docker rm game && docker run --name game -p 80:80 -d 10.0.0.11/image/game:${git_version} && docker ps"'
            }
        } 

        stage("清理构建镜像"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }        
            steps{
                sh "docker rmi 10.0.0.11/image/game:${params.git_version}"
            }
        }

        stage("回滚容器"){
            when {
                environment name: 'deploy_env', value: 'rollback'
            }

            steps{
                sh 'ssh 10.0.0.12 "docker stop game && docker rm game && docker run --name game -p 80:80 -d 10.0.0.11/image/game:${git_version} && docker ps"'
            }         
        }
    }  
}
```

### 优化版本

```sh
pipeline{ 
    agent any 
    parameters {
        gitParameter name: 'git_version', 
                     type: 'PT_TAG',
                     description: '请选择版本：', 
                     defaultValue: 'v1.0'
        choice(name: 'base_image', choices: ['nginx:1.17','nginx:1.18'],description: '请选择基础镜像版本')
        choice(name: 'deploy_env', choices: ['deploy','rollback'],description: 'deploy: 发布版本\nrollback: 回滚版本') 
    }

    stages{
        stage("下载代码"){ 
            steps{
                dir("game"){
                    checkout([$class: 'GitSCM', 
                          branches: [[name: "${params.git_version}"]], 
                          doGenerateSubmoduleConfigurations: false, 
                          extensions: [], 
                          gitTool: 'Default', 
                          submoduleCfg: [], 
                          userRemoteConfigs: [[url: 'ssh://git@10.0.0.11:2222/root/game.git']]
                        ])
                }
            }
        } 

        stage("编译镜像"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }
            steps{
                writeFile file: "Dockerfile", text: """FROM 10.0.0.11/base_image/${params.base_image}\nADD game /usr/share/nginx/html/"""
                sh "docker build -t 10.0.0.11/image/game:${params.git_version} . && docker push 10.0.0.11/image/game:${params.git_version}"               
            } 
        }

        stage("推送镜像"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }
            steps{
                sh "docker build -t 10.0.0.11/image/game:${params.git_version} . && docker push 10.0.0.11/image/game:${params.git_version}"               
            }         
        }

        stage("部署容器"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }         
            steps{
                sh 'ssh 10.0.0.12 "echo git_version=${git_version} > /root/.env"'
                sh 'ssh 10.0.0.12 docker-compose -f /root/docker-compose.yml up -d'
            }
        } 

        stage("清理构建镜像"){
            when {
                environment name: 'deploy_env', value: 'deploy'
            }        
            steps{
                sh "docker rmi 10.0.0.11/image/game:${params.git_version}"
            }
        }

        stage("回滚容器"){
            when {
                environment name: 'deploy_env', value: 'rollback'
            }

            steps{
                sh 'ssh 10.0.0.12 "echo git_version=${git_version} > /root/.env"'
                sh 'ssh 10.0.0.12 docker-compose -f /root/docker-compose.yml up -d'
            }         
        }
    }  
}
```

```
发布版本时
echo "v5.0" > .env


cat .env 
TAG=v3.0
```

```yaml
cat docker-compose.yml


version: '3'
services:
  game:
    image: '10.0.0.11/images/game:${TAG}'
    restart: always
    container_name: game
    privileged: true

    ports:
      - '80:80'

```

![](https://blogjinglobal.oss-cn-beijing.aliyuncs.com/sz-linux/20210729001208.png)

![](https://blogjinglobal.oss-cn-beijing.aliyuncs.com/sz-linux/20210728114436.png)



