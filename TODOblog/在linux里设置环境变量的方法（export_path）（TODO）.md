#在Linux里设置环境变量的方法（export PATH）

一般来说，配置交叉编译工具链的时候需要指定编译工具的路径，此时就需要设置环境变量。例如我的mips-linux-gcc编译器在“/opt/au1200_rm/build_tools/bin”目录下，build_tools就是我的编译工具，则有如下三种方法来设置环境变量：

 

1、直接用export命令：
#export PATH=$PATH:/opt/au1200_rm/build_tools/bin
查看是否已经设好，可用命令export查看：

 

[root@localhost bin]# export
declare -x BASH_ENV="/root/.bashrc"
declare -x G_BROKEN_FILENAMES="1"
declare -x HISTSIZE="1000"
declare -x HOME="/root"
declare -x HOSTNAME="localhost.localdomain"
declare -x INPUTRC="/etc/inputrc"
declare -x LANG="zh_CN.GB18030"
declare -x LANGUAGE="zh_CN.GB18030:zh_CN.GB2312:zh_CN"
declare -x LESSOPEN="|/usr/bin/lesspipe.sh %s"
declare -x LOGNAME="root"
declare -x LS_COLORS="no=00:fi=00:di=01;34:ln=01;36:pi=40;33:so=01;35:bd=40;33;01:cd=40;33;01:or=01;05;37;41:mi=01;05;37;41:ex=01;32:*.cmd=01;32:*.exe=01;32:*.com=01;32:*.btm=01;32:*.bat=01;32:*.sh=01;32:*.csh=01;32:*.tar=01;31:*.tgz=01;31:*.arj=01;31:*.taz=01;31:*.lzh=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.gz=01;31:*.bz2=01;31:*.bz=01;31:*.tz=01;31:*.rpm=01;31:*.cpio=01;31:*.jpg=01;35:*.gif=01;35:*.bmp=01;35:*.xbm=01;35:*.xpm=01;35:*.png=01;35:*.tif=01;35:"
declare -x MAIL="/var/spool/mail/root"
declare -x OLDPWD="/opt/au1200_rm/build_tools"
declare -x PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/X11R6/bin:/root/bin:/opt/au1200_rm/build_tools/bin"
declare -x PWD="/opt/au1200_rm/build_tools/bin"
declare -x SHELL="/bin/bash"
declare -x SHLVL="1"
declare -x SSH_ASKPASS="/usr/libexec/openssh/gnome-ssh-askpass"
declare -x SSH_AUTH_SOCK="/tmp/ssh-XX3LKWhz/agent.4242"
declare -x SSH_CLIENT="10.3.37.152 2236 22"
declare -x SSH_CONNECTION="10.3.37.152 2236 10.3.37.186 22"
declare -x SSH_TTY="/dev/pts/2"
declare -x TERM="linux"
declare -x USER="root"
declare -x USERNAME="root"
可以看到，环境变量已经设好，PATH里面已经有了我要加的编译器的路径。

 

2、修改profile文件： 
#vi /etc/profile 
在里面加入:
export PATH="$PATH:/opt/au1200_rm/build_tools/bin"

3. 修改.bashrc文件：
# vi /root/.bashrc
在里面加入：
export PATH="$PATH:/opt/au1200_rm/build_tools/bin"

后两种方法一般需要重新注销系统才能生效，最后可以通过echo命令测试一下：
# echo $PATH
看看输出里面是不是已经有了/my_new_path这个路径了。