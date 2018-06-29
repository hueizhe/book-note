、把最下面这段脚本加入.bashrc 最下面，保存退出然后重新登录

 

2、进入日常维护的目录，标记一个标签

Ruby代码   
cd /opt/logs/  
mark log  
 

3、以后再进入这个目录只需要

Ruby代码  
ggg log  
 

4、去掉这个标签

Ruby代码   
unmark log  
 

5、列出所有的标签

Ruby代码   
gs  
 

6、支持提示，键入ggg 不停的tab，就能提示当前所有的标签

 

原理： 在指定的目录下建立一堆软链接




 

代码：

Ruby代码   
~~~bash
# mark  
#书签保存的目录  
  
export MARKPATH=$HOME/.marks  
#设置你的默认书签，可以直接输入g跳转  
export MARKDEFAULT=api  
  
function ggg {  
        local m=$1  
        if [ "$m" = "" ]; then m=$MARKDEFAULT; fi  
        cd -P "$MARKPATH/$m" 2>/dev/null || echo "No such mark: $m"  
}  
function mark {  
        mkdir -p "$MARKPATH"  
        local m=$1  
        if [ "$m" = "" ]; then m=$MARKDEFAULT; fi  
        rm -f "$MARKPATH/$m"  
        ln -s "$(pwd)" "$MARKPATH/$m"  
}  
function unmark {  
        local m=$1  
        if [ "$m" = "" ]; then m=$MARKDEFAULT; fi  
        rm -i "$MARKPATH/$m"  
}  
function gs {  
       ＃我改了一下，原来用cut  
        ls -l "$MARKPATH" | grep ^l | awk '{print $9,$10,$11}'  
}  
_completemarks() {  
        local curw=${COMP_WORDS[COMP_CWORD]}  
        ＃我改了一下，原来用ls  
        local wordlist=$(find "$MARKPATH" -type l -printf "%f\n")  
        COMPREPLY=($(compgen -W '${wordlist[@]}' -- "$curw"))  
        return 0  
}  
complete -F _completemarks ggg unmark  
~~~