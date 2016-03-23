#go源码阅读笔记（math.1）

##abs.go

**func Abs(x float64) float64**

```
package math

// Abs returns the absolute value of x.
//
// Special cases are:
//	Abs(±Inf) = +Inf
//	Abs(NaN) = NaN
func Abs(x float64) float64 {
	// TODO: once golang.org/issue/13095 is fixed, change this to:
	// return Float64frombits(Float64bits(x) &^ (1 << 63))
	// But for now, this generates better code and can also be inlined:
	if x < 0 {
		return -x
	}
	if x == 0 {
		return 0 // return correctly abs(-0)
	}
	return x
}
```

求一个数的绝对值Abs()函数
主要地方在于
`if x == 0 {
		return 0 // return correctly abs(-0)
	} `

这里是考虑当x=-0的场景，所以返回0

##bits.go

```
const (
	uvnan    = 0x7FF8000000000001
	uvinf    = 0x7FF0000000000000
	uvneginf = 0xFFF0000000000000
	mask     = 0x7FF
	shift    = 64 - 11 - 1
	bias     = 1023
)
```
数值表示遵循IEEE 754标准

- uvnan,NaN, Not a Number
- uvinf,正无穷大
- uvneginf,负无穷大

**函数func Inf(sign int) float64**

```
// Inf returns positive infinity if sign >= 0, negative infinity if sign < 0.
func Inf(sign int) float64 {
	var v uint64
	if sign >= 0 {
		v = uvinf
	} else {
		v = uvneginf
	}
	return Float64frombits(v)
}
```

如果sign是正数，就返回正无穷大，如果是负数就返回负无穷大

**func NaN() float64**

```
// NaN returns an IEEE 754 ``not-a-number'' value.
func NaN() float64 { return Float64frombits(uvnan) }
```

NaN()返回NaN值

**func IsNaN(f float64) (is bool)**

```
// IsNaN reports whether f is an IEEE 754 ``not-a-number'' value.
func IsNaN(f float64) (is bool) {
	// IEEE 754 says that only NaNs satisfy f != f.
	// To avoid the floating-point hardware, could use:
	//	x := Float64bits(f);
	//	return uint32(x>>shift)&mask == mask && x != uvinf && x != uvneginf
	return f != f
}
```
该函数判断一个数是否是NaN
如果不想使用浮点硬件，可以按照注释所说的这种方式计算

```
x := Float64bits(f);
return uint32(x>>shift)&mask == mask && x != uvinf && x != uvneginf
```
uint32(x>>shift)&mask == mask，x的高12位为0x7FF
x != uvinf，x不是正无穷大
x != uvneginf，x不是负无穷大

**func IsInf(f float64, sign int) bool **


```
// IsInf reports whether f is an infinity, according to sign.
// If sign > 0, IsInf reports whether f is positive infinity.
// If sign < 0, IsInf reports whether f is negative infinity.
// If sign == 0, IsInf reports whether f is either infinity.
func IsInf(f float64, sign int) bool {
	// Test for infinity by comparing against maximum float.
	// To avoid the floating-point hardware, could use:
	//	x := Float64bits(f);
	//	return sign >= 0 && x == uvinf || sign <= 0 && x == uvneginf;
	return sign >= 0 && f > MaxFloat64 || sign <= 0 && f < -MaxFloat64
}
```
判断一个数是不是无穷大数

同样，如果不想使用浮点硬件，可以按照注释所说的这种方式计算

**func normalize(x float64) (y float64, exp int)**

```
// normalize returns a normal number y and exponent exp
// satisfying x == y × 2**exp. It assumes x is finite and non-zero.
func normalize(x float64) (y float64, exp int) {
	const SmallestNormal = 2.2250738585072014e-308 // 2**-1022
	if Abs(x) < SmallestNormal {
		return x * (1 << 52), -52
	}
	return x, 0
}
```

`这个不懂什么意思`

> 遗留问题，MaxFloat64和-MaxFloat64是多少呢？

另一个

> Float64frombits(v)像是把一个数转化成float64，是如何实现的呢？

##一些数学函数……
func Acosh(x float64) float64
返回x的反双曲余弦值
func Asin(x float64) float64
func Acos(x float64) float64
func Asinh(x float64) float64 
func Atan(x float64) float64
func Atan2(y, x float64) float64
func Atanh(x float64) float64




