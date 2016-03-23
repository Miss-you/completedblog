#go源码阅读笔记（unsafe）

> unsafe 包主要是可以使得用户绕过go的类型规范检查，能够对指针以及其指向的区域进行读写操作。

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

先看一下unsafe的解释

```
/*
	Package unsafe contains operations that step around the type safety of Go programs.

	Packages that import unsafe may be non-portable and are not protected by the
	Go 1 compatibility guidelines.
*/
```
unsafe包包含一些可以绕过Go语言中的类型安全的操作，如果引用了unsafe包可能会导致不稳定，而且也不会受Go 1的兼容性所保护

```
package unsafe

// ArbitraryType is here for the purposes of documentation only and is not actually
// part of the unsafe package.  It represents the type of an arbitrary Go expression.
type ArbitraryType int

// Pointer represents a pointer to an arbitrary type.  There are four special operations
// available for type Pointer that are not available for other types:
//	- A pointer value of any type can be converted to a Pointer.
//	- A Pointer can be converted to a pointer value of any type.
//	- A uintptr can be converted to a Pointer.
//	- A Pointer can be converted to a uintptr.
// Pointer therefore allows a program to defeat the type system and read and write
// arbitrary memory. It should be used with extreme care.
```
unsafe 包主要是可以使得用户绕过go的类型规范检查，能够对指针以及其指向的区域进行读写操作。但是使用时要格外小心。

```
// The following patterns involving Pointer are valid.
// Code not using these patterns is likely to be invalid today
// or to become invalid in the future.
// Even the valid patterns below come with important caveats.
```
下面将会介绍哪些格式是允许的，其他的操作方法都是禁止的，就算现在不符合下面规范的一些写法可以编译通过，但是未来很可能就无法通过。

```
// Running "go vet" can help find uses of Pointer that do not conform to these patterns,
// but silence from "go vet" is not a guarantee that the code is valid.
//
```
这里说使用`go vet`命令可以查看Pointer的使用方法，但是我这里，，不知道为啥不行，等以后有机会查看一下是我理解错了还是运行方法有问题

```
// (1) Conversion of a *T1 to Pointer to *T2.
//
// Provided that T2 is no larger than T1 and that the two share an equivalent
// memory layout, this conversion allows reinterpreting data of one type as
// data of another type. An example is the implementation of
// math.Float64bits:
//
//	func Float64bits(f float64) uint64 {
//		return *(*uint64)(unsafe.Pointer(&f))
//	}
//
```
这里的第一个例子是，把一种数值类型转换成另一种的方法，用的是float64转成uint64，基本过程就是先把存储f值**的地址的这个数值**转换成unsafe.Pointer，将这个Pointer地址转换成 *uint64类型的地址，之后按照使用uint64类型指针那样，调用这个位置的值就可以了。这样就可以绕过Go语言的类型检查，将二进制强制转换数值类型。

当然uint64转成float64转换方式类似，代码如下

```
func Float64frombits(f uint64) float64 {
	return *(*float64)(unsafe.Pointer(&f))
}
```

上面这个用法在math包里面用的很频繁

接下来看其他用法

```
// (2) Conversion of a Pointer to a uintptr (but not back to Pointer).
//
// Converting a Pointer to a uintptr produces the memory address of the value
// pointed at, as an integer. The usual use for such a uintptr is to print it.
//
// Conversion of a uintptr back to Pointer is not valid in general.
//
// A uintptr is an integer, not a reference.
// Converting a Pointer to a uintptr creates an integer value
// with no pointer semantics.
// Even if a uintptr holds the address of some object,
// the garbage collector will not update that uintptr's value
// if the object moves, nor will that uintptr keep the object
// from being reclaimed.
//
// The remaining patterns enumerate the only valid conversions
// from uintptr to Pointer.
//
```
把一个Pointer转换成uintptr类型，通常这个用法是打印该地址

通常来说，你得到了这个uintptr类型的地址数值，Go是不允许你再把它转换回Pointer类型的。

这里注释提及了一种情况，如果对某处申请的内存，但是你获取了该处内存的地址的话（假设存储值为object_ptr），那么Go的垃圾回收机制在对象改变存储位置的时候，不会更新object_ptr的值，object_ptr也不会使得这个对象不被重新声明（nor will that uintptr keep the object from being reclaimed， 感觉这句的意思应该是，这个对象被删除了，但是object_ptr虽然不会被垃圾回收机制处理，也不会使得这个它原本指向的对象重新声明）

> 今天先写到这里吧。。感觉自己用的不多，还是有很多不明白




