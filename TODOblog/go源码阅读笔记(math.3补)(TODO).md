#go源码阅读笔记(math.3补)
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


分段分析

```
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
```
当f=0的时候，（还是float类型，一定要考虑0这种情况）；如果f<0，那么其实就相当于求-f的值即可

```
	x := Float64bits(f)
	e := uint(x>>shift)&mask - bias

	// Keep the top 12+e bits, the integer part; clear the rest.
	if e < 64-12 {
		x &^= 1<<(64-12-e) - 1
	}
	int = Float64frombits(x)
	frac = f - int
	return
```

bias是多少？

```
	shift    = 64 - 11 - 1
	bias     = 1023
```


```
	x := Float64bits(f)
	e := uint(x>>shift)&mask - bias

	// Keep the top 12+e bits, the integer part; clear the rest.
	if e < 64-12 {
		x &^= 1<<(64-12-e) - 1
	}
	int = Float64frombits(x)
	frac = f - int
	return
```
