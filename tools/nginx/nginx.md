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
    
 获取pid
    - ps -ef | grep nginx
    - 在nginx服务启动后，在nginx安装目录下的logs目录中会产生文件名为nginx.pid的文件，此文件保存的就是nginx服务主进程的pid
    ~~~bash
        cat ./logs/nginx.pid
    ~~~

nginx 服务可接收的信号

| 信号       |   作用                                                    |
| ----       | --------                                                 |
|TERM 或INT  |快速停止nginx服务                                          |
|QUIT        |平缓停止nginx服务                                          |
|HUP         |使用新的配置文件启动进程，之后平缓停止原有进程，即平滑重启     |
|USR1        |重新打开日志文件，常用于日志切割                             |
|USR2        |使用新版本的Nginx文件启动服务，之后平缓停止原有进程，即平滑升级 |
|WINCH       |平缓停止worker process,用于nginx服务器平滑升级               |