```
// (3) Conversion of a Pointer to a uintptr and back, with arithmetic.
//
// If p points into an allocated object, it can be advanced through the object
// by conversion to uintptr, addition of an offset, and conversion back to Pointer.
//
//	p = unsafe.Pointer(uintptr(p) + offset)
//
// The most common use of this pattern is to access fields in a struct
// or elements of an array:
//
//	// equivalent to f := unsafe.Pointer(&s.f)
//	f := unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.f))
//
//	// equivalent to e := unsafe.Pointer(&x[i])
//	e := unsafe.Pointer(uintptr(unsafe.Pointer(&x[0])) + i*unsafe.Sizeof(x[0]))
//
// It is valid both to add and to subtract offsets from a pointer in this way,
// but the result must continue to point into the original allocated object.
// Unlike in C, it is not valid to advance a pointer just beyond the end of
// its original allocation:
//
//	// INVALID: end points outside allocated space.
//	var s thing
//	end = unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + unsafe.Sizeof(s))
//
//	// INVALID: end points outside allocated space.
//	b := make([]byte, n)
//	end = unsafe.Pointer(uintptr(unsafe.Pointer(&b[0])) + uintptr(n))
//
// Note that both conversions must appear in the same expression, with only
// the intervening arithmetic between them:
//
//	// INVALID: uintptr cannot be stored in variable
//	// before conversion back to Pointer.
//	u := uintptr(p)
//	p = unsafe.Pointer(u + offset)
//
```

```
// (4) Conversion of a Pointer to a uintptr when calling syscall.Syscall.
//
// The Syscall functions in package syscall pass their uintptr arguments directly
// to the operating system, which then may, depending on the details of the call,
// reinterpret some of them as pointers.
// That is, the system call implementation is implicitly converting certain arguments
// back from uintptr to pointer.
//
// If a pointer argument must be converted to uintptr for use as an argument,
// that conversion must appear in the call expression itself:
//
//	syscall.Syscall(SYS_READ, uintptr(fd), uintptr(unsafe.Pointer(p)), uintptr(n))
//
// The compiler handles a Pointer converted to a uintptr in the argument list of
// a call to a function implemented in assembly by arranging that the referenced
// allocated object, if any, is retained and not moved until the call completes,
// even though from the types alone it would appear that the object is no longer
// needed during the call.
//
// For the compiler to recognize this pattern,
// the conversion must appear in the argument list:
//
//	// INVALID: uintptr cannot be stored in variable
//	// before implicit conversion back to Pointer during system call.
//	u := uintptr(unsafe.Pointer(p))
//	syscall.Syscall(SYS_READ, uintptr(fd), u, uintptr(n))
//
```

```
// (5) Conversion of the result of reflect.Value.Pointer or reflect.Value.UnsafeAddr
// from uintptr to Pointer.
//
// Package reflect's Value methods named Pointer and UnsafeAddr return type uintptr
// instead of unsafe.Pointer to keep callers from changing the result to an arbitrary
// type without first importing "unsafe". However, this means that the result is
// fragile and must be converted to Pointer immediately after making the call,
// in the same expression:
//
//	p := (*int)(unsafe.Pointer(reflect.ValueOf(new(int)).Pointer()))
//
// As in the cases above, it is invalid to store the result before the conversion:
//
//	// INVALID: uintptr cannot be stored in variable
//	// before conversion back to Pointer.
//	u := reflect.ValueOf(new(int)).Pointer()
//	p := (*int)(unsafe.Pointer(u))
//
```

```
// (6) Conversion of a reflect.SliceHeader or reflect.StringHeader Data field to or from Pointer.
//
// As in the previous case, the reflect data structures SliceHeader and StringHeader
// declare the field Data as a uintptr to keep callers from changing the result to
// an arbitrary type without first importing "unsafe". However, this means that
// SliceHeader and StringHeader are only valid when interpreting the content
// of an actual slice or string value.
//
//	var s string
//	hdr := (*reflect.StringHeader)(unsafe.Pointer(&s)) // case 1
//	hdr.Data = uintptr(unsafe.Pointer(p))              // case 6 (this case)
//	hdr.Len = uintptr(n)
//
// In this usage hdr.Data is really an alternate way to refer to the underlying
// pointer in the slice header, not a uintptr variable itself.
//
// In general, reflect.SliceHeader and reflect.StringHeader should be used
// only as *reflect.SliceHeader and *reflect.StringHeader pointing at actual
// slices or strings, never as plain structs.
// A program should not declare or allocate variables of these struct types.
//
//	// INVALID: a directly-declared header will not hold Data as a reference.
//	var hdr reflect.StringHeader
//	hdr.Data = uintptr(unsafe.Pointer(p))
//	hdr.Len = uintptr(n)
//	s := *(*string)(unsafe.Pointer(&hdr)) // p possibly already lost
//
type Pointer *ArbitraryType

// Sizeof takes an expression x of any type and returns the size in bytes
// of a hypothetical variable v as if v was declared via var v = x.
// The size does not include any memory possibly referenced by x.
// For instance, if x is a slice,  Sizeof returns the size of the slice
// descriptor, not the size of the memory referenced by the slice.
func Sizeof(x ArbitraryType) uintptr

// Offsetof returns the offset within the struct of the field represented by x,
// which must be of the form structValue.field.  In other words, it returns the
// number of bytes between the start of the struct and the start of the field.
func Offsetof(x ArbitraryType) uintptr

// Alignof takes an expression x of any type and returns the required alignment
// of a hypothetical variable v as if v was declared via var v = x.
// It is the largest value m such that the address of v is always zero mod m.
// It is the same as the value returned by reflect.TypeOf(x).Align().
// As a special case, if a variable s is of struct type and f is a field
// within that struct, then Alignof(s.f) will return the required alignment
// of a field of that type within a struct.  This case is the same as the
// value returned by reflect.TypeOf(s.f).FieldAlign().
func Alignof(x ArbitraryType) uintptr
```