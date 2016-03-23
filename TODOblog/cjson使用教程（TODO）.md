#cjson使用教程

本文简单介绍cJSON后，说明读取json文件、解析json文件、生成json文件代码。

##json是什么？

- json 指的是 JavaScript 对象表示法（JavaScript Object Notation）
- json 是轻量级的文本数据交换格式
- json 独立于语言 
- json 具有自我描述性，更易理解

json 使用 JavaScript 语法来描述数据对象，但是 json 仍然独立于语言和平台。json 解析器和 json 库支持许多不同的编程语言。

##cjson是什么

JSON(JavaScriptObject Notation)是一种轻量级的数据交换格式。它基于JavaScript的一个子集。JSON采用完全独立于语言的文本格式，但是也使用了类似于C语言家族的习惯。这些特性使JSON成为理想的数据交换语言。易于人阅读和编写，同时也易于机器解析和生成。

cJSON是一个超轻巧，携带方便，单文件，简单的可以作为ANSI-C标准的JSON解析器。

##基本语法

json语法参照json语法,这里主要介绍cjson使用

http://blog.csdn.net/qq_15437667/article/details/50957996


1. cJSON存储的时候是采用链表存储的，其访问方式很像一颗树。每一个节点可以有兄妹节点，通过next/prev指针来查找，它类似双向链表；每个节点也可以有孩子节点，通过child指针来访问，进入下一层。不过，只有节点是对象或数组才可以有孩子节点。

cJSON基本数据结构：

```
typedefstruct cJSON {
	struct cJSON *next, *prev;
	struct cJSON *child;
	int type;
	char * valuestring;
	int valueint;
	double valuedouble;
	char *string;
}cJSON;
```

2、type一共有7种取值，分别是：

```
#define cJSON_False 0
#define cJSON_True 1
#define cJSON_NULL 2
#define cJSON_Number 3
#define cJSON_String 4
#define cJSON_Array 5
#define cJSON_Object 6
```

cJSON_NULL对应json中的null，cJSON_Number对应json中的整数或者浮点数，cJSON_String对应json中的字符串，cJSON_Array对应json中的数组，cJSON_Object对应json中的对象。

###API介绍

废话少说，不介绍API了，具体API会在代码解析中说明，这里直接进行实战。

##实战

> 以下介绍使用的json对象采用下面的json内容

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

###1.读取json文件

读取步骤比较直白，基本步骤是：1.读取文件二进制内容；2.将该数据使用cJSON_Parse()函数转换成cJSON格式

json数据文件名为'mme.json'，代码如下

```
static cJSON* read_json_file(char *filename, cJSON* json)
{
	long len = 0;
	int temp;
	char *JSON_content;
	
	FILE* fp = fopen(filename, "rb+");
    if(!fp)
    {
    	printf("open file %s failed.\n", filename);
        return NULL;
    }

	fseek(fp, 0, SEEK_END);
	len = ftell(fp);
	if(0 == len)
    {
        return NULL;
    }

	fseek(fp, 0, SEEK_SET);
    JSON_content = (char*) malloc(sizeof(char) * len);
    temp = fread(JSON_content, 1, len, fp);

	fclose(fp);
	json = cJSON_Parse(JSON_content);
	if (json == NULL)
	{
		return NULL;
	}
	free(JSON_content);
	
	return json;
}

int main()
{
	cJSON *json;

	json = read_json_file("mme.json", json);
	if (json == NULL)
	{
		printf("read_json_file json == NULL\n");
		return -1;
	}

	return 0;
}
```
read_json_file()函数解析

1. 先获取json文件数据长度，然后申请这么大的空间直接读取进来（前提是内容不是特别多）
2. 使用cJSON_Parse()函数进行将二进制数据转换成cJSON格式的数据
3. 返回

###2.解析json流

获取cJSON对象之后，就是按照层级解析。先来看一下本代码解析的json数据

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
那么前面读取json文件获得的对象就是这一个整个，不过是由多个对象层叠起来的。
那么比如我们这里要取得**{"virteNBName":"virt1", "virteNBNum":5, "begineNBID":0, "beginCtlPort":6000, "beginDataPort":7000, "virtIPNum":5}**这条信息中所有信息，那么流程如下：

1. 先取出"virtualeNB"对象，这样获得了一个数组，该数组包含两个对象
2. 对该数组进行操作，取出数组的第一个对象
3. 获取该对象的每个值

> 下面代码即按照该流程实现

```
static int handle_cJSON_data(cJSON* json)
{
	int virtENB_num;
	cJSON *virtENB_array_json;/* virtENB array Object */
	cJSON *virtENB_temp_json;/* virtENB temp Object */
	
	int index;
	int virteNB_num, begin_eNBID, begin_data_port, virtIP_num;
	int ret = 0;
	
	virtENB_array_json = cJSON_GetObjectItem(json, "virtualeNB");
	if (virtENB_array_json == NULL)
	{
		return -1;
	}

	virtENB_num = cJSON_GetArraySize(virtENB_array_json);
	index = 0;
	
	virtENB_temp_json = cJSON_GetArrayItem(virtENB_array_json, index);

	/* ugly code need todo */
	virteNB_num = get_intnum_from_JSONObject(cJSON_GetObjectItem(virtENB_temp_json, "virteNBNum"));
	begin_eNBID = get_intnum_from_JSONObject(cJSON_GetObjectItem(virtENB_temp_json, "begineNBID"));
	begin_data_port = get_intnum_from_JSONObject(cJSON_GetObjectItem(virtENB_temp_json, "beginDataPort"));
	virtIP_num = get_intnum_from_JSONObject(cJSON_GetObjectItem(virtENB_temp_json, "virtIPNum"));

	/*
	....特殊处理
	*/

	return ret;
}
```

代码对应

1. 先取出"virtualeNB"对象，这样获得了一个数组，该数组包含两个对象**virtENB_array_json = cJSON_GetObjectItem(json, "virtualeNB");**
2. 对该数组进行操作，取出数组的第一个对象**virtENB_temp_json = cJSON_GetArrayItem(virtENB_array_json, index);**
3. 获取该对象的每个值**virteNB_num = get_intnum_from_JSONObject(cJSON_GetObjectItem(virtENB_temp_json, "virteNBNum"));等代码即取出该对象的每一个值**

###3.输出json数据

输出json数据跟读取json文件步骤刚好相反，1.将该数据使用*cJSON_Print()函数转换成字符串格式；2.将字符串内容写入文件

代码日后补充。。。

###4.将自己的数据转换成cJSON对象

该部分自己暂时没用到，等我看完源码再补充