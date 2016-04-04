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


func Abs(x float64) float64
func Max(x, y float64) float64
func Min(x, y float64) float64
func Dim(x, y float64) float64
func Mod(x, y float64) float64
func Remainder(x, y float64) float64
func Sqrt(x float64) float64
func Cbrt(x float64) float64
func Hypot(p, q float64) float64
func Sin(x float64) float64
func Cos(x float64) float64
func Tan(x float64) float64
func Sincos(x float64) (sin, cos float64)
func Asin(x float64) float64
func Acos(x float64) float64
func Atan(x float64) float64
func Atan2(y, x float64) float64
func Sinh(x float64) float64
func Cosh(x float64) float64
func Tanh(x float64) float64
func Asinh(x float64) float64
func Acosh(x float64) float64
func Atanh(x float64) float64
func Log(x float64) float64
func Log1p(x float64) float64
func Log2(x float64) float64
func Log10(x float64) float64
func Logb(x float64) float64
func Ilogb(x float64) int
func Frexp(f float64) (frac float64, exp int)
func Ldexp(frac float64, exp int) float64
func Exp(x float64) float64
func Expm1(x float64) float64
func Exp2(x float64) float64
func Pow(x, y float64) float64
func Pow10(e int) float64
func Gamma(x float64) float64
func Lgamma(x float64) (lgamma float64, sign int)
func Erf(x float64) float64
func Erfc(x float64) float64
func J0(x float64) float64
func J1(x float64) float64
func Jn(n int, x float64) float64
func Y0(x float64) float64
func Y1(x float64) float64
func Yn(n int, x float64) float64