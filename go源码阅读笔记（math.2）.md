#go源码阅读笔记（math.2）


##浮点数与整形数转换math/unsafe.go

在阅读math代码的时候，发现Float64bits以及Float64frombits使用非常多，先查看一下这两个函数是做什么用的

```
package math

import "unsafe"

// Float32bits returns the IEEE 754 binary representation of f.
func Float32bits(f float32) uint32 { return *(*uint32)(unsafe.Pointer(&f)) }

// Float32frombits returns the floating point number corresponding
// to the IEEE 754 binary representation b.
func Float32frombits(b uint32) float32 { return *(*float32)(unsafe.Pointer(&b)) }

// Float64bits returns the IEEE 754 binary representation of f.
func Float64bits(f float64) uint64 { return *(*uint64)(unsafe.Pointer(&f)) }

// Float64frombits returns the floating point number corresponding
// the IEEE 754 binary representation b.
func Float64frombits(b uint64) float64 { return *(*float64)(unsafe.Pointer(&b)) }

```

解释：

- (*float64)(unsafe.Pointer(&b))是go提供的一种标准的写法，允许将一种指针转换成另一种指针。
- *(*float64)(unsafe.Pointer(&b))故该操作就是把uint64型的b转换成float64
- func Float64frombits(b uint64) float64，将uint64数转换成float64
- func Float64bits(f float64) uint64，将float64数转换成uint64

**应该是因为在go语言中，如果对float进行位运算，会使用浮点计算单元，但是把它转换成整型（转换过程中速度很快，并不使用浮点计算单元），就可以直接使用整型计算单元，相较于浮点计算单元，整型的计算会快很多，所以在API中会这么写**

##const.go

该代码主要是记录了一些数学常量，譬如e、pi，还有一些常用的类型的最大最小值，譬如MaxInt32、MinInt32

```
// Package math provides basic constants and mathematical functions.
package math

// Mathematical constants.
const (
	E   = 2.71828182845904523536028747135266249775724709369995957496696763 // http://oeis.org/A001113
	Pi  = 3.14159265358979323846264338327950288419716939937510582097494459 // http://oeis.org/A000796
	Phi = 1.61803398874989484820458683436563811772030917980576286213544862 // http://oeis.org/A001622

	Sqrt2   = 1.41421356237309504880168872420969807856967187537694807317667974 // http://oeis.org/A002193
	SqrtE   = 1.64872127070012814684865078781416357165377610071014801157507931 // http://oeis.org/A019774
	SqrtPi  = 1.77245385090551602729816748334114518279754945612238712821380779 // http://oeis.org/A002161
	SqrtPhi = 1.27201964951406896425242246173749149171560804184009624861664038 // http://oeis.org/A139339

	Ln2    = 0.693147180559945309417232121458176568075500134360255254120680009 // http://oeis.org/A002162
	Log2E  = 1 / Ln2
	Ln10   = 2.30258509299404568401799145468436420760110148862877297603332790 // http://oeis.org/A002392
	Log10E = 1 / Ln10
)

// Floating-point limit values.
// Max is the largest finite value representable by the type.
// SmallestNonzero is the smallest positive, non-zero value representable by the type.
const (
	MaxFloat32             = 3.40282346638528859811704183484516925440e+38  // 2**127 * (2**24 - 1) / 2**23
	SmallestNonzeroFloat32 = 1.401298464324817070923729583289916131280e-45 // 1 / 2**(127 - 1 + 23)

	MaxFloat64             = 1.797693134862315708145274237317043567981e+308 // 2**1023 * (2**53 - 1) / 2**52
	SmallestNonzeroFloat64 = 4.940656458412465441765687928682213723651e-324 // 1 / 2**(1023 - 1 + 52)
)

// Integer limit values.
const (
	MaxInt8   = 1<<7 - 1
	MinInt8   = -1 << 7
	MaxInt16  = 1<<15 - 1
	MinInt16  = -1 << 15
	MaxInt32  = 1<<31 - 1
	MinInt32  = -1 << 31
	MaxInt64  = 1<<63 - 1
	MinInt64  = -1 << 63
	MaxUint8  = 1<<8 - 1
	MaxUint16 = 1<<16 - 1
	MaxUint32 = 1<<32 - 1
	MaxUint64 = 1<<64 - 1
)
```

##copysign.go

以第二个参数y的符号（正或负）返回第一个参数x

```
// Copysign returns a value with the magnitude
// of x and the sign of y.
func Copysign(x, y float64) float64 {
	const sign = 1 << 63
	return Float64frombits(Float64bits(x)&^sign | Float64bits(y)&sign)
}
```
Float64bits(x)&^sign是把x的符号位置为0

Float64bits(y)&sign是取得y的符号位
