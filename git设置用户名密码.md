#git设置用户名密码
设置git用户名／邮箱

```
git config --global user.name [username]
git config --global user.email [email]
```

使用`git config --list`查看已设配置

```
feiqianyousadeMacBook-Pro:xt_GTPU yousa$ git config --list
core.excludesfile=/Users/yousa/.gitignore_global
user.name=Miss-you
user.email=snowfly1993@gmail.com
core.repositoryformatversion=0
core.filemode=true
core.bare=false
core.logallrefupdates=true
core.ignorecase=true
core.precomposeunicode=true
remote.origin.url=https://github.com/Miss-you/xt_GTPU.git
remote.origin.fetch=+refs/heads/*:refs/remotes/origin/*
branch.master.remote=origin
branch.master.merge=refs/heads/master
```
