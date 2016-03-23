#go源码阅读笔记（math.3）

##dim.go

```
package math

// Dim returns the maximum of x-y or 0.
//
// Special cases are:
//	Dim(+Inf, +Inf) = NaN
//	Dim(-Inf, -Inf) = NaN
//	Dim(x, NaN) = Dim(NaN, x) = NaN
func Dim(x, y float64) float64

func dim(x, y float64) float64 {
	return max(x-y, 0)
}
```

func dim(x, y float64) float64 ，返回x-y与0的较大者

这里我们可以看出，调用的函数max()进行了各种异常判断，所以在dim函数这里就不需要各种异常判断了，这里可以作为一点经验

```
// Max returns the larger of x or y.
//
// Special cases are:
//	Max(x, +Inf) = Max(+Inf, x) = +Inf
//	Max(x, NaN) = Max(NaN, x) = NaN
//	Max(+0, ±0) = Max(±0, +0) = +0
//	Max(-0, -0) = -0
func Max(x, y float64) float64

func max(x, y float64) float64 {
	// special cases
	switch {
	case IsInf(x, 1) || IsInf(y, 1):
		return Inf(1)
	case IsNaN(x) || IsNaN(y):
		return NaN()
	case x == 0 && x == y:
		if Signbit(x) {
			return y
		}
		return x
	}
	if x > y {
		return x
	}
	return y
}
```
这里看不太懂为什么要判断`case x == 0 && x == y:`，可能我需要看看Signbit()这个函数是做什么用的吧。

```
// Min returns the smaller of x or y.
//
// Special cases are:
//	Min(x, -Inf) = Min(-Inf, x) = -Inf
//	Min(x, NaN) = Min(NaN, x) = NaN
//	Min(-0, ±0) = Min(±0, -0) = -0
func Min(x, y float64) float64

func min(x, y float64) float64 {
	// special cases
	switch {
	case IsInf(x, -1) || IsInf(y, -1):
		return Inf(-1)
	case IsNaN(x) || IsNaN(y):
		return NaN()
	case x == 0 && x == y:
		if Signbit(x) {
			return x
		}
		return y
	}
	if x < y {
		return x
	}
	return y
}

```

这个同max

###遗留问题

什么要判断`case x == 0 && x == y:`，我需要看看Signbit()这个函数是做什么用？

##floor.go

floor.go主要求的是一个数的上界或者下界

```
package math

// Floor returns the greatest integer value less than or equal to x.
//
// Special cases are:
//	Floor(±0) = ±0
//	Floor(±Inf) = ±Inf
//	Floor(NaN) = NaN
func Floor(x float64) float64

func floor(x float64) float64 {
	if x == 0 || IsNaN(x) || IsInf(x, 0) {
		return x
	}
	if x < 0 {
		d, fract := Modf(-x)
		if fract != 0.0 {
			d = d + 1
		}
		return -d
	}
	d, _ := Modf(x)
	return d
}
```
func floor(x float64) float64，返回小于等于x的最大整数

> Modf()这个函数做什么用？先查看一下该函数

```
// Modf returns integer and fractional floating-point numbers
// that sum to f.  Both values have the same sign as f.
//
// Special cases are:
//	Modf(±Inf) = ±Inf, NaN
//	Modf(NaN) = NaN, NaN
func Modf(f float64) (int float64, frac float64)

func modf(f float64) (int float64, frac float64) {
	if f < 1 {
		switch {
		case f < 0:
			int, frac = Modf(-f)
			return -int, -frac
		case f == 0:
			return f, f // Return -0, -0 when f == -0
		}
		return 0, f
	}

	x := Float64bits(f)
	e := uint(x>>shift)&mask - bias

	// Keep the top 12+e bits, the integer part; clear the rest.
	if e < 64-12 {
		x &^= 1<<(64-12-e) - 1
	}
	int = Float64frombits(x)
	frac = f - int
	return
}
```

但是，，，我看了半天还是没看明白，我仔细研究一下，懂了之后再解释

> 就算这样，通过上面代码可以看出来，Modf()是传入一个数，然后返回这个数的整数部分和小数部分，譬如1.5返回1.0和0.5，-1.5返回-1.0和-0.5



```
// Ceil returns the least integer value greater than or equal to x.
//
// Special cases are:
//	Ceil(±0) = ±0
//	Ceil(±Inf) = ±Inf
//	Ceil(NaN) = NaN
func Ceil(x float64) float64

func ceil(x float64) float64 {
	return -Floor(-x)
}
```
func ceil(x float64) float64，写的很妙，可以参考，大概就是-x的下界其实就是x上界的相反数

感觉math这章基本是数学技巧

```
// Trunc returns the integer value of x.
//
// Special cases are:
//	Trunc(±0) = ±0
//	Trunc(±Inf) = ±Inf
//	Trunc(NaN) = NaN
func Trunc(x float64) float64

func trunc(x float64) float64 {
	if x == 0 || IsNaN(x) || IsInf(x, 0) {
		return x
	}
	d, _ := Modf(x)
	return d
}
```
func trunc(x float64) float64，返回的就是f的整数部分