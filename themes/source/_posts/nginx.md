---
title: Nginx
tags: nginx
categories: nginx
---

# Nginx

## 源码安装

1. 安装依赖
    ```bash
    # cat > /etc/apt/sources.list << EOF
    # deb http://cn.archive.ubuntu.com/ubuntu/ trusty main restricted
    # deb http://cn.archive.ubuntu.com/ubuntu/ trusty-updates main restricted
    # deb http://cn.archive.ubuntu.com/ubuntu/ trusty universe
    # deb http://cn.archive.ubuntu.com/ubuntu/ trusty-updates universe
    # EOF

    apt-get update

    apt-get -y install build-essential
    apt-get -y install libpcre++-dev libssl-dev make \
        libxml2-dev libxslt-dev libgd-dev libgeoip-dev \
        libgoogle-perftools-dev libatomic-ops-dev libperl-dev
    ```

2. 下载源码
    ```bash
    cd /usr/local/src

    curl -O http://nginx.org/download/nginx-1.15.0.tar.gz

    # 解压
    tar zxvf nginx-1.15.0.tar.gz
    ```

3. 新建 nginx 用户及用户组
    ```bash
    groupadd nginx
    useradd -g nginx -M nginx -s /sbin/nologin
    ```

4. 编译配置、编译、安装  
    ```bash
    cd /usr/local/src/nginx*

    ./configure --prefix=/usr/local/nginx \
        --pid-path=/usr/local/nginx/run/nginx.pid \
        --with-http_ssl_module \
        --user=nginx \
        --group=nginx \
        --with-pcre \
        --without-mail_pop3_module \
        --without-mail_imap_module \
        --without-mail_smtp_module
    
    make
    make install
    ```
    > --prefix=/usr/local/nginx 指定安装到 /usr/local/nginx 目录下

5. 查看安装后的程序版本  
    `/usr/local/nginx/sbin/nginx -v`

## 配置

> [可视化配置并生成配置文件](https://nginxconfig.io/)

编辑配置文件 /usr/local/nginx/conf/nginx.conf

### 标准配置

```nginx
events {
    worker_connections  1024;
}

http {
    server {
        listen      80;
        server_name localhost;
        location / {
            root    html; # /usr/local/nginx/html
            index   index.html index.htm; # /usr/local/nginx/html/index.html
        }
    }
}
```

> curl http://127.0.0.1:80/

### [虚拟目录配置](https://www.cnblogs.com/kevingrace/p/6187482.html)

```nginx
events {
    worker_connections  1024;
}

http {
    server {
        listen      80;
        server_name localhost;
        location / {
            root    html; # /usr/local/nginx/html
            index   index.html index.htm; # /usr/local/nginx/html/index.html
        }

        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }

            # root 响应的路径：配置的路径 + 完整访问路径(完整的 location 配置路径 + 静态文件)
            # root /usr/share/nginx/html; # "/usr/share/nginx/html" + "/hls/oceans.m3u8"
            # alias 响应的路径：配置路径 + 静态文件(去除 location 中配置的路径)
            alias /usr/share/nginx/html/hls/; # "/usr/share/nginx/html/hls/" + "oceans.m3u8"

            # expires -1;
            add_header Cache-Control no-cache;
        }
    }
}
```
> - 使用 alias 时目录名后面一定要加 “/”
> - 一般情况下，在 location / 中配置 root，在 location /other 中配置 alias

### HTTPS 加密

生成 CA 自签名证书并签发服务器证书

```bash
cd /var/ssl

# 生成 CA 自签名证书
openssl genrsa -out ca.key 4096
openssl req -new -x509 -days 7300 -key ca.key -subj "/C=CN/ST=JS/L=NJ/O=Sunke, Inc./CN=Sunke Root CA" -out ca.crt

# 生成服务器证书
openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=JS/L=NJ/O=Sunke, Inc./CN=localhost" -out server.csr
openssl x509 -sha256 -req -extfile <(printf "subjectAltName=DNS:localhost,IP:127.0.0.1") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt

cat server.crt ca.crt > server.b.crt # 证书链
```

在 `/http` 节点下，配置 `server` 如下

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    ssl_protocols TLSv1.2; # 支持 TLSv1.2 协议

    ssl_certificate /var/ssl/server.b.crt; # server 公钥，需使用绝对路径
    ssl_certificate_key /var/ssl/server.key; # server 私钥

    location / {
        ...
    }
}
```

### HTTPS 双向认证

生成客户端证书

```bash
cd /var/ssl

