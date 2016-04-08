#DBL_EPSILON和 FLT_EPSILON的用法


DBL_EPSILON和 FLT_EPSILON主要用于单精度和双精度的比较当中：

![image](http://img.blog.csdn.net/20140225170017421)

比较方式

```
double b = sin(M_PI / 6.0);
if (fabs(((double)valueint)-value)<=DBL_EPSILON)
	(is int num);
else
	(is double num)
```

EPSILON是最小误差。如果整数值减去浮点数值误差低于DBL_EPSILON，则说明该数可以近似看成整数，否则则是浮点数……

