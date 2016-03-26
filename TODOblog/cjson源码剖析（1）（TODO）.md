#cjson源码剖析（1）

##cJSON类型

```
    /* cJSON Types: */
#define cJSON_False  (1 << 0)
#define cJSON_True   (1 << 1)
#define cJSON_NULL   (1 << 2)
#define cJSON_Number (1 << 3)
#define cJSON_String (1 << 4)
#define cJSON_Array  (1 << 5)
#define cJSON_Object (1 << 6)
    
#define cJSON_IsReference 256
#define cJSON_StringIsConst 512
```

前面七个是cJSON支持的七种类型

##cJSON结构

```
    /* The cJSON structure: */
    typedef struct cJSON {
        struct cJSON *next,*prev;	/* next/prev allow you to walk array/object chains. Alternatively, use GetArraySize/GetArrayItem/GetObjectItem */
        struct cJSON *child;		/* An array or object item will have a child pointer pointing to a chain of the items in the array/object. */
        
        int type;					/* The type of the item, as above. */
        
        char *valuestring;			/* The item's string, if type==cJSON_String */
        int valueint;				/* The item's number, if type==cJSON_Number */
        double valuedouble;			/* The item's number, if type==cJSON_Number */
        
        char *string;				/* The item's name string, if this item is the child of, or is in the list of subitems of an object. */
    } cJSON;
```

1. struct cJSON *next,*prev;用于指向同级对象
2. struct cJSON *child;如果一个对象是一个对象集合或者是对象数组，该指针指向存储对象集合或者是对象数组
3. int type;即 /* cJSON Types: */
4. char *valuestring;int valueint;double valuedouble;存储对应值
5. char *string;该值的对象名字。。

##cJSON接口

```
    typedef struct cJSON_Hooks {
        void *(*malloc_fn)(size_t sz);
        void (*free_fn)(void *ptr);
    } cJSON_Hooks;
    //释放/申请用
    
    /* Supply malloc, realloc and free functions to cJSON */
    extern void cJSON_InitHooks(cJSON_Hooks* hooks);
    //初始化cJSON_Hooks
    
    /* Supply a block of JSON, and this returns a cJSON object you can interrogate. Call cJSON_Delete when finished. */
    extern cJSON *cJSON_Parse(const char *value);
    //将json格式的字符串转换成cJSON对象
    
    /* Render a cJSON entity to text for transfer/storage. Free the char* when finished. */
    extern char  *cJSON_Print(cJSON *item);
    //将cJSON对象转换成json格式的字符串
    
    /* Render a cJSON entity to text for transfer/storage without any formatting. Free the char* when finished. */
    extern char  *cJSON_PrintUnformatted(cJSON *item);
    //输出无格式化字符串，不太懂
    
    /* Render a cJSON entity to text using a buffered strategy. prebuffer is a guess at the final size. guessing well reduces reallocation. fmt=0 gives unformatted, =1 gives formatted */
    extern char *cJSON_PrintBuffered(cJSON *item,int prebuffer,int fmt);
    //还是不懂。似乎是可以选择式，提前申请的char空间为prebuffer，然后可以减少realloc次数
    
    /* Delete a cJSON entity and all subentities. */
    extern void   cJSON_Delete(cJSON *c);
    //删除cJSON对象
    
    /* Returns the number of items in an array (or object). */
    extern int	  cJSON_GetArraySize(cJSON *array);
    //获得JSON数组对象个数
    
    /* Retrieve item number "item" from array "array". Returns NULL if unsuccessful. */
    extern cJSON *cJSON_GetArrayItem(cJSON *array,int item);
    //获得数组中第item 的对象
    
    /* Get item "string" from object. Case insensitive. */
    extern cJSON *cJSON_GetObjectItem(cJSON *object,const char *string);
    //取得名字为‘string’的该对象
    extern int cJSON_HasObjectItem(cJSON *object,const char *string);
    //看有没有名字为'string'的该对象
    
    /* For analysing failed parses. This returns a pointer to the parse error. You'll probably need to look a few chars back to make sense of it. Defined when cJSON_Parse() returns 0. 0 when cJSON_Parse() succeeds. */
    extern const char *cJSON_GetErrorPtr(void);
    //没看懂，好像是cJSON_Parse失败后用，用于查看是哪里格式不对，得看代码才知道
    
    /* These calls create a cJSON item of the appropriate type. */
    extern cJSON *cJSON_CreateNull(void);
    extern cJSON *cJSON_CreateTrue(void);
    extern cJSON *cJSON_CreateFalse(void);
    extern cJSON *cJSON_CreateBool(int b);
    extern cJSON *cJSON_CreateNumber(double num);
    extern cJSON *cJSON_CreateString(const char *string);
    extern cJSON *cJSON_CreateArray(void);
    extern cJSON *cJSON_CreateObject(void);
    //创建对象
    
    /* These utilities create an Array of count items. */
    extern cJSON *cJSON_CreateIntArray(const int *numbers,int count);
    extern cJSON *cJSON_CreateFloatArray(const float *numbers,int count);
    extern cJSON *cJSON_CreateDoubleArray(const double *numbers,int count);
    extern cJSON *cJSON_CreateStringArray(const char **strings,int count);
    //创建对象数组
    
    /* Append item to the specified array/object. */
    extern void cJSON_AddItemToArray(cJSON *array, cJSON *item);
    extern void	cJSON_AddItemToObject(cJSON *object,const char *string,cJSON *item);
    extern void	cJSON_AddItemToObjectCS(cJSON *object,const char *string,cJSON *item);
    //添加
    /* Use this when string is definitely const (i.e. a literal, or as good as), and will definitely survive the cJSON object */
    /* Append reference to item to the specified array/object. Use this when you want to add an existing cJSON to a new cJSON, but don't want to corrupt your existing cJSON. */
    extern void cJSON_AddItemReferenceToArray(cJSON *array, cJSON *item);
    extern void	cJSON_AddItemReferenceToObject(cJSON *object,const char *string,cJSON *item);
    
    
    /* Remove/Detatch items from Arrays/Objects. */
    extern cJSON *cJSON_DetachItemFromArray(cJSON *array,int which);
    extern void   cJSON_DeleteItemFromArray(cJSON *array,int which);
    extern cJSON *cJSON_DetachItemFromObject(cJSON *object,const char *string);
    extern void   cJSON_DeleteItemFromObject(cJSON *object,const char *string);
    //删除对象
    
    /* Update array items. */
    extern void cJSON_InsertItemInArray(cJSON *array,int which,cJSON *newitem);	/* Shifts pre-existing items to the right. */
    extern void cJSON_ReplaceItemInArray(cJSON *array,int which,cJSON *newitem);
    extern void cJSON_ReplaceItemInObject(cJSON *object,const char *string,cJSON *newitem);
    //替换对象
    
    /* Duplicate a cJSON item */
    extern cJSON *cJSON_Duplicate(cJSON *item,int recurse);
    /* Duplicate will create a new, identical cJSON item to the one you pass, in new memory that will
     need to be released. With recurse!=0, it will duplicate any children connected to the item.
     The item->next and ->prev pointers are always zero on return from Duplicate. */
    
    /* ParseWithOpts allows you to require (and check) that the JSON is null terminated, and to retrieve the pointer to the final byte parsed. */
    extern cJSON *cJSON_ParseWithOpts(const char *value,const char **return_parse_end,int require_null_terminated);
    
    extern void cJSON_Minify(char *json);
    
```