openssl req -newkey rsa:2048 -nodes -keyout client.key -subj "/C=CN/ST=JS/L=NJ/O=/CN=" -out client.csr
openssl x509 -sha256 -req -days 365 -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt
```

在 `/http` 节点下，配置 `server` 如下

```nginx
server {
    listen 443 ssl;
    server_name localhost;

    ssl_protocols TLSv1.2; # 支持 TLSv1.2 协议

    ssl_certificate /var/ssl/server.b.crt; # server 公钥
    ssl_certificate_key /var/ssl/server.key; # server 私钥

    ssl_client_certificate /var/ssl/ca.crt; # 根级证书公钥，用于验证各个二级 client
    ssl_verify_client on;

    location / {
        ...

        # proxy_set_header X-SSL-Client-Cert $ssl_client_cert; # 客户端证书内容信息
        # proxy_set_header X-SSL-serial $ssl_client_serial; # 客户端证书序列号
        # proxy_set_header cert-subject $ssl_client_s_dn; # 客户端证书 subject 信息
    }
}
```

### 负载均衡

在 `/http` 节点下，新增 `upstream` 配置如下

```nginx
http {
    upstream myproject {
        server 127.0.0.1:8000 weight=3;
        server 127.0.0.1:8001;
        server 127.0.0.1:8002;
        server 127.0.0.1:8003;
        ip_hash; # 配置 ip hash，可用于解决 session 不同步的问题
    }       
    server {
        listen 80;
        server_name www.domain.com;
        location / {
            proxy_pass http://myproject;
        }
    }
}
```

### 双机热备

在 `/http` 节点下，新增 `upstream` 配置如下

```nginx
http {
    upstream myserver {
        server 192.168.1.101:8080;
        server 127.0.0.1:8080 backup;
    }
    server {
        listen 8000;
        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

            proxy_pass http://myserver;
        }
    }
}
```

### 配置静态内容服务器

在 `/http/server` 节点下，配置 `location` 如下

```nginx
location / {
    root   html;
    index  index.html index.htm;
}
location /image/ {
    root   /usr/local/myImage/;
    autoindex on;
}
```

### 反向代理

在 `/http/server` 节点下，配置 `location` 如下

```nginx
location / {
    proxy_pass http://127.0.0.1:8080; # 反向代理服务器的地址
    index index.html index.htm;  
}
location /a {
    proxy_pass http://127.0.0.1:8081;
    proxy_set_header Host $host:$proxy_port; # 区别于 add_header 只能用于新增响应消息头，proxy_set_header 用于设置 Nginx 转发至上游服务的请求消息头
}
location /b {
    proxy_pass http://192.168.1.102:8081/b;
    proxy_cookie_path /a /b;
}

