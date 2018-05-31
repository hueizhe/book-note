## git 高级用法

## git process
![git](https://github.com/hueizhe/book-note/blob/master/tools/image/git.png)


## git alias
~~~bash
git config --global alias.co checkout

git config  --global alias.ci commit

git config  --global alias.br branch

git config  --global alias.st status
~~~

## git log
You can config your git log command with pretty format.
~~~bash
git config --global alias.lgs "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
~~~
~~~bash
git config --global alias.lg "log --graph --color --pretty=format:' %Cred%h %Creset/ %<(10,trunc)%Cblue%an%Creset | %<(60,trunc)%s | %cr %Cred%d' --remotes --branches"
~~~
