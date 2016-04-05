#go源码阅读笔记（math.4）

> 参考godoc API

##API列表

###func NaN() float64

函数返回一个IEEE 754“这不是一个数字”值。

###func IsNaN(f float64) (is bool)

判断f是否是NaN值

###func Inf(sign int) float64

如果sign>=0返回正无穷大，否则返回负无穷大

###func IsInf(f float64, sign int) bool

判断其是否是无穷大数

###func Float32bits(f float32) uint32

函数返回浮点数f的IEEE 754格式二进制表示的值对应的4字节无符号整数（**每位值不变**）。**主要是用于位运算之类的，转换成无符号整数，这样不会使用浮点运算器，速度快**

###func Float32frombits(b uint32) float32

将4字节无符号整数每位不变，转换成float32值，与Float32bits对应

###func Float64bits(f float64) uint64

与Float32bits类似

###func Float64frombits(b uint64) float64

与Float32frombits类似

###func Signbit(x float64) bool

```
func Signbit(x float64) bool {
	return Float64bits(x)&(1<<63) != 0
}
```

如果x是负数或者负0（**一种0的表示方式**），则返回true

###func Copysign(x, y float64) float64

```
// Copysign returns a value with the magnitude
// of x and the sign of y.
func Copysign(x, y float64) float64 {
	const sign = 1 << 63
	return Float64frombits(Float64bits(x)&^sign | Float64bits(y)&sign)
}
```
返回拥有x的量值（绝对值）和y的标志位（正负号）的浮点数。

###func Ceil(x float64) float64

返回不小于x的最小整数（的浮点值）

特殊值场景

```
Ceil(±0) = ±0
Ceil(±Inf) = ±Inf
Ceil(NaN) = NaN
```

###func Floor(x float64) float64

返回不大于x的最小整数（的浮点值）

特殊值场景

```
Floor(±0) = ±0
Floor(±Inf) = ±Inf
Floor(NaN) = NaN
```

###func Trunc(x float64) float64

返回x的整数部分（浮点数类型）

特殊值场景

```
Trunc(±0) = ±0
Trunc(±Inf) = ±Inf
Trunc(NaN) = NaN
```

###func Modf(f float64) (int float64, frac float64)

返回f的整数部分和小数部分，结果的正负号和都x相同，比如1.1返回1.0和0.1，-1.1返回-1.0和-0.1

特殊值场景

```
Modf(±Inf) = ±Inf, NaN
Modf(NaN) = NaN, NaN
```
> 今天太晚了，后面的下次再补。

###func Nextafter(x, y float64) (r float64)

参数x到参数y的方向上，下一个可表示的数值；如果x==y将返回x。

特殊值场景

```
Nextafter(NaN, y) = NaN
Nextafter(x, NaN) = NaN
```

###func Abs(x float64) float64

返回x的绝对值

###func Max(x, y float64) float64

返回x和y中大的。。

###func Min(x, y float64) float64

返回x和y中小的。。

###func Dim(x, y float64) float64

函数返回x-y和0中的较大者

特殊值场景

```
Dim(+Inf, +Inf) = NaN
Dim(-Inf, -Inf) = NaN
Dim(x, NaN) = Dim(NaN, x) = NaN
```

###func Mod(x, y float64) float64

取余运算，可以理解为 x-Trunc(x/y)*y，结果正负号跟x相同

###func Remainder(x, y float64) float64

IEEE 754差数求值，即x减去最接近x/y的整数值（如果有两个整数与x/y距离相同，则取其中的偶数）与y的乘积。

> 以下均为数学用函数……

###func Sqrt(x float64) float64

返回x的二次方根

###func Cbrt(x float64) float64

返回x的三次方根

###func Hypot(p, q float64) float64

返回Sqrt(p*p + q*q)

###func Sin(x float64) float64

求正弦

###func Cos(x float64) float64

求余弦

###func Tan(x float64) float64

求正切

###func Sincos(x float64) (sin, cos float64)

返回x的sin值和cos值

`sin, cos = math.Sincos(x)`

###func Asin(x float64) float64

求反正弦

###func Acos(x float64) float64

求反余弦

###func Atan(x float64) float64

求反正切

###func Atan2(y, x float64) float64

类似Atan(y/x)，但会根据x，y的正负号确定象限

###func Sinh(x float64) float64

双曲正弦

###func Cosh(x float64) float64

双曲余弦

###func Tanh(x float64) float64

双曲正切

###func Asinh(x float64) float64

反双曲正弦

###func Acosh(x float64) float64

反双曲余弦

###func Atanh(x float64) float64

反双曲正切

###func Log(x float64) float64

自然对数

###func Log1p(x float64) float64

等价于Log(1+x)，在x趋近于0时，准确度高

###func Log2(x float64) float64

2为底的对数

###func Log10(x float64) float64

10为底的对数

###func Logb(x float64) float64

返回x的二进制指数值，可以理解为Trunc(Log2(x))

###func Ilogb(x float64) int

类似Logb，但返回值是整型

###func Frexp(f float64) (frac float64, exp int)

返回一个标准化小数frac和2的整型指数exp，满足f == frac * 2**exp，且0.5 <= Abs(frac) < 1

###func Ldexp(frac float64, exp int) float64

Frexp的反函数，返回 frac * 2**exp

###func Exp(x float64) float64

返回E**x；x绝对值很大时可能会溢出为0或者+Inf，x绝对值很小时可能会下溢为1

###func Expm1(x float64) float64

等价于Exp(x)-1，但是在x接近零时更精确；x绝对值很大时可能会溢出为-1或+Inf

###func Exp2(x float64) float64

返回2**x

###func Pow(x, y float64) float64

返回x**y

###func Pow10(e int) float64

10**e

###func Gamma(x float64) float64

伽玛函数（当x为正整数时，值为(x-1)!）

###func Lgamma(x float64) (lgamma float64, sign int)

返回Gamma(x)的自然对数和正负号

###func Erf(x float64) float64

误差函数

###func Erfc(x float64) float64

余补误差函数

###func J0(x float64) float64

第一类贝塞尔函数，0阶

###func J1(x float64) float64

第一类贝塞尔函数，1阶

###func Jn(n int, x float64) float64

第一类贝塞尔函数，n阶

###func Y0(x float64) float64

第二类贝塞尔函数，0阶

###func Y1(x float64) float64

第二类贝塞尔函数，1阶

###func Yn(n int, x float64) float64

第二类贝塞尔函数，n阶