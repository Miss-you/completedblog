#使用python pep8常见问题记录


##import

**不要在一句import中引用多个库**

譬如

```
import os, sys
```
这样写不好，最好这么写

```
import os
import sys
```

##代码长度约束

1. 一行列数：PEP8 规定最大为79列，如果拼接url很容易超限
2. 一个函数：不可以超过30行；直观来讲就是完整显示一个函数一个屏幕就够了，不需要上下拖动
3. 一个类：不要超过200行代码，不要超过10个方法
4. 一个模块：不要超过500行

##一些其他格式问题

###1、W292 no newline at end of file

处理：在代码末尾加一行回车就行

###2、E302 expected 2 blank lines，found 1

处理：需要再补一个空白行（函数之间需要最少2个空白行，方便查阅、区分）

###3、E231 missing whitespace after ','

处理：原因简单来说还是要方便查看，即逗号后“，”需要补空格

示例：

```
print("%s %s %s %s" %(A,B,C,D))
print("%s %s %s %s" % (A, B, C, D))
```

也许在这里并不能非常明显看出来，但是当代码多的时候，你会发现适当的空格会显得代码容易观看~

###4、E225 missing whitespace around operator

处理：主要原因其实跟上面的问题三差不多，主要目的都是为了查看方便

###5、W291 trailing whitespace

处理：字面意思，函数、或者代码段终止处出现了多余的空格

举例

```
return kw   （这里多了几个空格，错误）
return kw
```




