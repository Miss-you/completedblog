#go使用时间作为种子生成随机数

设置时间种子使用time包
生成随机数需要math/rand包
打印输出使用fmt包

不设置时间种子的话，每次生成的rand值相同

```
package main

import "fmt"
import "math/rand"
import "time"

func Generate_Randnum() int{
	rand.Seed(time.Now().Unix())
	rnd := rand.Intn(100)
	
	fmt.Printf("rand is %v\n", rnd)
	
	return rnd
} 

func main(){
	Generate_Randnum()	
}
```

文件保存为GetRand.go，运行

```
feiqianyousadeMacBook-Pro:go yousa$ go run GetRand.go
rand is 56
feiqianyousadeMacBook-Pro:go yousa$ go run GetRand.go
rand is 25
```
rand.Intn(int n)函数生成从0-n的随机数
rand.Int()函数生成随机数

**这里不太确定随机数是从0开始还是1，请自己查询代码验证**


###另外，不设置时间种子的情况

```
package main

import "fmt"
import "math/rand"
//import "time"

func Generate_Randnum() int{
//	rand.Seed(time.Now().Unix())
	rnd := rand.Intn(100)
	
	fmt.Printf("rand is %v\n", rnd)
	
	return rnd
} 

func main(){
	Generate_Randnum()	
}
```

运行，生成结果不变

```
feiqianyousadeMacBook-Pro:go yousa$ go run GetRand.go
rand is 81
feiqianyousadeMacBook-Pro:go yousa$ go run GetRand.go
rand is 81
feiqianyousadeMacBook-Pro:go yousa$ go run GetRand.go
rand is 81
```