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
其中，TERM和INT 信号用于快速停止，QUIT用于平缓停止, 或者
~~~bash
kill TERM | INT | QUIT `./logs/nginx.pid`
~~~

#### nginx 的重启

平滑重启： nginx进程接收到信号后，首先读取新的nginx配置文件，如果配置语法正确，则启动新的nginx服务，然后平缓关闭旧的服务进程，如果新的nginx配置有问题，将显示错误，仍然使用旧的nginx进程提供服务

~~~bash
./sbin/nginx -g HUP {-c newConfFile}
~~~
HUP信号用于发送平滑重启信号， newConfFile, 可选项，用于指定新配置文件的路径

或者，使用新的配置文件代替了旧的配置文件后，使用：
~~~bash
kill HUP `./logs/nginx.pid`
~~~
也可以实现平滑重启

#### nginx 服务器的升级

平滑升级：nginx服务接收到`USR2`信号后，首先将旧的 `nginx.pid`文件添加后缀`.oldbin`,变为 `nginx.pid.oldbin`; 然后执行新版本nginx服务器的二进制文件启动服务。 如果新的服务启动成功，系统中将有新旧两个nginx服务共同提供web服务。之后，需要向旧的nginx服务发送 `WINCH` 信号，使旧的nginx服务平滑停止，并删除 `niginx.pid.oldbin`文件。在发送`WINCH`信号之前，可以随时停止新的nginx服务。

使用以下命令将旧服务器的安装路径更改为新服务器的安装路径
~~~bash
$ ./nginx -p newInstallPath
~~~
其中，newInstallPath为新服务器的安装路径。之后备份旧服务器，安装新服务器即可。然后，使用以下命令实现nginx服务的平滑升级：
~~~bash
$ ./sbin/nginx -g USR2
~~~
其中，USR2 信号用于发送平滑升级信号，或者使用：
~~~bash
$ kill USR2 `/nginx/logs/nginx.pid`
~~~
通过 `ps -ef | grep nginx` 查看新的nginx服务启动正常，再使用
~~~bash
$ ./sbin/nginx -g WINCH
~~~
或者，使用
~~~bash
$ kill WINCH `/nginx/logs/nginx.pid`
~~~
这样就在不停止提供web服务的前提下完成了nginx服务的平滑升级