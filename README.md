# frp

服务端配置

1 登陆vps主机
```
ssh root@12.12.12.12
```
 

2 下载一键安装包
```
wget --no-check-certificate https://raw.githubusercontent.com/clangcn/onekey-install-shell/master/frps/install-frps.sh -O ./install-frps.sh
```
3 chmod 700

```
chmod 700 ./install-frps.sh
```
 

4 安装
```
./install-frps.sh install
```

安装成功截图

![frps安装截图](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171226155743729-1396171587.png)

5 浏览器输入http://12.12.12.12:6443 查看frps Dashboard

![frps Dashboard](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171222165529850-183084607.png)

```
frps 常用命令：启动（frps start），停止（frps stop）

frps status manage : {start|stop|restart|status|config|version}

Example:

  start: frps start

   stop: frps stop

restart: frps restart
```
6 修改frps 配置文件

方法一：
```
frps config
```
方法二：
```
cd /usr/local/frps/
vi frps.ini
```
7 frps.ini 配置文件

```
[common]
# 绑定端口号
bind_addr = 0.0.0.0
bind_port = 5443
kcp_bind_port = 5443
# dashboard端口号 用户名 密码
dashboard_port = 6443
dashboard_user = admin
dashboard_pwd = pCSRqw43
# assets_dir = ./static

# 绑定 http，https 端口号 
vhost_http_port = 80
vhost_https_port = 443
# 日志文件路径
log_file = ./frps.log
# debug, info, warn, error 日志打印
log_level = debug
log_max_days = 3
# privilege token 客户端连接使用
privilege_token = wcnk53kvgNm7cdt8
# only allow frpc to bind ports you list, if you set nothing, there won't be any limit
#privilege_allow_ports = 1-65535
# pool_count in each proxy will change to max_pool_count if they exceed the maximum value
max_pool_count = 50
# if tcp stream multiplexing is used, default is true
tcp_mux = true
```

客户端配置

1 根据系统下载对应的版本（https://github.com/fatedier/frp/releases）
```
wget https://github.com/fatedier/frp/releases/download/v0.14.1/frp_0.14.1_linux_arm.tar.gz

tar zxvf frp_0.14.1_linux_arm.tar.gz #解压
```
2 进入frp目录
```
cd frp_0.14.1_linux_arm
```
3 编辑 frpc.ini
```
vi frpc.ini
```
frpc.ini

```
[common]
server_addr = 12.12.12.12#vps主机地址
server_port = 5443
privilege_token = Q5NqGtn4YxjtJz9t
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

4 保存
```
:wq
```

5 启动frpc 客户端
```
./frpc -c frpc.ini
```
6 如图，启动成功

![frpc 启动成功](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171222170044318-162684052.png)

7 使用ssh 远程连接，这个地方要注意：[从外网访问内网，你应该是访问外网的ip，frp帮你打洞到内网](https://github.com/fatedier/frp/issues/97)
```
ssh -oPort=6000 wandou@12.12.12.12#wandou是内网用户名，12.12.12.12是公网ip
```

![ssh连接](https://images2017.cnblogs.com/blog/1044995/201712/1044995-20171222170740162-1712874836.png)

8 配置路由转发

frps.ini
```
# frps.ini
[common]
bind_port = 5443
vhost_http_port = 80#http 端口号 也可以是其他任何端口号
```

启动frps：
```
./frps -c ./frps.ini
```

修改 frpc.ini 文件，假设 frps 所在的服务器的 IP 为 x.x.x.x，local_port 为本地机器上 web 服务对应的端口, 绑定自定义域名 cc.yourdomain.com:

```
# frpc.ini
[common]
server_addr = x.x.x.x
server_port = 5443 #和服务端对应

[web1]#web1名称不能重复
type = http
local_port = 8080#端口号对应本机web服务器端口
custom_domains = cc.yourdomain.com

[web2]#web2名称不能重复
type = http
local_port = 8082#端口号对应本机web服务器端口
custom_domains = bb.yourdomain.com#可以配置多个子域名
```

`出现问题：the page you visit not found.
Sorry, the page you are looking for is currently unavailable.
Please try again later.
The server is powered by frp.`[解决方法：https://github.com/fatedier/frp/issues/506](https://github.com/fatedier/frp/issues/506)

启动 frpc：
```
./frpc -c ./frpc.ini
```

将 yourdomain.com 的域名 A 记录解析到 IP x.x.x.x，如果服务器已经有对应的域名，也可以将 CNAME 记录解析到服务器原先的域名

通过浏览器访问 http://cc.yourdomain.com 即可访问到处于内网机器上的 web 服务。

如果 frps.ini文件 vhost_http_port = 端口号#http 不是80端口

请通过：http://cc.yourdomain.com:端口号 即可访问到处于内网机器上的 web 服务
 

[参考链接1：内网穿透 frp](http://www.cnblogs.com/mnstar/p/8085113.html)

[参考链接2：frp原作 GitHub](https://github.com/fatedier/frp/)


