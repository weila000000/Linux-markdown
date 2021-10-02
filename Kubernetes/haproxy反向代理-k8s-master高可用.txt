#安装docker
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce-19.03.6 docker-ce-cli-19.03.6 containerd.io

#开机启动docker
systemctl enable docker
systemctl start  docker
systemctl status docker


#编写haproxy的配置文件
global

    # 设置日志文件输出定向
    log 127.0.0.1 local3 info

    # 改变当前工作目录
    chroot /usr/local/haproxy

    # 用户与用户组
    #user haproxy
    #group haproxy

    # 守护进程启动，运维方式为后台工作
    daemon

    # 最大连接数
    maxconn 4000
    #pid文件位置
    pidfile /var/run/haproxy.pid
# 作用于其后紧跟的listen块，直至下一个defaults 块，下一个default 将替换上一个块作用于以后的listen
defaults

    # 启用每个实例日志记录事件和流量。
    log global

    # 默认的模式mode { tcp|http|health }，tcp是4层，http是7层，health只会返回OK
    mode http
    #option forwardfor    #如果后端服务器需要获得客户端的真实ip，需要配置的参数，记录客户端IP在X-Forwarded-For头域中，这里设置的话在backend段里面则不需要设置

    # maxconn 65535         maxconn 每个进程可用的最大连接数
    # retries 3         当对server的connection失败后，重试的次数 　
    # option abortonclose     启用或禁用在队列中挂起的中止请求的早期丢弃
    # option redispatch     启用或禁用在连接故障情况下的会话重新分配
    # option dontlognull     启用和禁用 记录 空连接
    # option httpclose         每次请求完毕后主动关闭http通道，HA-Proxy不支持keep-alive模式
    # option forwardfor     获得客户端IP
    # option httplog        记录HTTP 请求,session 状态和计时器

    option abortonclose #当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接
    option redispatch #当serverid对应的服务器挂掉后,强制定向到其他健康的服务器

    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000
listen  k8s-control-plane
    #访问的IP和端口
    bind  0.0.0.0:8888
    #网络协议
    mode  tcp
    #负载均衡算法（轮询算法）
    #轮询算法：roundrobin
    #权重算法：static-rr
    #最少连接算法：leastconn
    #请求源IP算法：source
    balance  roundrobin
    #日志格式
    option  tcplog
    #在MySQL中创建一个没有权限的haproxy用户，密码为空。Haproxy使用这个账户对MySQL数据库心跳检测
    server  master-01 192.168.0.18:6443 check weight 1 maxconn 2000
    server  master-02 192.168.0.19:6443 check weight 1 maxconn 2000
    server  master-03 192.168.0.20:6443 check weight 1 maxconn 2000




#编写Dockerfile
FROM haproxy:1.7
RUN mkdir -p /usr/local/haproxy/
COPY ./haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg

#build镜像
docker build -t my-haproxy .

#run运行镜像
docker run -it --name haproxy -p 8888:8888 --privileged -d my-haproxy






