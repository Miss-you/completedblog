#json简易语法

> json我觉得很多人用，所以就仅仅介绍一下简单的语法，以供理解

##json是什么？

- json 指的是 JavaScript 对象表示法（JavaScript Object Notation）
- json 是轻量级的文本数据交换格式
- json 独立于语言 
- json 具有自我描述性，更易理解

json 使用 JavaScript 语法来描述数据对象，但是 json 仍然独立于语言和平台。json 解析器和 json 库支持许多不同的编程语言。

##json语法规则

json语法简单来说就是四条：

- 数据在名称/值对中
- 数据由逗号分隔
- 花括号保存对象
- 方括号保存数组

> 声明：以下使用的对象均来自于以下内容

```
{
    "virtualeNB":[
        {"virteNBName":"virt1", "virteNBNum":5, "begineNBID":0, "beginCtlPort":6000, "beginDataPort":7000, "virtIPNum":5},
        {"virteNBName":"virt2", "virteNBNum":10, "begineNBID":10, "beginCtlPort":6000, "beginDataPort":7000, "virtIPNum":10}
    ],
    "eRAN":[
        {"eRANName":"eNB1", "eRANID":3002, "ctlPort":36412, "dataPort":2152},
        {"eRANName":"eNB2", "eRANID":10000, "ctlPort":36412, "dataPort":2152}
    ]
}
```

##json名称/值对

json数据的书写格式是：名称：值，这样的一对。即名称在前，该名称的值在冒号后面。例如：

`"virteNBName":"virt1"`

这里的名称是**"virteNBName"**，值是**"virt1"**，他们均是字符串

名称和值得类型可以有以下几种：

- 数字（整数或浮点数）
- 字符串（在双引号中）
- 逻辑值（true 或 false）
- 数组（在方括号中）
- 对象（在花括号中）
- null

##json数据由逗号分隔

譬如：

`"virteNBName":"virt1", "virteNBNum":5, "begineNBID":0`这几个对象之间就是使用逗号分隔。

数组内的对象之间当然也是要用逗号分隔。只要是对象之间，分隔就是用逗号`,`。但是，要注意，对象结束的时候，不要加逗号。数组内也是，例如：

```
	[
        {"eRANName":"eNB1", "eRANID":3002, "ctlPort":36412, "dataPort":2152},
        {"eRANName":"eNB2", "eRANID":10000, "ctlPort":36412, "dataPort":2152},
    ]
```

上面这个就是错误的，因为在数组中，两个对象之间需要逗号，但是到这个数组末尾了，不需要加逗号了。

##json花括号保存对象

对象可以包含多个名称/值对，如：

```
{"eRANName":"eNB1", "eRANID":3002, "ctlPort":36412, "dataPort":2152}
```

这一点也容易理解，与这条 JavaScript 语句等价：

```
"eRANName" = "eNB1"
"eRANID" = 3002
"ctlPort" = 36412
"dataPort" = 2152
```

##json方括号保存数组

数组可包含多个对象：

```
	"eRAN":[
        {"eRANName":"eNB1", "eRANID":3002, "ctlPort":36412, "dataPort":2152},
        {"eRANName":"eNB2", "eRANID":10000, "ctlPort":36412, "dataPort":2152}
    ]
```
在上面的例子中，对象 "eRAN" 是包含2个对象的数组。每个对象代表一条基站的记录。

##补充

json文件的文件类型是 ".json"
