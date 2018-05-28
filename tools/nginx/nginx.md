## nginx


### 1. linux版本的编译和安装

~~~bash
# yum -y install gcc gcc-c++ automake pcre pcre-devel zlib zlib-devel open openssl-devel
# mkdir -p /opt/nginx
# cp nginx-xx.tar.gz /opt/nginx
# tar xf nginx-xx.tar.gz
# ./configure --prefix=/nginx
# make && make install

~~~

### 2. nginx 服务的启停