location /api {
    # 代理转发
    proxy_pass http://192.168.1.101:3000/; # 如果在 proxy_pass 中的 url 后面加 /，表示绝对根路径，如 http://127.0.0.1:80/api/greeting => http://192.168.1.101:3000//greeting
    # proxy_pass http://192.168.1.101:3000; # 否则表示相对路径，把匹配的路径部分也给代理走，如 http://127.0.0.1:80/api/greeting => http://192.168.1.101:3000/api/greeting
}
# location /api/ {
#     proxy_pass http://192.168.1.101:3000/; # 绝对路径，如 http://127.0.0.1:80/api/greeting => http://192.168.1.101:3000/greeting
#     proxy_pass http://192.168.1.101:3000; # 相对路径，如 http://127.0.0.1:80/api/greeting => http://192.168.1.101:3000/api/greeting
#     proxy_pass http://192.168.1.101:3000/user/; # 如 http://127.0.0.1:80/api/greeting => http://192.168.1.101:3000/user/greeting
#     proxy_pass http://192.168.1.101:3000/user; # 如 http://127.0.0.1:80/api/greeting => http://192.168.1.101:3000/usergreeting
# }
```

#### 修改上游服务的响应头

```nginx
location / {
    proxy_hide_header X-Frame-Options; # Nginx 会将上游服务器的响应转发给客户端，但默认不会转发以下 HTTP 头部字段：Date、Server、X-Pad和X-Accel-*。使用 proxy_hide_header 后可以任意地指定哪些 HTTP 头部字段不能被转发。
    add_header X-Frame-Options SAMEORIGIN; # 新增响应头
    proxy_pass http://127.0.0.1:8080; # 反向代理服务器的地址  
}
```

### 压缩和解压

在 `/http` 节点下，配置 `server` 如下

```nginx
server {
    gzip on;
    gzip_types      text/plain application/xml;
    gzip_proxied    no-cache no-store private expired auth;
    gzip_min_length 1000;
    ...
}
```

### 内容缓存

在 `/http` 节点下，配置 `server` 如下

```nginx
server {
    listen 80;
    server_name localhost;
    location ~ .*.(html|htm)$ {
        # 缓存开关
        expires 24h;
        root /home/project/nginx-code;
    }
}
```

### 配置日志

在 `/` 节点下，配置 `http` 如下

```nginx
http {
    ...
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"'
                    '$upstream_addr $upstream_response_time $request_time';

    access_log  logs/access.log  main;
    ...
}
```

### 跨域配置

在 `/http/server` 节点下，配置 `location` 如下

```nginx
server {
    listen 80;
    server_name localhost;
    location ~ .*.(html|htm)$ {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS;
        root /home/project/nginx-code;
    }
}
```

### 防盗链

在 `/http/server` 节点下，配置 `location` 如下

```nginx
location ~* \.(gif|jpg|png|swf|flv)$ { # 表示对 gif、jpg、png、swf、flv 后缀的文件实行防盗链
    root html # 如果在 server {...} 中有设置，这里可以不设置
    valid_referers none blocked 127.0.0.1 *.nginxcn.com; # 表示对 127.0.0.1、*.nginxcn.com 这 2 个 url 进行判断
    if ($invalid_referer) {
        rewrite ^/ www.nginx.cn # 如果不是指定 url 就跳转到 www.nginx.cn
        # return 403; # 或这可以直接返回 403 状态码
    }
}
```
> valid_referers 指令后面可以接 none、blocked、serevr_names、string 或正则表达式
> - none 代表没有 referer
> - blocked 代表有 referer 但是被防火墙或者是代理给去除了
> - string 或正在表达式，用来匹配 referer

### 配置使用 WebSocket

在 `/http/server` 节点下，配置 `location` 如下

```nginx
location /api/ {
    proxy_pass ws://127.0.0.1:3000/;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### 获取自定义 cookie 和 header，设置 header

在 `/http` 节点下，配置 `server` 如下

```nginx
location /api/ {
    set $token "";
    if ( $http_cookie ~* "X-Token=(.+?)(?=;|$)" ) {
        set $token $1; # 获取请求 cookie 中的 X-Token 值
    }
    if ( $http_x_token != "" ) {
        set $token $http_x_token; # 获取请求 header 中的 X-Token 值
    }
    proxy_set_header X-Auth-Token $token;
    proxy_pass http://127.0.0.1:3000/;
}
```

### 子域名

#### 方法一：使用正则将子域名的请求映射到对应目录中

在 `/http/server` 节点下，配置如下

```nginx
if ( $host ~* (\b(?!www\b)\w+)\.\w+\.\w+ ) {
    set $subdomain /$1;
}
location / {
    root /home/web/public_html$subdomain; # 如 www.myserver.com 映射目录 /home/web/public_html，blog.myserver.com 映射目录 /home/web/public_htmlblog
    index index.html index.php;
}
```

#### 方法二：

在 `/http` 节点下，配置 `server` 如下

```nginx
server {
    listen      80;
    server_name www.myserver.com myserver.com;
    location / {
        root    html;
        index   index.html;
    }
}
server {
    listen      80;
    server_name blog.myserver.com;
    location / {
        root    html;
        index   index.html;
    }
}

# 动态配置如下
server {
    listen      80;
    server_name ~$ (.*).myserver.com;
    root        $root_path;
    if ( $host ~* blog.myserver.com ) {
        set root_path '/home/web/public_htmlblog';
    }
}
```

修改 `/etc/hosts` 文件（windows 下位于 C:\Windows\System32\drivers\etc\hosts）如下

```
127.0.0.1    blog.myserver.com
```

### [安全配置](https://www.jianshu.com/p/163c26766a80)

- 限制请求率  
    <font color="grey">将 Nginx 可接受的入站请求率限制为某个适合真实用户的值。例如，根据 ip 限制每个用户每两秒钟才能访问一次页面</font>
    ```nginx
    http {
        # limit_req_zone $binary_remote_addr $uri zone=api_read:20m rate=50r/s;
        limit_req_zone $binary_remote_addr zone=one:10m rate=30r/m;

        server {
            location / {
                # limit_req zone=api_read burst=5;
                limit_req zone=one;
                ...
            }
            ...
        }
    }
    ```
    > - limit_req_zone 配置参数：
    >   - `$binary_remote_addr` 表示通过 remote_addr 这个标识来做限制，"binary_" 的目的是缩写内存占用量，是限制同一客户端 ip 地址
    >   - `zone=api_read:20m` 表示生成一个大小为 20M，名字为 api_read 的内存区域，用来存储访问的频次信息
    >   - `rate=50r/s` 表示允许相同标识的客户端的访问频次，这里限制的是每秒 50 次，还可以有比如 30r/m 的
    > - limit_req 配置参数：
    >   - `zone=api_read` 设置使用哪个配置区域来做限制，与上面 limit_req_zone 里的 name 对应
    >   - `burst=5`（可选，默认为 0），burst（英文为爆发的意思）配置是指设置一个大小为 5 的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内
    >   - `nodelay`（可选），如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回 503，如果没有设置，则所有请求会等待排队
- 限制连接的数量  
    <font color="grey">将某个客户端 ip 地址所能打开的连接数限制为真实用户的合理值。例如，限制每一个 ip 对网站打开的连接数不超过 10 个</font>
    ```nginx
    http {
        # # 按 ip 配置一个连接 zone
        # limit_conn_zone $binary_remote_addr zone=perip_conn:10m;
        
        # # 按 server 配置一个连接 zone
        # limit_conn_zone $server_name zone=perserver_conn:100m;

        limit_conn_zone $binary_remote_addr zone=addr:10m;

        server {
            location / {
                # # 连接数限制,每个 ip 并发请求为 2
                # limit_conn perip_conn 2;
                
                # # 服务所限制的连接数（即限制了该 server 并发连接数量）
                # limit_conn perserver_conn 1000;

                limit_conn addr 10;
                ...
            }
            ...
        }
    }
    ```
- 关闭慢连接  
    <font color="grey">关闭那些一直保持打开同时写数据又特别频繁的连接，因为它们会降低服务器接受新连接的能力。Slowloris 就是这种类型的攻击。对此，可以通过 client_body_timeout 和 client_header_timeout 指令控制请求体或者请求头的超时时间，例如，通过下面的配置将等待时间控制在 5s 之内</font>
    ```nginx
    http {
        ...
        server {
            client_body_timeout 5s;
            client_header_timeout 5s;
            ...
        }
    }
    ```
- 设置 ip 黑名单  
    <font color="grey">如果能识别攻击者所使用的客户端 ip 地址，那么通过 deny 指令将其屏蔽，拒绝来自这些地址的连接或请求。例如，通过下面的指令拒绝来自 123.123.123.3、123.123.123.5 的请求</font>
    ```nginx
    http {
        ...
        server {
            location / {
                deny 123.123.123.3;
                deny 123.123.123.5;
                ...
            }
            ...
        }
    }
    ```
- 设置 ip 白名单  
    <font color="grey">如果允许访问的 ip 地址比较固定，那么通过 allow 和 deny 指令让网站或者应用程序只接受来自于某个 ip 地址或者某个 ip 地址段的请求。例如，通过下面的指令将访问限制为本地网络的一个 ip 段</font>
    ```nginx
    http {
        ...
        server {
            location / {
                allow 192.168.1.0/24;
                deny all;
                ...
            }
            ...
        }
    }
    ```
- 阻塞请求  
    <font color="grey">通过下面的配置阻塞已识别出的 User-Agent 头的值是 foo 或者 bar 的 DDoS 攻击</font>
    ```nginx
    http {
        ...
        server {
            location / {
                if ($http_user_agent ~* foo|bar) {
                    return 403;
                }
                ...
            }
            ...
        }
    }
    ```
- 限制对后端服务器的连接数  
    <font color="grey">例如，通过下面的配置限制每一台后端服务器之间建立的连接数不多于 200 个</font>
    ```nginx
    http {
        ...
        upstream website {
            server 192.168.100.1:80 max_conns=200;
            server 192.168.100.2:80 max_conns=200;
            queue 10 timeout=30s;
            ...
        }
    }
    ```
- 连接限速  
    <font color="grey">例如，通过下面的配置限制连接速度小于 100k</font>
    ```nginx
    http {
        ...
        server {
            ...
            location / {
                # 连接限速
                limit_rate 100k;
                ...
            }
        }
    }
    ```

<details>
<summary>官方完整示例</summary>

```nginx
user www www; # 使用的用户和组

worker_processes  2; # 指定工作衍生进程数

pid /var/run/nginx.pid; # 指定 pid 存放的路径

# 错误日志级别，可以为 [ debug | info | notice | warn | error | crit ]
error_log  /var/log/nginx.error_log  info;

events {
    worker_connections   2000;  # 允许的连接数

    # use [ kqueue | epoll | /dev/poll | select | poll ];
    use kqueue; # 具体内容查看 http://wiki.codemongers.com/ 事件模型
}

http {

    include       conf/mime.types;
    default_type  application/octet-stream;


    log_format main      '$remote_addr - $remote_user [$time_local] '
                         '"$request" $status $bytes_sent '
                         '"$http_referer" "$http_user_agent" '
                         '"$gzip_ratio"';

    log_format download  '$remote_addr - $remote_user [$time_local] '
                         '"$request" $status $bytes_sent '
                         '"$http_referer" "$http_user_agent" '
                         '"$http_range" "$sent_http_content_range"';

    client_header_timeout  3m;
    client_body_timeout    3m;
    send_timeout           3m;

    client_header_buffer_size    1k;
    large_client_header_buffers  4 4k;

    gzip on;
    gzip_min_length  1100;
    gzip_buffers     4 8k;
    gzip_types       text/plain;

    output_buffers   1 32k;
    postpone_output  1460;

    sendfile         on;
    tcp_nopush       on;
    tcp_nodelay      on;
    send_lowat       12000;

    keepalive_timeout  75 20;

    #lingering_time     30;
    #lingering_timeout  10;
    #reset_timedout_connection  on;


    server {
        listen        one.example.com;
        server_name   one.example.com  www.one.example.com;

        access_log   /var/log/nginx.access_log  main;

        location / {
            proxy_pass         http://127.0.0.1/;
            proxy_redirect     off;

            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;
            #proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;

            client_max_body_size       10m;
            client_body_buffer_size    128k;

            client_body_temp_path      /var/nginx/client_body_temp;

            proxy_connect_timeout      70;
            proxy_send_timeout         90;
            proxy_read_timeout         90;
            proxy_send_lowat           12000;

            proxy_buffer_size          4k;
            proxy_buffers              4 32k;
            proxy_busy_buffers_size    64k;
            proxy_temp_file_write_size 64k;

            proxy_temp_path            /var/nginx/proxy_temp;

            charset  koi8-r;
        }

        error_page  404  /404.html;

        location /404.html {
            root  /spool/www;
        }

        location /old_stuff/ {
            rewrite   ^/old_stuff/(.*)$  /new_stuff/$1  permanent;
        }

        location /download/ {

            valid_referers  none  blocked  server_names  *.example.com;

            if ($invalid_referer) {
                #rewrite   ^/   http://www.example.com/;
                return   403;
            }

            #rewrite_log  on;

            # rewrite /download/*/mp3/*.any_ext to /download/*/mp3/*.mp3
            rewrite ^/(download/.*)/mp3/(.*)\..*$
                    /$1/mp3/$2.mp3                   break;

            root         /spool/www;
            #autoindex    on;
            access_log   /var/log/nginx-download.access_log  download;
        }

        location ~* \.(jpg|jpeg|gif)$ {
            root         /spool/www;
            access_log   off;
            expires      30d;
        }
    }
}
```

</details>

## 命令

```bash
# 验证配置文件是否合法
/usr/local/nginx/sbin/nginx -t

# 启动 nginx
/usr/local/nginx/sbin/nginx
# /usr/local/nginx/sbin/nginx -c ./nginx.conf

# 重启 nginx
/usr/nginx/sbin/nginx -s reload

# 停止
nginx -s stop

# 退出
nginx -s quit
```

## Troubleshooting

### nginx 启动后，js、html 可以加载，但是引用的 css 样式失效

- 现象  
    查看引用失效的 css 文件后发现请求头中 Accept 为 text/css，但是响应头中 Content-Type 为 text/plain

- 解决方案  
    1. 在 nginx.conf 文件中 http 内增加配置 `include mime.types;` 如下
        ```nginx
        events {
            worker_connections  1024;
        }

        http {
            include mime.types;

            server {
                listen      80;
                server_name localhost;
                location / {
                    root    html; # /usr/local/nginx/html
                    index   index.html index.htm; # /usr/local/nginx/html/index.html
                }
            }
        }
        ```
    2. 在 /etc/nginx/mime.types 放入配置文件
        ```
        types {
            text/html                                        html htm shtml;
            text/css                                         css;
            text/xml                                         xml;
            image/gif                                        gif;
            image/jpeg                                       jpeg jpg;
            application/javascript                           js;
            application/atom+xml                             atom;
            application/rss+xml                              rss;

            text/mathml                                      mml;
            text/plain                                       txt;
            text/vnd.sun.j2me.app-descriptor                 jad;
            text/vnd.wap.wml                                 wml;
            text/x-component                                 htc;

            image/png                                        png;
            image/svg+xml                                    svg svgz;
            image/tiff                                       tif tiff;
            image/vnd.wap.wbmp                               wbmp;
            image/webp                                       webp;
            image/x-icon                                     ico;
            image/x-jng                                      jng;
            image/x-ms-bmp                                   bmp;

            application/font-woff                            woff;
            application/java-archive                         jar war ear;
            application/json                                 json;
            application/mac-binhex40                         hqx;
            application/msword                               doc;
            application/pdf                                  pdf;
            application/postscript                           ps eps ai;
            application/rtf                                  rtf;
            application/vnd.apple.mpegurl                    m3u8;
            application/vnd.google-earth.kml+xml             kml;
            application/vnd.google-earth.kmz                 kmz;
            application/vnd.ms-excel                         xls;
            application/vnd.ms-fontobject                    eot;
            application/vnd.ms-powerpoint                    ppt;
            application/vnd.oasis.opendocument.graphics      odg;
            application/vnd.oasis.opendocument.presentation  odp;
            application/vnd.oasis.opendocument.spreadsheet   ods;
            application/vnd.oasis.opendocument.text          odt;
            application/vnd.openxmlformats-officedocument.presentationml.presentation
                                                            pptx;
            application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
                                                            xlsx;
            application/vnd.openxmlformats-officedocument.wordprocessingml.document
                                                            docx;
            application/vnd.wap.wmlc                         wmlc;
            application/x-7z-compressed                      7z;
            application/x-cocoa                              cco;
            application/x-java-archive-diff                  jardiff;
            application/x-java-jnlp-file                     jnlp;
            application/x-makeself                           run;
            application/x-perl                               pl pm;
            application/x-pilot                              prc pdb;
            application/x-rar-compressed                     rar;
            application/x-redhat-package-manager             rpm;
            application/x-sea                                sea;
            application/x-shockwave-flash                    swf;
            application/x-stuffit                            sit;
            application/x-tcl                                tcl tk;
            application/x-x509-ca-cert                       der pem crt;
            application/x-xpinstall                          xpi;
            application/xhtml+xml                            xhtml;
            application/xspf+xml                             xspf;
            application/zip                                  zip;

            application/octet-stream                         bin exe dll;
            application/octet-stream                         deb;
            application/octet-stream                         dmg;
            application/octet-stream                         iso img;
            application/octet-stream                         msi msp msm;

            audio/midi                                       mid midi kar;
            audio/mpeg                                       mp3;
            audio/ogg                                        ogg;
            audio/x-m4a                                      m4a;
            audio/x-realaudio                                ra;

            video/3gpp                                       3gpp 3gp;
            video/mp2t                                       ts;
            video/mp4                                        mp4;
            video/mpeg                                       mpeg mpg;
            video/quicktime                                  mov;
            video/webm                                       webm;
            video/x-flv                                      flv;
            video/x-m4v                                      m4v;
            video/x-mng                                      mng;
            video/x-ms-asf                                   asx asf;
            video/x-ms-wmv                                   wmv;
            video/x-msvideo                                  avi;
        }
        ```
    3. 重启 nginx

## 第三方模块

<details>
<summary>使用 nginx-rtmp-module 搭建流媒体服务器</summary>

1. 下载 nginx-rtmp-module  
    ```bash
    # apt-get -y install git

    git clone https://github.com/arut/nginx-rtmp-module.git /usr/local/src/nginx-rtmp-module
    ```
2. 安装 nginx
    ```bash
    # apt-get install openssl openssl-devel

    # cd /usr/local/src
    # curl -O http://nginx.org/download/nginx-1.15.0.tar.gz
    # tar -zxvf nginx-1.15.0.tar.gz
    
    cd /usr/local/src/nginx-1.15.0
    
    make clean

    ./configure --prefix=/usr/local/nginx \
                --add-module=../nginx-rtmp-module \
                --with-http_ssl_module

    make && make install

    mkdir -p /usr/share/nginx/html/hls
    mkdir -p /usr/share/nginx/html/vod
    ```
    > 查看 nginx 版本、编译参数信息 `/usr/local/nginx/sbin/nginx -V`
3. 修改 nginx 配置文件  
    `vi /usr/local/nginx/conf/nginx.conf`  
    添加 rtmp 配置如下
    ```nginx
    events {
        worker_connections  1024;
    }

    http {
        server {
            listen      80;
            server_name localhost;
            
            location / {
                root    html; # /usr/local/nginx/html
                index   index.html index.htm; # /usr/local/nginx/html/index.html
            }
        }
    }
    
    rtmp {
        server {
            listen 1935; # 监听的端口
            chunk_size 4096;

            application live { # rtmp 推流请求路径
                live on; # 开启 rtmp 直播
            }

            application vod {
                play /usr/share/nginx/html/vod; # 视频文件如 oceans.mp4 存放路径
            }
        }
    }
    ```
4. 启动 nginx  
    ~~`/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf`~~  
    `/usr/local/nginx/sbin/nginx`
5. <span id="streaming">推流</span>  
    ```bash
    ffmpeg -re -i oceans.mp4 -c copy -f flv rtmp://127.0.0.1:1935/live/oceans
    ```
6. 播放  
    ```bash
    ffplay rtmp://127.0.0.1:1935/live/oceans

    ffplay rtmp://127.0.0.1:1935/vod/oceans.mp4 # /usr/share/nginx/html/vod/oceans.mp4
    ```

- 搭建 hls 直播
    1. 修改 nginx 配置文件  
        `vi /usr/local/nginx/conf/nginx.conf`  
        配置 nginx.conf 如下
        ```nginx
        user root;

        events {
            worker_connections  1024;
        }

        http {
            server {
                listen      80;
                server_name localhost;
                
                location / {
                    root    html; # /usr/local/nginx/html
                    index   index.html index.htm; # /usr/local/nginx/html/index.html
                }

                location /hls {
                    types {
                        application/vnd.apple.mpegurl m3u8;
                        video/mp2t ts;
                    }

                    alias /usr/share/nginx/html/hls/; # "/usr/share/nginx/html/hls/" + "oceans.m3u8"

                    # expires -1;
                    add_header Cache-Control no-cache;

                    # valid_referers blocked 127.0.0.1 192.168.56.5; # 开启防盗链功能，只允许本站（192.168.56.5）网页上访问
                    # if ($invalid_referer) {
                    #     return 403;
                    # }
                }
            }
        }
        
        rtmp {
            server {
                listen 1935; # 监听的端口
                chunk_size 4096;

                application live { # rtmp 推流请求路径
                    live on; # 开启 rtmp 直播

                    hls on; # 开启 hls 直播
                    hls_path /usr/share/nginx/html/hls; # hls 的 m3u8 文件保存路径
                    hls_fragment 5s; # 切片长度

                    hls_cleanup off; # 关闭自动清理 m3u8、ts 文件功能，默认为开启 on
                }
            }
        }
        ```
    2. 创建 live.html  
        `vi /usr/local/nginx/html/live.html`  
        编辑 live.html 如下
        ```html
        <!DOCTYPE html>
        <html>

            <head>
                <meta charset="UTF-8">
                <title>live</title>
                <!-- PC 端不支持播放 m3u8 格式，需要引入 video.js -->
                <link href="https://unpkg.com/video.js/dist/video-js.css" rel="stylesheet">
                <script src="https://unpkg.com/video.js/dist/video.js"></script>
                <script src="https://unpkg.com/videojs-contrib-hls/dist/videojs-contrib-hls.js"></script>
            </head>

            <body>
                <video id="video" class="video-js vjs-default-skin fillWidth" controls width="800" height="480" data-setup='{}'>
                    <source id="source" type="application/x-mpegURL" />
                    <p>Your browser does not support HTML5 video.</p>
                </video>

                <script type="text/javascript">
                    var m = window.location.href.match(/#(.+)$/);
                    if(null != m) {
                        var name = m[1];

                        document.getElementById("source").src = "/hls/" + name + ".m3u8"
                        document.getElementById("video").load();
                        document.title = "Live - " + name;
                    }
                </script>
            </body>

        </html>
        ```
        > 在 html5 中播放（需开启 hls 直播）
        > ```html
        > <video>
        >     <source src="http://127.0.0.1:80/hls/oceans.m3u8"/>
        >     <p>Your browser does not support HTML5 video.</p>
        > </video>
        > ```
    3. 重启 nginx  
        `/usr/local/nginx/sbin/nginx -s reload`
    4. [推流](#streaming)
    5. 播放  
        `ffplay http://127.0.0.1:80/hls/oceans.m3u8`
    6. 访问 http://127.0.0.1/live.html#oceans

</details>