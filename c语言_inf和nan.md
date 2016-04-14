#C语言 inf和nan 

     头文件：include<math.h>
     宏的用法（类似于函数原型）：
```     
int fpclassify(x);
                                 int isfinite(x);
                                 int isnormal(x);
                                 int isnan(x);
                                 int isinf(x);
```

     具体用法：
          1、int fpclassify(x)  用来查看浮点数x的情况，fpclassify可以用任何浮点数表达式作为参数，fpclassify的返回值有以下几种情况。
                  FP_NAN：x是一个“not a number”。
                  FP_INFINITE: x是正、负无穷。
                  FP_ZERO: x是0。
                  FP_SUBNORMAL: x太小，以至于不能用浮点数的规格化形式表示。
                  FP_NORMAL: x是一个正常的浮点数（不是以上结果中的任何一种）。
          2、int isfinite(x)  当（fpclassify(x)!=FP_NAN&&fpclassify(x)!=FP_INFINITE）时，此宏得到一个非零值。
          3、int isnormal(x)  当（fpclassify(x)==FP_NORMAL）时，此宏得到一个非零值。
          4、int isnan(x)   当（fpclassify(x)==FP_NAN）时，此宏返回一个非零值。
          5、int isinf(x)   当x是正无穷是返回1，当x是负无穷时返回-1。（有些较早的编译器版本中，无论是正无穷还是负无穷，都返回非零值，不区分正负无穷）。