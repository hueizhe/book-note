在大型项目中如何使用Git子模块开发

# 写在前面

公司需要开发一个内部系统，要求每个部门都要接入。老板钦点，工期又压得短，于是浩浩汤汤的上百人就调过来了。

再简单的事情，只要人多起来就会变得复杂，开发的世界也是如此。

# 痛点

一个几百人的大项目，使用Git协作的时候，想一想我们的痛点：

项目过大，每个人clone等待时间过长
一会没有拉取代码就会发现有上百条更新待拉取
一人提交出错，上百人待机（笔者真实体验）
代码回溯困难，查找具体的修改记录无异于大海捞针
解决方案

这时候Git子模块就派上用场。

首先需要的当然是一个合理的架构，由于公司的保密原则这里就不贴项目了，本文主要描述在协作中子模块的用法。

项目结构

项目主体结构大概是这样的：
~~~
└── src
    ├── layout // 公共布局目录
    ├── public // 公共页面目录
    ├── router // 路由入口
    ├── components // 通用组件
    ├── modules    // 模块项目开发目录（子模块）
    |    ├── tms    // 子模块必须
    |    │    ├── components // 模块通用组件
    |    │    ├── pages // 模块页面
    |    │    ├── router // 模块路由
    |    │    └── store // 模块vuex数据
    |    └── ... // 各子模块
    ├── app.vue   // 跟组件
    └──  main.js   // 项目入口
~~~
一个部门一个子模块，每个子模块必须包含master（生产）、dev（开发）分支（推荐 gitflow 工作流）。

# 开发流程

## 克隆项目

如所有的webpack项目一样，src只是业务代码，开发配置都放在src外层，所以跑起开发环境首先需要克隆主项目。
~~~bash
$ git clone https://github.com/test/main.git
~~~
添加子模块

当然我们一般不用master分支做开发，正确的姿势是clone项目之后马上切换到基于dev的开发分支（原则上业务组不需要关注主项目的开发，主项目由架构组负责，但是为了保证代码的最新，并且将子模块添加合并进dev分支中，所以切换到主分支dev）。
~~~bash
$ git checkout -b dev origin/dev
~~~
这时候如果你的项目中已经有子模块了，你会发现modules文件夹下已经有了一个个子模块，但是显然现在这些模块都是空目录（预期的结果，我们不需要关注其他模块）。同时项目根目录下有一个.gitmodules文件，内容如下：
~~~bash
[submodule "modules/suba"]
    path = modules/suba
    url = https://github.com/test/suba.git
~~~
这里就是你的子模块关联文件，每添加一个子模块就会新增一条记录，如果是第一次添加Git子模块会自动生成。

开发环境有了，现在需要关联你的子模块：
~~~bash
$ git submodule add https://github.com/test/subb.git modules/subb
~~~
首次添加的子模块会拉取整个项目，打开muodules/subb文件夹，整个子模块项目都完好地列在那里，同时.gitmodules里新增了一条子模块记录modules/subb。

## 编辑子模块

同样的，我们也不应该在子模块的master分支上做任何编辑，这时候我们需要将子模块切换到基于dev的开发分支。

进入子模块目录下：
~~~bash
$ cd modules/subb/
$ git checkout -b feature/some-change origin/dev
~~~
当你在子模块做完一些修改一些修改之后，你想要把这这些修改推送到远程。
~~~bash
$ git commit -am 'test commit submodule'
$ git checkout dev
$ git merge feature/some-change
$ git push origin dev
$ git branch -d feature/some-change
~~~

这时候你的子模块的修改就已经提交到远程了，但是如果你进入到主项目的根目录查看差异，你会发现主项目中多了一些修改：
~~~bash
$ cd ../../
$ git diff
>   diff --git a/subb b/subb
    index 433859c..b78179a 160000
    --- a/subb
    +++ b/subb
    @@ -1 +1 @@
    -Subproject commit 433859c90e539d2a1b9fda27b32bef0d0acae9e6
    +Subproject commit b78179adab252a524ff2a41d6407a7daa6dad34f
~~~
这是因为你修改了子模块subb并提交了，但是主项目的指针依旧指向那个老的commit id，如果你不提交这个修改的话，别人拉取主项目并且使用git submodule update更新子模块还是会拉取到你修改前的代码。关于：Git 提交规范

所以这时候你需要把主项目也提交一遍：
~~~bash
$ git commit -am "test commit submodule"
$ git push origin dev
~~~
这样，你的修改就已经全部提交了。

## 新成员加入

当有新成员你加入你的子模块并且需要拉取代码的时候：
~~~bash
$ git clone https://github.com/test/main.git
$ git checkout -b dev origin/dev
$ git submodule init
$ git submodule update subb
~~~
git submodule update [submodule name]是指定拉取指定子模块的用法，如果你需要更新所有的子模块只需要使用git submodule update就可以了，一般我们在协作中只关注自己的模块。

那么接下来新成员也可以延续我们上面的开发流程了。

## 删除子模块

当然也有需求变动或者添加错误的时候，这时候就需要删除子模块了，值得吐槽的是git没有直接删除子模块的命令，所以只能逐步删除相关文件：

在版本控制中删除子模块：
~~~bash
$ git rm -r modules/subb
~~~
在编辑器中删除如下相关内容，也可以使用命令vi .gitmodules在vim中删除：
~~~bash
 [submodule "modules/subb"]
          path = modules/subb
          url = https://github.com/test/subb.git
          branch = dev
~~~
在编辑器中删除如下相关内容，也可以使用命令vi .git/config在vim中删除：
~~~bash
 [submodule "modules/subb"]
         url = https://github.com/test/subb.git
         active = true
~~~
删除.git下的缓存模块：
~~~bash
$ rm -rf .git/modules/subb
~~~
接下来提交修改：
~~~bash
$ git commit -am "delete subb"
$ git push origin dev
~~~

## 发布项目

当整个开发流程都结束了，终于到了发布的时刻，当然需要在主项目更新你的所有子模块：
~~~bash
$ git checkout master
$ git pull origin master
$ git submodule update
$ yarn build
~~~
这时候就可以发布你整个项目了，关于协作中使用子模块的操作就写到这里