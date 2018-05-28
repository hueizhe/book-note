## nginx


### 1. linux版本的编译和安装

~~~bash
$ yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open openssl-devel
$ mkdir -p /opt/nginx
$ cp nginx-xx.tar.gz /opt/nginx
$ tar xf nginx-xx.tar.gz
$ ./configure --prefix=/nginx
$ make && make install

~~~

### 2. nginx 服务的启停
    
#### 获取pid

 - ps -ef | grep nginx

 - 在nginx服务启动后，在nginx安装目录下的logs目录中会产生文件名为nginx.pid的文件，此文件保存的就是nginx服务主进程的pid
    ~~~bash
        cat ./logs/nginx.pid
    ~~~

#### nginx 服务可接收的信号

| 信号       |   作用                                                    |
| ----       | --------                                                 |
|TERM 或INT  |快速停止nginx服务                                          |
|QUIT        |平缓停止nginx服务                                          |
|HUP         |使用新的配置文件启动进程，之后平缓停止原有进程，即平滑重启     |
|USR1        |重新打开日志文件，常用于日志切割                             |
|USR2        |使用新版本的Nginx文件启动服务，之后平缓停止原有进程，即平滑升级 |
|WINCH       |平缓停止worker process,用于nginx服务器平滑升级               |

#### nginx 的启动

`-t` 检查nginx配置文件是否有语法错误，可以与 `-c` 联用，使输出内容更详细
~~~bash
./nginx -t
~~~

使用默认配置文件，可以直接运行
~~~bash
./sbin/nginx 
~~~

#### nginx 的停止

- 快速停止
    立即停止当前nginx服务正在处理的请求
- 平缓停止
    允许nginx服务将正在处理的网络请求处理完成，但不再接收新的请求，之后关闭连接，停止工作

~~~bash
./sbin/nginx -g TERM | INT | QUIT
~~~
其中，TERM和INT 信号用于快速停止，QUIT用于平缓停止
或者
~~~bash
kill TERM | INT | QUIT `./logs/nginx.pid`
~~~




