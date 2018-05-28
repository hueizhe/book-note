## nginx 服务 配置

## nginx.conf 
~~~
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

~~~

nginx.conf一共由三部分组成，分别为`全局块`，`events块`和`http块`。

在http块中，又包含http全局块，多个server块。

每个server块中，可以包含server全局块和多个location块

#### 1. 全局块

全局块是默认配置文件从开始到events块之间的一部分内容，主要设置一些影响nginx服务器整体运行的配置指令

通常包括配置运行nginx服务器的用户（组），允许生成的 worker process数，nginx进程pid存放路径，日志存放路径和类型 以及配置文件引入等。

#### 2.events 块

events块涉及的指令主要影响nginx服务器与用户的网络连接。

常用设置包括是否开启对worker process下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型处理连接请求，每个worker process 可以同时支持的最大连接数等。

#### 3.http块

代理，缓存和日志定义等绝大多数的功能和第三方模块的配置都可以放在这个模块中

可以在http全局块中配置的指令包括文件引入、MIME-Type定义。是否使用sendfile传输文件，连接超时时间、单连接请求数上限等。

#### 4.server块

server块和`虚拟主机`的概念有密切联系。

`虚拟主机`, 又称虚拟服务器、主机空间或是网页空间，它是一种技术。该技术是为了节省互联网服务器硬件成本而出现的。虚拟主机技术主要应用于http,FTP及EMAIL等多项服务，将一台服务器的某项或者全部服务内容逻辑划分为多个服务单位，对外表现为多个服务器，从而充分利用服务器硬件资源。从用户的角度看，一台虚拟主机和一台独立的硬件主机是完全一样的。

虚拟主机技术使得nginx服务可以在同一台服务器上只运行一组nginx进程，就可以运行多个网站。

每个server块就相当于一台虚拟主机

在server全局块中，最常见的两个配置是本虚拟机的监听配置 和 本虚拟机的名称或 ip 配置

#### 5. location 块

从严格意义上说，location 其实是server块的一个指令

地址定向，数据缓存和应答控制等功能都是 在这部分实现。














