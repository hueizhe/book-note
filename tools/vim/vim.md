vim
安装YouCompleteMe出现：YouCompleteMe unavailable no module named frozendict或者 YouCompleteMe unavailable no module named future

原因就是你或者没用Vundle安装，或者Vundle由于网速太慢下载到一半不能把安装依赖包完全下载下来

解决方案：

进入到YouCompleteMe目录，在terminal窗口敲入git submodule update --init --recursive

