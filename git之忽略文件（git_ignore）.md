#git之忽略文件（git ignore）

> 每月总有那么几天，你想忽略git目录下的一些文件

##创建git ignore文件

在项目根目录与.git目录同一位置创建一个.gitignore文件

`这种方法是仅仅对该项目有效`

```
touch .gitignore
vim .gitignore
```

If you already have a file checked in, and you want to ignore it, Git will not ignore the file if you add a rule later. In those cases, you must untrack the file first, by running the following command in your terminal:

如果你想ignore一个已经在git目录下的文件A，你如果直接修改ignore文件是并不会导致Git忽略该文件A的，你必须要先在git中移除该文件A，假定该文件A目录为Project/mikuPlugin/out/**/*

```
git rm -r --cached  Project/mikuPlugin/out/**/*
git commit
```

当然你还需要先修改.gitignore文件

##git ignore简单语法

```

# 以'#'开始的行，被视为注释.                                                                                                                          
# 忽略*.o和*.a文件
 *.[oa]
# 忽略*.b和*.B文件，my.b除外
*.[bB]
!my.b
# 忽略dbg文件和dbg目录
dbg
# 只忽略dbg目录，不忽略dbg文件
dbg/
# 只忽略dbg文件，不忽略dbg目录
dbg
!dbg/
# 只忽略当前目录下的dbg文件和目录，子目录的dbg不在忽略范围内
/dbg
```