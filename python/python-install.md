## 从安装python3.5.2开始


1. 操作环境：CentOS 7
2. 编译环境的安装：
~~~
yum install zlib-devel bzip2-devel openssl-devel ncurese-devel（要先安装这个不然python安装不完整，会没有pip3)
~~~
3. 进入下载目录，并解压，进入解压后的目录
~~~ 
 cd ~/下载 wget https://www.python.org/ftp/python/3.5.2/Python-3.5.2.tgz
 tar -xf Python-3.5.2.tgz 
 cd ~/下载/Python-3.5.2
~~~ 
4. 设置安装目录：
~~~
./configure --prefix=/usr/bin/python-3.5.2
~~~
5. 新建安装的文件夹：
~~~
sudo mkdir /usr/bin/python-3.5.2
~~~
6. 编译命令 ：
~~~
make（等待..)
~~~
7. 安装命令：
~~~
sudo make install
~~~
8. 修改默认的python版本：
~~~
sudo mv /usr/bin/python  /usr/bin/python.bak
sudo ln -s /usr/bin/python-3.5.2/bin/python3.5 /usr/bin/python
~~~
9. 查看默认版本：python -V（如果是3.5.2，则修改完成）
10. 更改yum配置：
~~~
sudo vi /usr/bin/yum（更改头部的python为python.bak）这样yum又可以使用python2
~~~
11. pip3使用前配置：
~~~ 
ln -s /usr/bin/python-3.5.2/bin/pip3 /usr/bin/pip3
~~~
输入,查看是否可以运行
~~~
pip3 list，
~~~

pip3升级：
~~~
pip3 install --upgrade pip
~~~