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