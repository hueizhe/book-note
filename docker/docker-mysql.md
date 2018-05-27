
## 创建Dockerfile

1. 首先，创建目录mysql,用于存放后面的相关东西。
~~~bash
$ mkdir -p ~/mysql/data ~/mysql/logs ~/mysql/conf


$ docker pull hub.c.163.com/library/mysql:5.7
~~~
2 重命名镜像名
~~~bash
docker tag hub.c.163.com/library/mysql:5.7 mysql:5.7
~~~
3.行容器
~~~bash
docker run -p 3306:3306   --name mysql \
            -v $PWD/conf/my.cnf:/etc/mysql/my.cnf  \
            -v $PWD/logs:/logs \
            -v $PWD/data:/mysql_data \
            -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
21cb89213c93d805c5bacf1028a0da7b5c5852761ba81327e6b99bb3ea89930e
~~~



## 使用Docker安装部署RabbitMQ

1. 拉取镜像
    ~~~bash
    $ docker search rabbitmq:management

    $ docker pull rabbitmq:management
    ~~~
    注意：如果docker pull rabbitmq 后面不带management，启动rabbitmq后是无法打开管理界面的，所以我们要下载带management插件的rabbitmq.
    
2. 开始创建rabbitmq容器
    ~~~bash
    docker run -d --name rabbitmq --publish 5671:5671 \ 
    --publish 5672:5672 --publish 4369:4369 --publish 25672:25672 --publish 15671:15671 --publish 15672:15672 \
    rabbitmq:management
    ~~~
3. 容器启动之后就可以访问web 管理端了

    http://宿主机IP:15672，默认创建了一个 guest 用户，密码也是 guest。