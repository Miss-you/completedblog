#python None与Null


是Python的特殊类型，Null对象是None Type，它只有一个值None.

1. 它不支持任何运算也没有任何内建方法.

2. None和任何其他的数据类型比较永远返回False。

3. None有自己的数据类型NoneType。

4. 你可以将None复制给任何变量，但是你不能创建其他NoneType对象。

```
feiqianyousadeMacBook-Pro:~ yousa$ python
Python 2.7.10 (default, Oct 23 2015, 18:05:06)
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> type(None)
<type 'NoneType'>
>>> None
>>> None == 0
False
>>> None == ''
False
>>> None == None
True
>>> None == False
False
>>>

```