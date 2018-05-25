## docker

## Docker快速入门
以下为在Docker Compose安装程序中安装和配置Istio的快速入门说明。

## 先决条件
[Docker](https://docs.docker.com/install/)
[Docker Compose](https://docs.docker.com/compose/install/)

## 安装步骤

1. 转至Istio发布页面下载与您的操作系统相对应的安装文件。如果您使用的是MacOS或Linux系统，则还可以运行以下命令自动下载并提取最新版本：
```bash
curl -L https://git.io/getLatestIstio | sh -
```

2. 提取安装文件并将目录更改为文件位置。安装目录包含：
- samples/中的示例应用程序
- 目录bin/中的istioctl客户端二进制文件。istioctl用于创建路由规则和策略。
- istio.VERSION配置文件

3. 将istioctl客户端添加到您的PATH。例如，在MacOS或Linux系统上运行以下命令：
```bash
export PATH=$PWD/bin:$PATH
```
4. 对于Linux用户，请配置DOCKER_GATEWAY环境变量
```bash
export DOCKER_GATEWAY=172.28.0.1:
```
5. 将目录更改为Istio安装目录的根目录。
6. 调出Istio控制面板容器：
```bash
docker-compose -f install/consul/istio.yaml up -d
```
7. 确认所有docker容器正在运行：
```bash
docker ps -a
```
如果Istio Pilot容器终​​止，请确保您运行istioctl context-create命令并重新运行上一步中的命令。

8. 配置istioctl为Istio API服务器使用映射的本地端口：
```bash
istioctl context-create --api-server http://localhost:8080
```

### 部署您的应用程序
您现在可以部署您自己的应用程序，或像[BookInfo](https://istio.io/docs/guides/bookinfo.html)一样安装提供的其中一个示例应用程序。

>注1：因为在Docker设置中没有任何pods理念，所以Istio sidecar运行在与应用程序相同的容器中。我们将使用注册者在Consul服务注册表中自动注册服务的实例。

>注2：应用程序必须使用HTTP / 1.1或HTTP / 2.0协议来处理所有HTTP通信，因为HTTP / 1.0不受支持。

```bash
docker-compose -f <your-app-spec>.yaml up -d
```

### 卸载
卸载Docker容器，卸载Istio核心组件：
```bash
docker-compose -f install/consul/istio.yaml down
```