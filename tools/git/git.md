## git 

- 从Git远程仓库中将项目复制到本地，使用`git clone`命令（该命令类似于SVN中的`checkout`，但在Git中`checkout`命令用于切换本地分支）。

    举例：`git clone git@github.com:drogba321/alibaba-learning-docs.git`
- 新建一个项目以后，首先要使用`git init`命令进行git初始化（创建.git隐藏文件夹，表示这是一个git项目）。然后使用`git remote add`命令创建远程仓库。

    举例：`git remote add origin git@gitlab.alibaba-inc.com:trip/h5-test.git`（该命令的意思是，创建一个名为origin的远程仓库，其git地址为`git@gitlab.alibaba-inc.com:trip/h5-test.git`）。
- 新增文件、修改文件和删除文件后，要首先将这些变化添加到版本控制中，使用`git add`命令（类似于SVN中的`add`）。

    举例：`git add . `（圆点符“.”代表将当前目录下的所有内容都添加到版本控制中）。
- Git add之后，要完成提交，第一步需要将代码提交到本地仓库，使用`git commit`命令。

    举例：`git commit -m 'this is the first commit'`（引号中的内容为该版本的提交信息）。
- 第二步是将本地仓库中的代码“推送”到远程仓库，使用`git push`命令，
    
    举例：`git push origin daily/0.1.1:daily/0.1.1`（origin为远程仓库的名称，第一个daily/0.1.1是本地分支的名字，第二个daily/0.1.1是远程分支的名字。该命令的意思是，将本地分支daily/0.1.1的代码推送到远程仓库origin的分支daily/0.1.1下）。
- 将远程仓库中的代码更新到本地，使用`git pull`命令（类似于SVN中的`update`）。

    举例：`git pull origin daily/0.1.1`（该命令的意思是，将远程仓库origin的内容更新到本地分支daily/0.1.1）；或者`git pull origin master:daily/0.1.1`（该命令的意思是，将远程仓库origin的远程分支master更新到本地分支daily/0.1.1。该命令可能会被reject，原因有待研究）。
- 切换本地分支，使用`git checkout`命令。

    举例：`git checkout master`（该命令的意思是，将本地分支切换到master分支）。
- 查看当前状态，包括当前本地所处的分支，有哪些修改等等，
    
    使用`git status`命令。
- 查看本地与远程分支的列表，使用`git branch`命令。如果只想查看远程分支的列表，使用`git branch -r`命令。如果要新建分支，使用`git branch [newBranchName]`命令（其中newBranchName为新分支名）


- 如果已经对工作目录做了一些修改，但这些修改还未提交，此时想撤销这些修改的话，可以使用reset命令,该命令会把工作目录中所有未提交的内容清空，使其恢复到上次提交时的状态。使用示例：

		$ git reset --hard HEAD

- 如果已经做了一个提交（commit），但此时后悔了，想撤销此次提交，可以使用revert命令，该命令会撤销掉上一次（或更早之前的）提交。使用示例：

		$ git revert HEAD

- 要想创建一个新的本地分支（新分支名假定是daily/2.0.1），可以使用如下命令：
		
		$ git checkout -b daily/2.0.1
    该命令相当于执行以下这两条命令：
    ~~~bash
		$ git branch daily/2.0.1 //创建新分支
		$ git checkout daily/2.0.1 //切换到新分支
    ~~~
- 分支的合并。例如在分支daily/2.0.1上完成了修改，希望将修改的内容也应用在master分支上，可以将daily/2.0.1分支合并到master分支上。首先切换到master分支，然后将daily/2.0.1分支并入当前分支（即master分支），如下示例：

		$ git checkout master //切换到master分支
		$ git merge daily/2.0.1 //将daily/2.0.1并入当前分支（即master分支）

- 更多参考资料，可以参见[这里](http://gitbook.liuhui998.com/4_9.html)和[这里](http://git-scm.com/book/zh/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)



### Git与SVN的比较 

- SVN需要有一个主干（trunk），而若干个分支（branch）只能合并到主干，分支之间不能直接合并。相当于必须要以主干作为中转，才能实现分支的合并。
- Git中没有主干的概念，只有分支的概念。其中一个名为master的分支作为主分支，类似于SVN中的主干。但是master本质上也只是一个分支，与其他分支无异。Git可以支持分支之间直接合并，比较灵活，适用于互联网前端开发这种需求变化较大的应用场景。
- SVN的提交操作（commit）是直接将本地代码上传到远程仓库中。
- Git的提交操作是先将本地代码提交到本地仓库（通过git commit命令），然后再将本地仓库中的代码“推送”到远程仓库中（通过Git的push命令）。
- SVN中的版本控制是以“文件”为单元的，项目中的每一个文件都有自己的版本历史。
- Git中的版本控制是以整个项目为单元的，整个项目作为一个整体，有一个统一的版本历史。
- SVN远程仓库中存储的是各个版本的全量拷贝。
- Git远程仓库中存储的是各个版本之间的增量变化（实际上是每个版本的“快照”），因此所需的存储空间较小。