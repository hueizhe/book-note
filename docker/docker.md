

## Docker快速入门

- [Docker](https://docs.docker.com/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)

查看所有容器
~~~bash
docker ps -a
~~~
进入容器 其中字符串为容器ID:
~~~bash
docker exec -it d27bd3008ad9 /bin/bash
~~~
1.停用全部运行中的容器:
~~~bash
docker stop $(docker ps -q)
~~~
2.删除全部容器：
~~~bash
docker rm $(docker ps -aq)
~~~
3.一条命令实现停用并删除容器：
~~~bash
docker stop $(docker ps -q) & docker rm $(docker ps -aq)
~~~