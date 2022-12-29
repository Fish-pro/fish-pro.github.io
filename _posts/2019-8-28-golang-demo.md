---
layout: post
title: Golang demo
author: Fish-pro
tags:
- go
- golang
date: 2019-08-28 13:56 +0800
---
### golang的学习方向

go语言我们可以简单的写成Golanguage

一个工程只能有一个main函数

命令：

```golang
go build xxx.go
xxx.exe
go run xxx.go //不编译程序就执行
```

导包命名

```golang
import(
	."fmt"
)
PrintIn("hello world")

import(
	f"fmt"
)
f.PrintIn("hello world")

import(
    "database/sql"
    _"github.com/ziutek/mymysql/godrv"
)
//调用了包中的init函数
import(
	//标准库包
	//程序内部包
	//第三方包
)
```

编译命令

```golang
go run -n test.go
go run -work test.go
go build //默认编译目录下所有文件 同一目录下两个main不能编译
go install //将执行文件移动至bin目录下
go get //安装第三方包
go get -x //可以看到过程，后跟网址，不需要https://
go clean //移除文件
go fmt //格式化代码
go test //读取测试文件
go doc net/http //查看包信息
	go doc fmt Println
godoc -http=:9527 //本地端口开启查看go复制文档
go list //列出所有包
go env //查看环境变量
go version //查看版本
```

### 变量的使用

```golang
package main

import "fmt"

func main() {
	/*
	变量：variable
	概念：一小块内存，用于存储数据，在程序运行过程中数值可以改变

	使用：
		step1:变量的声明，也叫定义
			第一种：var 变量名 数据类型
				变量名 = 赋值

				var 变量名 数据类型 = 赋值
			第二种：类型推断，省略数据类型
				var 变量名 = 赋值
			第三种：变量 ：= 赋值
		step2:变量的访问，赋值和取值
			直接通过变量的名字访问
	go语言的特性：
		静态语言：强类型语言
			go,java,c++,c#...
		动态语言：弱类型语言
			JavaScript,php,python,ruby...
	*/
	//第一种：定义变量，然后进行赋值
	var num1 int
	num1 = 20
	fmt.Printf( "num1的数值是：%d\n",num1)
	var num2 int = 15
	fmt.Printf( "num2的数值是：%d\n",num2)
	//第二种：类型推断
	var name="Tom"
	fmt.Printf("类型是：%T,数值是：%s\n",name,name)
	//简短定义，也叫简短声明
	sum := 100
	fmt.Printf("sum的值是:%d\n",sum)

	//多个变量同时定义
	var a,b,c int
	a=1
	b=2
	c=3
	fmt.Println(a,b,c)
	var m,n int = 100,200
	fmt.Println(m,n)
	var n1,f1,s1=100,3.14,"GO"
	fmt.Println(n1,f1,s1)

	var (
		studentName = "Jerry"
		age = 18
		sex = "男"
	)
	fmt.Printf("姓名：%s 年龄：%d 性别：%s",studentName,age,sex)
}

```

#### 注意事项

```golang
package main

import "fmt"

//var a int
//a = 10000 //non-declaration statement outside function body
var b = 20000
var c int = 30000
//d := 40000//语法错误
func main() {
	/*
	注意点：
	1.变量必须先定义才能使用
	2.变量的类型和赋值必须一致
	3.同一个作用域内变量名不能冲突
	4.简短定义方式，左边的变量至少有一个是新的
	5.简短定义方式不能定义全局变量
	6.变量的灵芝，就是默认值
		整型：0
		浮点型：0
		字符串：""
	7.声明的变量没有使用会报错，导入的包没有使用会报错
	*/
	var num int
	num = 100
	fmt.Printf("num的数值是：%d,地址是：%p\n",num,&num)

	num = 200
	fmt.Printf("num的数值是：%d,地址是：%p\n",num,&num)

	//fmt.Printf("num的数值是：%d,地址是：%p\n",num2,&num2)//undefined: num2
	//var name string
	//name=100
	//fmt.Printf("num的数值是：%d,地址是：%p\n",name,&name)//cannot use 100 (type int) as type string in assignment

	//var num int
	//num = 300
	//fmt.Printf("num的数值是：%d,地址是：%p\n",num,&num)

	//num := 1000//no new variables on left side of :=
	//fmt.Printf("num的数值是：%d,地址是：%p\n",num,&num)

	num,sex := 300,"你好"
	fmt.Println(num,sex)

	fmt.Println(b,c)

	var m int
	fmt.Println(m)//整数，默认是0
	var n float64//0.0----->0
	fmt.Println(n)
	var s string//""
	fmt.Println(s)
	var s2 []int//切片[]
	fmt.Println(s2)
	fmt.Println(s2 == nil)
	//var sum = 100 //sum declared and not used

}
```

### 常量的使用

```golang
package main

import "fmt"

func main() {
	/*
		常量(constant)：
		1.概念:同变量类似，程序执行过程中数值不能改变
		2.语法：
			显示类型定义
			隐式类型定义
		3.常数
			固定的数字:100，"abc"
	*/
	fmt.Println(100)
	fmt.Println("hello")

	//1.定义常量
	const PATH string = "http://www.baidu.com"
	const PI = 3.14
	fmt.Println(PATH)
	fmt.Println(PI)

	//2.尝试修改常量的数值，报错
	//PATH = "http://www.sina.com"//cannot assign to PATH

	//3.定义一组常量
	const C1, C2, C3 = 100, 3.15, "haha"
	const (
		MALE   = 0
		FEMALE = 1
		UNKNOW = 2
	)

	//4.一组常量中，如果某一个常量没有初始值，默认和上一行一致
	const (
		a int = 100
		b
		c string = "ruby"
		d
		e
	)
	fmt.Printf("%T,%d\n", a, a)
	fmt.Printf("%T,%d\n", b, b)
	fmt.Printf("%T,%s\n", c, c)
	fmt.Printf("%T,%s\n", d, d)
	fmt.Printf("%T,%s\n", e, e)

	//5.枚举类型：使用常量作为枚举类型，一组相关数值的数据
	const (
		SPRING = 0
		SUMMER = 1
		AUTUMN = 2
		WINTER = 3
	)
	//6.常量定义没有用，不会报错
}
```

### iota的使用

```golang
package main

import "fmt"

func main() {
	const (
		A = iota   //0
		B          //1
		C          //2
		D = "haha" //3
		E          //4
		F = 100    //5
		G          //6
		H = iota   //7
		I          //8
	)
	const (
		J = iota //0
	)
	fmt.Println(A)
	fmt.Println(B)
	fmt.Println(C)
	fmt.Println(D)
	fmt.Println(E)
	fmt.Println(F)
	fmt.Println(G)
	fmt.Println(H)
	fmt.Println(I)
	fmt.Println(J)
}

```

### go基本数据类型

整数

| 1    | **uint8** 无符号 8 位整型 (0 到 255)                         |
| ---- | ------------------------------------------------------------ |
| 2    | **uint16** 无符号 16 位整型 (0 到 65535)                     |
| 3    | **uint32** 无符号 32 位整型 (0 到 4294967295)                |
| 4    | **uint64** 无符号 64 位整型 (0 到 18446744073709551615)      |
| 5    | **int8** 有符号 8 位整型 (-128 到 127)                       |
| 6    | **int16** 有符号 16 位整型 (-32768 到 32767)                 |
| 7    | **int32** 有符号 32 位整型 (-2147483648 到 2147483647)       |
| 8    | **int64** 有符号 64 位整型 (-9223372036854775808 到 9223372036854775807) |

浮点数

| 1    | **float32** IEEE-754 32位浮点型数 |
| ---- | --------------------------------- |
| 2    | **float64** IEEE-754 64位浮点型数 |

复数

| 3    | **complex64** 32 位实数和虚数  |
| ---- | ------------------------------ |
| 4    | **complex128** 64 位实数和虚数 |

其他数字类型

| 1    | **byte** 类似 uint8                      |
| ---- | ---------------------------------------- |
| 2    | **rune** 类似 int32                      |
| 3    | **uint** 32 或 64 位                     |
| 4    | **int** 与 uint 一样大小                 |
| 5    | **uintptr** 无符号整型，用于存放一个指针 |

### 类型转换

```golang
package main

import "fmt"

func main() {
	/*
	数据类型转换Type Convert
		go语言是静态语言，定义，赋值，运算必须类型一致
		语法格式：Type(Value)
		注意点：兼容类型可以转换
		常数：在有需要的时候自动转换
		变量：需要手动转换
	*/
	var a int8
	a = 10

	var b int16
	b = int16(a)
	fmt.Println(a,b)

	f1 := 4.83
	var c int
	c = int(f1)
	fmt.Println(f1,c)

	f1 = float64(a)
	fmt.Println(f1,a)

	//b1 := true
	//a = int16(b1)//cannot convert b1 (type bool) to type int16

	sum := f1+100
	fmt.Printf("%T,%f",sum,sum)
}

```

### 运算符

```golang
package main

import "fmt"

func main() {
	/*
		算数运算符:+，-，*，/，%,++,--
			+
			-
			*乘法
			/除法
				取商
				取余
			++
				给自己加1
			--
				给自己减1
	*/
	a := 10
	b := 3
	sum := a + b
	fmt.Printf("%d + %d = %d\n", a, b, sum)

	sub := a - b
	fmt.Printf("%d - %d = %d\n", a, b, sub)

	mul := a * b
	fmt.Printf("%d * %d = %d\n", a, b, mul)

	div := a / b //取商
	mod := a / b //取余
	fmt.Printf("%d / %d = %d\n", a, b, div)
	fmt.Printf("%d %% %d = %d\n", a, b, mod)

	c := 3
	c++ //就是给c+1
	fmt.Println(c)

	c-- //给c减1
	fmt.Println(c)

	//sum2 := c+ c++ - --c//不支持
}
```

### 关系运算符

```golang
package main

import "fmt"

func main() {
	/*
	关系运算符：>,<,<=,>=,==,!=
	关系运算符结果总是布尔值
	*/
	a := 3
	b := 5
	c := 3
	res1 := a>b
	res2 := b>c
	fmt.Printf("%T,%t\n",res1,res1)
	fmt.Printf("%T,%t\n",res2,res2)

	fmt.Println(a==c,a==b)
}

```

### 逻辑运算符

| 运算符 | 描述                                                         | 实例               |
| :----- | :----------------------------------------------------------- | :----------------- |
| &&     | 逻辑 AND 运算符。 如果两边的操作数都是 True，则条件 True，否则为 False。 | (A && B) 为 False  |
| \|\|   | 逻辑 OR 运算符。 如果两边的操作数有一个 True，则条件 True，否则为 False。 | (A \|\| B) 为 True |
| !      | 逻辑 NOT 运算符。 如果条件为 True，则逻辑 NOT 条件 False，否则为 True。 | !(A && B) 为 True  |

### 位运算符

| 运算符 | 描述                                                         | 实例                                   |
| :----- | :----------------------------------------------------------- | :------------------------------------- |
| &      | 按位与运算符"&"是双目运算符。 其功能是参与运算的两数各对应的二进位相与。 | (A & B) 结果为 12, 二进制为 0000 1100  |
| \|     | 按位或运算符"\|"是双目运算符。 其功能是参与运算的两数各对应的二进位相或 | (A \| B) 结果为 61, 二进制为 0011 1101 |
| ^      | 按位异或运算符"^"是双目运算符。 其功能是参与运算的两数各对应的二进位相异或，当两对应的二进位相异时，结果为1。 | (A ^ B) 结果为 49, 二进制为 0011 0001  |
| <<     | 左移运算符"<<"是双目运算符。左移n位就是乘以2的n次方。 其功能把"<<"左边的运算数的各二进位全部左移若干位，由"<<"右边的数指定移动的位数，高位丢弃，低位补0。 | A << 2 结果为 240 ，二进制为 1111 0000 |
| >>     | 右移运算符">>"是双目运算符。右移n位就是除以2的n次方。 其功能是把">>"左边的运算数的各二进位全部右移若干位，">>"右边的数指定移动的位数。 | A >> 2 结果为 15 ，二进制为 0000 1111  |

```golang
package main

import "fmt"

func main() {
	/*
	位运算符
		将数值转为二进制后，按位操作
		按位与（&）:
			对应位的值如果都为1才为1，有一个为0就为0
		按位或（|）:
			对应位只要有一个为1都为1，全为0才为0
		异或（^）:
			二元：a^b
				相同为0，不同为1
			一元：^a
				按位取反
				1---->0
				0---->1
		位清空（&^）：
			对于 a&^b
				对于b上的数字
				为0，则取a对应位上的数值
				为1，则结果位就取0
	位移运算符
	<< 按位左移
	>> 按位右移
	*/
	a := 60
	b := 13
	/*
	a:60,  11 1100
	a:13,  00 1101(四位断开，开头补0)
	&      00 1100  12
	|      11 1101  61
	^	   11 0001  49
	&^     11 0000  48
	^a     00 0011  -61
	*/
	fmt.Printf("a:%d,%b\n",a,a)
	fmt.Printf("a:%d,%b\n",b,b)

	res1 := a&b
	res2 := a|b
	res3 := a^b
	res4 := a&^b
	res5 := ^a
	fmt.Println(res1)
	fmt.Println(res2)
	fmt.Println(res3)
	fmt.Println(res4)
	fmt.Println(res5)

	c:=8
	/*
	c:...0000 1000
	   0000 100000  32
	     0000 0010  2
	*/
	res6 := c<<2
	res7 := c>>2
	fmt.Println(res6)
	fmt.Println(res7)

}
```

### 赋值运算符

| 运算符 | 描述                                           | 实例                                       |
| :----- | :--------------------------------------------- | :----------------------------------------- |
| =      | 简单的赋值运算符，将一个表达式的值赋给一个左值 | C = A  将 A 赋值给 C，结果：21             |
| +=     | 相加后再赋值                                   | C += A 等于 C = C + A，结果：42            |
| -=     | 相减后再赋值                                   | C -= A 等于 C = C - A，结果：21            |
| *=     | 相乘后再赋值                                   | C *= A 等于 C = C * A，结果：441           |
| /=     | 相除后再赋值                                   | C /= A 等于 C = C / A，结果：21            |
| %=     | 求余后再赋值                                   | C %= A 等于 C = C % A，结果：0//不记入计算 |
| <<=    | 左移后赋值                                     | C <<= 2 等于 C = C << 2，结果：84          |
| >>=    | 右移后赋值                                     | C >>= 2 等于 C = C >> 2，结果：21          |
| &=     | 按位与后赋值                                   | C &= 2 等于 C = C & 2，结果：0             |
| ^=     | 按位异或后赋值                                 | C ^= 2 等于 C = C ^ 2，结果：2             |
| \|=    | 按位或后赋值                                   | C \|= 2 等于 C = C \| 2，结果：2           |

### 输入输出

```golang
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	/*
	输入和输出
		fmt包：输入，输出
		输出：
			Print()//打印
			Printf()//格式化打印
			Println()//打印之后换行
		格式化打印占位符
			%v,原样输出
			%T,打印类型
			%s,字符串
			%t,bool类型
			%f,浮点型
			%d,整数
			%b,二进制
			%o,八进制
			%x,%X,16禁止
				%x:0-9,a-f
				%X:0-9,A-F
			%c,打印字符
			%p,打印地址
	输入：
		fmt.Scanln(&x,&y)
		fmt.Scanf("%d,%f",&x,&y)
	bufio包

	*/
	//var x int
	//var y float64
	//fmt.Println("请输入一个整数，一个浮点类型：")
	//fmt.Scanln(&x,&y)//读取键盘的输入，通过操作地址，赋值给x和y,阻塞式
	//fmt.Printf("a的数值：%d,b的数值：%f\n",x,y)
	//
	//fmt.Scanf("%d,%f",&x,&y)
	//fmt.Printf("x:%d,y:%f",x,y)

	fmt.Println("请输入一个字符串")
	reader := bufio.NewReader(os.Stdin)
	s1,_ := reader.ReadString('\n')
	fmt.Println("读到的数据",s1)
}
```

### if-else if -else

```golang
package main

import "fmt"

func main() {
	/*
	条件语句：if
	语法格式：
		if 条件表达式{
			//
		}
	条件语句：if-else
	语法格式：
		if 条件表达式{
			//
		} else if{
			//
		} else{
			//
		}
	注意点：
	1.if后的{一定是和if写在同一行
	2.else跟在}后，否则报错
	*/

	var x int
	fmt.Println("请输入一个数字:")
	fmt.Scanf("%d",&x)
	if x>0{
		fmt.Println("这个数是正数")
	}
	if x > 10 {
		fmt.Println("这个数字大于10")
	} else if x==10 {
		fmt.Println("这个数等于10")
	} else{
		fmt.Println("这个数字小于10")
	}

	fmt.Println("main..over...")
}

package main

import "fmt"

func main() {
	/*
	if语句的其他写法(类似于python的匿名函数)
	if 初始化;条件{
		//作用域只在if内
	}
	*/
	if num1:=4; num1>0{
		fmt.Println("num1正数")
	}else if num1<0{
		fmt.Println("num1负数")
	}
	//fmt.Println(num1)//undefined: num1

	num2:=5
	if num2>0{
		fmt.Println("num2正数")
	}
	fmt.Println(num2)
}

```

### switch

```golang
package main

import "fmt"

func main() {
	/*
	switch语句
	语法结构：
	switch 变量名{
	case 数值1：分支1，
	case 数值2：分支2，
	...
	default:
		最后一个分支
	}
	注意事项：
	1.switch可以做用在其它类型上，case后的数值的必须和变量类型一致
	2.case是无序的
	3.case后的数值是唯一的
	4.default是可选的操作
	*/
	num := 6
	switch num {
	case 1:
		fmt.Println("第一节度")
	case 2:
		fmt.Println("第二季度")
	case 3:
		fmt.Println("第三季度")
	case 4:
		fmt.Println("第四季度")
	//default:
	//	fmt.Println("输入错误")
	}
	fmt.Println("main...over...")
}



package main

import "fmt"

func main() {
	/*
		1.switch的标注写法：
		switch 变量{
		case 赋值1:分支1
		case 赋值2:分支2
		。。。
		default：
			最后一个分支
		}
		2.省略switch后的变量，相当于直接作用在true上
		switch{//true
		case true:
		case false:
		}
		3.case后可以同时跟多个数值
		switch 变量{
		case 数值1，数值2，数值3:
		case 数值4：
		default：
		最后一个分支
		}
	4.switch后可以多一条初始化语句
	switch初始化语句;变量{

	}
	*/
	switch {
	case true:
		fmt.Println("haha")
	case false:
		fmt.Println("lala")
	}
	/*
		0-59 不及格
		60-69 及格
		70-79 中
		80-89 良好
		90-100 优秀
	*/
	score := 88
	switch {
	case score >= 0 && score < 60:
		fmt.Println("不及格")
	case score >= 60 && score < 70:
		fmt.Println("及格")
	case score >= 70 && score < 80:
		fmt.Println("中")
	case score >= 80 && score < 90:
		fmt.Println("良好")
	case score >= 90 && score <= 100:
		fmt.Println("优秀")
	default:
		fmt.Println("成绩有误")
	}

	letter := "A"
	switch letter {
	case "A","B":
		fmt.Println("哈哈")
	case "c":
		fmt.Println("呵呵")
	default:
		fmt.Println("嘻嘻")
	}
	fmt.Println("++++++++++++++++++++++++")
	month := 5
	day := 0
	year := 2019
	switch month {
	case 1,3,5,7,8,10,12:
		day = 31
	case 2:
		if year%400==0 || year%4==0 && year%100 != 0{
			day = 29
		} else{
			day = 28
		}
	default:
		fmt.Println("月份有误")
	}
	fmt.Printf("%d年%d月有%d天",year,month,day)

	fmt.Println("++++++++++++++++++++++++")
	switch language := "golang";language {
	case "golang":
		fmt.Println("go语言")
	case "java":
		fmt.Printf("java语言")
	default:
		fmt.Println("其他语言")
	}
	//fmt.Println(language)//undefined: language
}

```

### switch中的break和fallthrough

```golang
package main

import "fmt"

func main() {
	/*
		switch中的break和fallthrough语句
		break的用法，也可以用在for中
			break强制结束语句，从而结束switch分支
		fallthrough用于穿透switch语句，当case之后，会执行case之后的case
			只能写在case最后一行
	*/
	n := 2
	switch n {
	case 1:
		fmt.Println("我是熊大")
		fmt.Println("我是熊大")
		fmt.Println("我是熊大")
	case 2:
		fmt.Println("我是熊二")
		fmt.Println("我是熊二")
		break //用于强制结束case
		fmt.Println("我是熊二")

	case 3:
		fmt.Println("我是光头强")
		fmt.Println("我是光头强")
		fmt.Println("我是光头强")

	}
	fmt.Println("main...over...")

	fmt.Println("+++++++++++++++++++++++++++++++")
	m := 2
	switch m {
	case 1:
		fmt.Println("one")
	case 2:
		fmt.Println("two")
		fallthrough
	case 3:
		fmt.Println("three")
		fallthrough
	case 4:
		fmt.Println("four")
	}
}

```

### for循环语句

```golang
package main

import "fmt"

func main() {
	/*
		1.标准for循环
			使用循环结构某些代码会重复执行
			语法：
			for 表达式1；表达式2；表达式3{
				//循环体
			}
			表达式1:只执行一次
			表达式2：bool类型，循环的条件

		2.同时省略表达式1和表达式3
			for 表达式2{
			}
			相当于while（条件）
		3.同时省略三个表达式
			for{
			}
			相当于while(true)语句
			当for循环中省略表达式2，就相当于作用在true上
		4.其他的写法：for循环中省略介个表达式都可以
			省略表达式1：变量定义在外部
			省略表达式2：for永远作用在true上
			上略表达式3：变量变化放在循环体内部
	*/
	for i := 1; i <= 5; i++ {
		fmt.Println("hollo world...")
	}
	//fmt.Println(i)//undefined: i
	fmt.Println("-------------------------")
	i := 1
	for i <= 5 {
		fmt.Println("hollo world...")
		i++
	}
	fmt.Println(i) //i=6

	fmt.Println("------------------------")
	for {
		fmt.Println("i------>")
		i++
	}
}
```

### for练习

```golang
package main

import "fmt"

func main() {
	/*
		for循环练习题：
		练习1：打印58-23数字
		练习2：球1-100的和
		练习3：打印1-100,能被3整除的数字但是不能被5整除的数字，统计打印的数字的个数，每行打印5个
	*/
	for i := 58; i >= 23; i-- {
		fmt.Printf("%d ", i)
	}
	sum := 0
	for i := 1; i <= 100; i++ {
		sum += i
	}
	fmt.Println(sum)

	num := 0
	for i := 1; i <= 100; i++ {
		if i%3 == 0 && i%5 != 0 {
			fmt.Printf("%d  ", i)
			num++
			if num%5 == 0 {
				fmt.Printf("\n")
			}
		}
	}
	fmt.Printf("\n")
	fmt.Println("个数：", num)
}
```

### for打印九九乘法表

```golang
package main

import "fmt"

func main() {
	/*
	九九乘法表
	*/
	for i:=1;i<=9;i++{
		for j:=1;j<=9;j++{
			if j<=i{
				fmt.Printf("%d x %d = %d\t",i,j,i*j)
			}
		}
		fmt.Println()
	}
}

```

### for-break-continue

```golang
package main

import "fmt"

func main() {
	/*
	循环结束：
		循环条件不满足，循环自动结束了
		但是可以通过break和continue来强制结束循环
	循环语句控制
	break:
	continue:
	*/
	for i:=1;i<=10;i++{
		if i==5{
			break
		}
		fmt.Println(i)
		if i==3{
			continue
		}
		fmt.Println("这次结束了")
	}
	fmt.Println("main...over...")
}
```

### for求水仙花数

```golang
package main

import (
	"fmt"
	"math"
)

func main() {
	/*
		水仙花数
	*/
	for i := 1; i <= 9; i++ {
		for j := 0; j <= 9; j++ {
			for k := 0; k <= 9; k++ {
				n := i*100 + j*10 + k
				if math.Pow(float64(i), 3)+math.Pow(float64(j), 3)+math.Pow(float64(k), 3) == float64(n) {
					fmt.Println(n)
				}
			}
		}
	}
	for i := 100; i < 1000; i++ {
		x := i / 100
		y := i / 10 % 10
		z := i % 10
		if math.Pow(float64(x), 3)+math.Pow(float64(y), 3)+math.Pow(float64(z), 3) == float64(i) {
			fmt.Println(i)
		}
	}
}
```

### for求素数

```golang
package main

import (
	"fmt"
	"math"
)

func main() {
	/*
		求2-100内的素数
	*/
	for i := 2; i <= 100; i++ {
		flag := true
		for j := 2; j <= int(math.Sqrt(float64(i))); j++ {
			if i%j == 0 {
				flag = false
				break
			}
		}
		if flag {
			fmt.Println(i)
		}
	}
}
```

### goto语句

```golang
package main

import "fmt"

func main() {
	/*
		go to语句
	*/
	var a = 10
LOOP:
	for a < 20 {
		if a == 15 {
			a += 1
			goto LOOP
		}
		fmt.Println(a)
		a++
	}

	fmt.Println("--------------------------")
	for i := 0; i < 10; i++ {
		for j := 0; j < 10; j++ {
			if j == 2 {
				goto breakHere
			}
		}
	}
	//手动返回，避免执行进入标签
	return

breakHere:
	fmt.Println("done....")
}
```

### 随机数获取time/rand

```golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	/*
		生成随机数random
		伪随机数，根据一定的算法公式算出来的
		math/rand
		Intn(n)---->[0,n)
	*/
	num1 := rand.Int()
	fmt.Println(num1)

	for i := 0; i < 10; i++ {
		num := rand.Intn(10)
		fmt.Println(num)
	}

	rand.Seed(3)
	num2 := rand.Intn(10)
	fmt.Println("--->", num2)

	t1 := time.Now()
	fmt.Println(t1)
	fmt.Printf("%T\n", t1)
	//时间戳：指定时间，距离1970年1月1日的描述
	timeStamp1 := t1.Unix()
	fmt.Println(timeStamp1) //秒
	timeStemp2 := t1.UnixNano()
	fmt.Println(timeStemp2) //纳秒
	//设置中子数，可以设置为时间戳
	rand.Seed(time.Now().UnixNano())
	for i := 1; i <= 10; i++ {
		fmt.Println("---->", rand.Intn(100))
	}

	num3 := rand.Intn(46) + 3 //[3,48]
	fmt.Println("++++++++", num3)
}
```

### 数组（array）

```golang
package main

import "fmt"

func main() {
	/*
		数据类型：
			基本类型：整数 浮点 布尔 字符串
			复合类型：array slice map struct pointer function channel...
		数组：
			1.概念：存储一组相同数据类型的数据结构
				存储一组数据
			2.语法：
				var 数组名 [长度] 数据类型
				var 数组名 = [长度] 数据类型{元素1，元素2，...}
				var 数组名:=[...]数据类型{元素....}
			3.通过下标访问
				下标，也叫索引：index,
				默认从0开始的整数，到长度件1
				数组名[index]
					赋值
					取值
				不能越界：[0,长度-1]
			4.长度和容量
				len(array/map/slice/string),长度
				cap(),容器长度
	*/
	var num1 int
	num1 = 100
	num1 = 200
	fmt.Println(num1)
	fmt.Printf("%p\n", &num1)

	//step1:创建数组
	var arr1 [4] int
	fmt.Printf("arr1地址：%p\n", &arr1) //0xc000054140
	//step2:数据的访问
	arr1[0] = 1
	arr1[1] = 2
	arr1[2] = 3
	arr1[3] = 4
	fmt.Printf("arr1[3]地址：%p\n", &arr1[3]) //0xc000054158
	fmt.Println(arr1[0])                   //打印第一个数值
	fmt.Println(arr1[2])                   //打印第三个数值
	//fmt.Println(arr1[4])//invalid array index 4 (out of bounds for 4-element array)

	fmt.Println("数组的长度:", len(arr1)) //实际存储量
	fmt.Println("数组的容量:", cap(arr1)) //容器能够存储的最大的数量
	//因为数组定长，长度与容量相等
	arr1[0] = 100
	fmt.Println(arr1)
	fmt.Println(arr1[0])

	//数组的其他创建方式
	var a [4] int //var a= [4] int
	fmt.Println(a)

	var b = [4]int{1, 2, 3, 4}
	fmt.Println(b)

	var c = [5]int{1, 2, 3}
	fmt.Println(c)

	var d = [5]int{1: 1, 3: 2}
	fmt.Println(d)

	var e = [5]string{"tom", "jerry"}
	fmt.Println(e)

	f := [...]int{1, 2, 3, 4, 5}
	fmt.Println(f)
	fmt.Println(len(f))

	g := [...]int{1: 3, 6: 5}
	fmt.Println(g)
	fmt.Println(len(g))
}
```

### for-range遍历array

```golang
package main

import "fmt"

func main() {
	/*
		数组的遍历
			一次访问数组中的元素
			方法一：arr[0],arr[1]...
			方法二：通过循环遍历下标
				for i:0;i<len(arr);i++{
					arr[i]
				}
			方法三：使用range
				range词义范围
				不需要操作数组的下标，到达数组的末尾，自动接收for range循环，
				每次都从数组中获取下标和相应的数值
	*/
	arr1 := [5]int{1, 2, 3, 4, 5}
	fmt.Println(arr1[0])
	fmt.Println(arr1[1])
	fmt.Println(arr1[2])
	fmt.Println(arr1[3])
	fmt.Println(arr1[4])

	fmt.Println("-------------------------")
	for i := 0; i < len(arr1); i++ {
		arr1[i] = i*2 + 1
	}
	fmt.Println(arr1)

	fmt.Println("-------------------------")
	for index, value := range arr1 {
		fmt.Printf("下标是:%d,数值是:%d\n", index, value)
	}

	sum := 0
	for _, value := range arr1 {
		sum += value
	}
	fmt.Println(sum)
}
```

### 数组类型（array_type）

```golang
package main

import "fmt"

func main() {
	/*
		数据类型：
			基本类型：int float string bool
			符合类型：array slice map function pointer chnnel
		数组的数据类型：
			[size]type
		值类型：理解为存储的数值本身
			将数据传递给其他变量，传递的是数据的副本（备份）
			int float string bool array
		引用类型：
			slice map ...
	*/
	num := 10
	fmt.Printf("%T\n", num)

	//1.数据类型
	arr1 := [4]int{1, 2, 3, 4}
	arr2 := [3]float64{2.15, 3.18, 6.19}
	arr3 := [4]int{5, 6, 7, 8}
	arr4 := [2]string{"tom", "jerry"}
	fmt.Printf("%T\n", arr1) //[4]int
	fmt.Printf("%T\n", arr2) //[3]float64
	fmt.Printf("%T\n", arr3) //[4]int
	fmt.Printf("%T\n", arr4) //[2]string

	//2.赋值
	num2 := num //值传递
	fmt.Println(num, num2)
	num2 = 20
	fmt.Println(num, num2)

	//数组呢
	arr5 := arr1
	fmt.Println(arr1)
	fmt.Println(arr5)
	//arr5[0] = 100//值传递
	fmt.Println(arr1)
	fmt.Println(arr5)

	a := 3
	b := 4
	fmt.Println(a == b)       //比较a和b的数值是否相等
	fmt.Println(arr1 == arr5) //比较数组对应位置的数值是否相等
	//fmt.Println(arr1==arr2)//invalid operation: arr1 == arr2 (mismatched types [4]int and [3]float64)
}
```

### array-sort(冒泡排序)

```golang
package main

import "fmt"

func main() {
	/*
		数组的排序：
			让数组中的元素具有一定的顺序
			arr := [5]int{15,23,8,10,7}
				升序：[7,8,10,15,23]
				降序：[23,15,10,8,7]
		排序算法：
			冒泡排序 插入排序 选择排序 希尔排序 堆排序 快速排序
		冒泡排序：（Bubble Sort）
			依次比较两个相邻的元素，如果他们的顺序从大到小，就把它们交换过来
	*/
	arr := [5]int{15, 23, 8, 10, 7}
	//第一轮排序
	//for j:=0;j<4;j++{
	//	if arr[j]>arr[j+1]{
	//		arr[j],arr[j+1] = arr[j+1],arr[j]
	//	}
	//}
	//fmt.Println(arr)

	for i := 1; i < len(arr); i++ {
		for j := 0; j < len(arr)-1; j++ {
			if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
			}
		}
		fmt.Println(arr)
	}
}
```

### 多维数组

```golang
package main

import "fmt"

func main() {
	/*
		一位数组：存储的多个数据是数值本身
			a1:=[3]int{1,2,3}
		二维数组：存储的是一堆一堆的
			a2:=[0][0]int{{}}
			该二维数组的长度就是3
			存储的元素是一堆数组，一维数组的元素是数值，每个一维数组的长度为4
		多维数组:...
	*/
	a2 := [0][0]int{{}}
	fmt.Println(a2)
	fmt.Printf("%T\n", a2)
	fmt.Printf("二维数组的地址：%p\n", &a2)     //0xc000040060
	fmt.Printf("二维数组的长度：%d\n", len(a2)) //3

	fmt.Printf("二维数组的地址：%p\n", &a2[0])     //0xc000040060
	fmt.Printf("二维数组的地址：%p\n", &a2[1])     //0xc000040080
	fmt.Printf("二维数组的长度：%d\n", len(a2[0])) //4

	//遍历二维数组
	for i := 0; i < len(a2); i++ {
		for j := 0; j < len(a2[i]); j++ {
			fmt.Print(a2[i][j], "\t")
		}
		fmt.Println()
	}
}

```

### 切片（slice）使用

```golang
package main

import "fmt"

func main() {
	/*
		数组array:
			存储一组相同数据类型的数据结构
				特点：定长
		切片：slice
			同数组类似，也叫做变长数组或者动态数组
				特点：变长
			是一个引用类型的容器，指向了一个底层数组
		make()
			func make(t Type,size ...IntegerType) Type
			第一个参数：类型
				slice,map,chan
			第二个参数:长度
				实际存储元素的数量
			第三个参数：容量cap
				最多能够存储的元素的数量
		append():专门用于切片尾部追加元素
			append(slice1,ele1,ele2,...)
			append(slice1,slice2...)
	*/
	//1.数组
	arr := [4]int{1, 2, 3, 4}
	fmt.Println(arr)

	//2.切片
	var s1 []int
	fmt.Println(s1)

	s2 := []int{1, 2, 3, 4} //变长
	fmt.Println(s2)
	fmt.Printf("arr:%T，s2:%T\n", arr, s2) //arr:[4]int，s2:[]int

	s3 := make([]int, 3, 8)
	fmt.Println(s3)
	fmt.Printf("容量：%d,长度：%d\n", cap(s3), len(s3))
	s3[0] = 1
	s3[1] = 2
	s3[2] = 3
	fmt.Println(s3)
	//fmt.Println(s3[3])//index out of range

	//append
	s4 := make([]int, 0, 5)
	fmt.Println(s4)
	s4 = append(s4, 1, 2)
	fmt.Println(s4)
	s4 = append(s4, 1, 2, 3, 4, 5) //自动扩容
	fmt.Println(s4)

	s4 = append(s4, s3...)
	fmt.Println(s4)

	//切片遍历
	fmt.Println("===========================")
	for i := 0; i < len(s4); i++ {
		fmt.Println(s4[i])
	}
	for i, v := range s4 {
		fmt.Printf("index:%d,value:%d\n", i, v)
	}
}
```

### slice内存分析

```golang
package main

import "fmt"

func main() {
	/*
		切片slice
			1.每个切片引用一个底层数组
			2.切片本身不存储任何数据，都是这个底层数组存储，所以修改切片也就是修改这个数组中的数据
			3.当向切片中添加数据是，如果超过容量，直接添加，如果超过容器，自动扩容（成倍增长）
			4.切片一旦扩容，就是重新指向一个新的底层数组
		cap:
			s1:3--->6--->12--->24
			s2:4--->8--->16--->32
	*/
	s1 := []int{1, 2, 3}
	fmt.Println(s1)
	fmt.Printf("长度：%d,容量：%d\n", len(s1), cap(s1))
	fmt.Printf("地址：%p\n", s1) //0xc000010340
	s1 = append(s1, 4, 5)
	fmt.Println(s1)
	fmt.Printf("长度：%d,容量：%d\n", len(s1), cap(s1))
	fmt.Printf("地址：%p\n", s1) // 0xc00000c2a0
	s1 = append(s1, 6, 7, 8)
	fmt.Println(s1)
	fmt.Printf("长度：%d,容量：%d\n", len(s1), cap(s1))
	fmt.Printf("地址：%p\n", s1) //0xc0000180c0
	s1 = append(s1, 9, 10)
	fmt.Println(s1)
	fmt.Printf("长度：%d,容量：%d\n", len(s1), cap(s1))
	fmt.Printf("地址：%p\n", s1) //0xc000040060
	s1 = append(s1, 11, 12, 13, 14, 15)
	fmt.Println(s1)
	fmt.Printf("长度：%d,容量：%d\n", len(s1), cap(s1))
	fmt.Printf("地址：%p\n", s1) //0xc00007a000
}
```

### slice操作已有底层数组

```golang
package main

import "fmt"

func main() {
	/*
		slice := arr[start:end]
			切片中的数据：[start,end]
			arr[:end],从头到end
			arr[start:],从start到末尾

		从已有的数组上，直接创建切片，该切片的底层数组就是当前数组
			长度是从start到end的数据量
			但是容量从start到数组的末尾
	*/
	a := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	fmt.Println("---------------1.已有数组直接切片-----------------")
	s1 := a[:5]     //1-5
	s2 := a[3:8]    //4-8
	s3 := a[6:]     //7-10
	s4 := a[:]      //1-10
	fmt.Println(s1) //[1 2 3 4 5]
	fmt.Println(s2) //[4 5 6 7 8]
	fmt.Println(s3) //[7 8 9 10]
	fmt.Println(s4) //[1 2 3 4 5 6 7 8 9 10]

	fmt.Println("----------------2.长度和容量-----------------")
	fmt.Printf("a地址：%p,s1地址：%p\n", &a, s1)             //a地址：0xc00005c0f0,s1地址：0xc00005c0f0
	fmt.Printf("s1,len:%d,cap:%d\n", len(s1), cap(s1)) //s1,len:5,cap:10
	fmt.Printf("s2,len:%d,cap:%d\n", len(s2), cap(s2)) //s2,len:5,cap:7
	fmt.Printf("s3,len:%d,cap:%d\n", len(s3), cap(s3)) //s3,len:4,cap:4
	fmt.Printf("s4,len:%d,cap:%d\n", len(s4), cap(s4)) //s4,len:10,cap:10

	fmt.Println("--------------3.更改数组的内容----------------")
	a[4] = 100
	fmt.Println(a)  //[1 2 3 4 100 6 7 8 9 10]
	fmt.Println(s1) //[1 2 3 4 100]
	fmt.Println(s2) //[4 100 6 7 8]
	fmt.Println(s3) //[7 8 9 10]
	fmt.Println(s4) //[1 2 3 4 100 6 7 8 9 10]

	fmt.Println("--------------4.更改切片内容 ----------------")
	s1 = append(s1, 1, 1, 1, 1)
	fmt.Println(a)  //[1 2 3 4 100 1 1 1 1 10]
	fmt.Println(s1) //[1 2 3 4 100 1 1 1 1]
	fmt.Println(s2) //[4 100 1 1 1]
	fmt.Println(s3) //[1 1 1 10]
	fmt.Println(s4) //[1 2 3 4 100 1 1 1 1 10]

	fmt.Println("--------------5.切片扩容 ----------------")
	fmt.Println(len(s1), cap(s1)) //9 10
	s1 = append(s1, 2, 2, 2, 2, 2)
	fmt.Println(a)                //[1 2 3 4 100 1 1 1 1 10]
	fmt.Println(s1)               //[1 2 3 4 100 1 1 1 1 2 2 2 2 2]
	fmt.Println(s2)               //[4 100 1 1 1]
	fmt.Println(s3)               //[1 1 1 10]
	fmt.Println(s4)               //[1 2 3 4 100 1 1 1 1 10]
	fmt.Println(len(s1), cap(s1)) //14 20
	fmt.Printf("%p\n", s1)        //0xc00008e000
	fmt.Printf("%p\n", &a)        //0xc00005c0f0
}
```

### slice是引用类型

```golang
package main

import "fmt"

func main() {
	/*
		按照类型来分：
			基本类型：int float string bool
			复合类型：array slice map struct pointer function chan
		按照特点来分：
			值类型：int float string bool array
				传递的是数据副本
			引用类型：slice
				传递的地址，多个变量指向了同一块内存地址
		所以：切片是引用类型的数据，存储了底层数组的引用
	*/
	//1.数组：值类型
	a1 := [4]int{1, 2, 3, 4}
	a2 := a1            //值传递，传递的是数据
	fmt.Println(a1, a2) //[1 2 3 4] [1 2 3 4]
	a1[0] = 100
	fmt.Println(a1, a2) //[100 2 3 4] [1 2 3 4]

	//2.切片：引用类型
	s1 := []int{1, 2, 3, 4}
	s2 := s1
	fmt.Println(s1, s2) //[1 2 3 4] [1 2 3 4]
	s1[0] = 100
	fmt.Println(s1, s2) //[100 2 3 4] [100 2 3 4]

	fmt.Printf("%p\n", s1)  //0xc0000541e0
	fmt.Printf("%p\n", s2)  //0xc0000541e0
	fmt.Printf("%p\n", &s1) //0xc00004c440
	fmt.Printf("%p\n", &s2) //0xc00004c460
}
```

### 深拷贝和浅拷贝-----copy()

```golang
package main

import "fmt"

func main() {
	/*
		深拷贝：拷贝的是数据本身
			值类型的数据，默认就是深拷贝：array int float string  bool struct
		浅拷贝：拷贝的是数据的的 地址
			导致多个变量指向同一块内存
			引用类型的数据，默认就是浅拷贝：slice map

			因为切片是引用类型的数据，直接拷贝地址
		copy(dst,src[]type)int
			可以实现切片拷贝
	*/
	s1 := []int{1, 2, 3, 4}
	s2 := make([]int, 0)
	for i := 0; i < len(s1); i++ {
		s2 = append(s2, s1[i])
	}
	fmt.Println(s1) //[1 2 3 4]
	fmt.Println(s2) //[1 2 3 4]

	s1[0] = 100
	fmt.Println(s1) //[100 2 3 4]
	fmt.Println(s2) //[1 2 3 4]

	//copy()
	s3 := []int{7, 8, 9}
	fmt.Println(s2) //[1 2 3 4]
	fmt.Println(s3) //[7 8 9]

	//copy(s2, s3)//将s3中的元素拷贝到s2中
	//fmt.Println(s2) //[7 8 9 4]
	//fmt.Println(s3) //[7 8 9]
	//copy(s3,s2)//将s2中的元素拷贝到s4中
	//fmt.Println(s2) //[1 2 3 4]
	//fmt.Println(s3) //[1 2 3]
	copy(s3[1:], s2[2:])
	fmt.Println(s2) //[1 2 3 4]
	fmt.Println(s3) //[7 3 4]
}
```

### map(python的字典)

```golang
package main

import "fmt"

func main() {
	/*
		map:映射，是一种专门用于存储键值对的集合，属于引用类型

		存储特点：
			A：存储的无序的键值对
			B：键不能重复，并且和value值一一对应
				map中的key不能重复，如果重复，那么新的value会覆盖原来的，程序不会报错
		语法结构：
			1.创建map
				var map1 map[key类型]value类型
					nil map 无法直接使用
				var map2 = make(map[key类型]value类型)
				var map3 = map[key类型]value类型{key1:value1,key2:value2,...}
			2.添加/修改
				map[key]=value
					如果key不存在则添加新数据
					如果key存在则修改value
			3.获取
				map[key]--->value
				value,ok := map[key]
					如果key存在，value就是对应的数值，ok为true
					如果key不存在，value为值类型零值，ok为false
			4.删除数据
				delete(map,key)
					如果key存在，就可以直接删除
					如果key不存在，删除失败，不会对map产生任何影响
			5.长度
				len(map)
		每种类型：
			int:0
			float:0.0--->0
			string:""
			array:[0000]
			slice:nil(python的None)
			map:nil
	*/
	//1.创建map
	var map1 map[int]string         //没有初始化，nil
	var map2 = make(map[int]string) //创建
	var map3 = map[string]int{"go": 98, "python": 87, "java": 79, "html": 93}
	fmt.Println(map1) //map[]
	fmt.Println(map2) //map[]
	fmt.Println(map3) //map[go:98 html:93 java:79 python:87]

	fmt.Println(map1 == nil) //true
	fmt.Println(map2 == nil) //false
	fmt.Println(map3 == nil) //false

	//map1[1]="hello"//panic: assignment to entry in nil map
	//2.nil map
	if map1 == nil {
		map1 = make(map[int]string)
		fmt.Println(map1 == nil) //false
	}
	//3.存储键值对到map中
	//map1[key]=value
	map1[1] = "hello"
	map1[2] = "world"
	map1[3] = "么么哒"
	map1[4] = "tom"
	map1[5] = "jerry"
	fmt.Println(map1) //map[1:hello 2:world 3:么么哒 4:tom 5:jerry]

	//4.获取数据，根据key获取对应的value值
	//根据key获取value，如果可以存在，获取数值，如果不存在，获取value类型零值
	fmt.Println(map1[4])  //tom
	fmt.Println(map1[40]) //""

	v1, ok := map1[40]
	if ok {
		fmt.Println("获取到的对应的值是：", v1)
	} else {
		fmt.Println("操作的key不存在，获取到的是零值") //操作的key不存在，获取到的是零值
	}

	//5.修改数据
	fmt.Println(map1) //map[1:hello 2:world 3:么么哒 4:tom 5:jerry]
	map1[3] = "如花"
	fmt.Println(map1) //map[1:hello 2:world 3:如花 4:tom 5:jerry]

	//6.删除数据
	delete(map1, 3)
	fmt.Println(map1) //map[1:hello 2:world 4:tom 5:jerry]
	delete(map1, 30)
	fmt.Println(map1) //map[1:hello 2:world 4:tom 5:jerry]

	//7.map的长度
	fmt.Println(len(map1)) //4
}
```

### map遍历

```golang
package main

import (
	"fmt"
	"sort"
)

func main() {
	/*
		map的遍历：
			使用 for range
				数组，切片：index,vlaue
				map:key,value
	*/
	map1 := make(map[int]string)
	map1[1] = "Tom"
	map1[2] = "Jerry"
	map1[3] = "Shelly"
	map1[4] = "Billy"
	map1[5] = "Bob"
	map1[6] = "York"

	//1.遍历map
	for k, v := range map1 {
		fmt.Printf("key:%d,value:%s\n", k, v)
	}
	for i := 1; i < len(map1); i++ {
		fmt.Println(i, "----->", map1[i])
	}
	/*
		1.获取所有的key,--->切片/数组
		2.进行排序
		3.遍历key--->map[key]
	*/
	keys := make([]int, 0, len(map1))
	fmt.Println(keys)
	for k, _ := range map1 {
		keys = append(keys, k)
	}
	fmt.Println(keys) //[2 3 4 5 6 1]
	//冒泡排序或者使用sort包
	sort.Ints(keys) //[1 2 3 4 5 6]
	fmt.Println(keys)
	for _, key := range keys {
		fmt.Println(key, map1[key])
	}

	s1 := []string{"Apple", "Unix", "Linux", "Windows"}
	fmt.Println(s1)
	sort.Strings(s1)
	fmt.Println(s1)//[Apple Linux Unix Windows]
}
```

### map结合slice使用

```golang
package main

import "fmt"

func main() {
	/*
		map和slice的结合使用
			1.创建map用于存储人的信息
				name,age,sex,address
			2.每个map存储一个人的信息
			3.将这些map存储到slice中
			4.打印遍历输出
	*/
	//1.创建map存储第一个人的信息
	map1 := make(map[string]string)
	map1["name"] = "王二狗"
	map1["age"] = "30"
	map1["sex"] = "男性"
	map1["address"] = "北京市xxx路xxx号"
	fmt.Println(map1) //map[address:北京市xxx路xxx号 age:30 name:王二狗 sex:男性]

	//2.第二个人
	map2 := make(map[string]string)
	map2["name"] = "李小花"
	map2["age"] = "29"
	map2["sex"] = "女性"
	map2["address"] = "北京市xxx路xxx号"
	fmt.Println(map2) //map[address:北京市xxx路xxx号 age:29 name:李小花 sex:女性]

	//3.
	map3 := map[string]string{"name": "York", "age": "24", "sex": "男性", "address": "成都市xxx路xxx号"}
	fmt.Println(map3) //map[address:成都市xxx路xxx号 age:24 name:York sex:男性]

	//将map存入到slice
	s1 := make([]map[string]string, 0, 3)
	s1 = append(s1, map1)
	s1 = append(s1, map2)
	s1 = append(s1, map3)

	//遍历切片
	for i, val := range s1 {
		//val:map1,map2,map3
		fmt.Printf("第%d个人的信息是:\n", i+1)
		fmt.Printf("\t姓名：%s\n", val["name"])
		fmt.Printf("\t年龄：%s\n", val["age"])
		fmt.Printf("\t性别：%s\n", val["sex"])
		fmt.Printf("\t地址：%s\n", val["address"])
	}
}
```

### map是引用类型数据

```golang
package main

import "fmt"

func main() {
	/*
		一、数据类型
			基本数据类型：int float string bool
			复合数据类型：array slice map function pointer struct...
				array:[size]数据类型
				slice:[]数据类型
				map:map[key类型]value类型
		二、存储特点
			值类型：int float string bool array struct
			引用类型：slice map
				make():slice map chan(全都是引用类型的数据)
	*/
	map1 := make(map[int]string)
	map2 := make(map[string]float64)
	fmt.Printf("%T\n", map1) //map[int]string
	fmt.Printf("%T\n", map2) //map[string]float64

	map3 := make(map[string]map[string]string)
	m1 := make(map[string]string)
	m1["name"] = "王二狗"
	m1["age"] = "30"
	m1["sex"] = "男"
	m1["address"] = "北京市"
	map3["hr"] = m1

	m2 := make(map[string]string)
	m2["name"] = "York"
	m2["age"] = "24"
	m2["sex"] = "男"
	m2["address"] = "成都市"
	map3["技术"] = m2
	fmt.Println(map3) //map[hr:map[address:北京市 age:30 name:王二狗 sex:男] 技术:map[address:成都市 age:24 name:York sex:男]]

	fmt.Println("----------------------------")
	map4 := make(map[string]string)
	map4["Tom"] = "猫"
	map4["Jerry"] = "鼠"
	fmt.Println(map4) //map[Jerry:鼠 Tom:猫]

	map5 := map4
	fmt.Println(map5) //map[Jerry:鼠 Tom:猫]

	map5["Tom"] = "tom猫"
	fmt.Println(map4) //map[Jerry:鼠 Tom:tom猫]
	fmt.Println(map5) //map[Jerry:鼠 Tom:tom猫]
}
```

### 字符串的使用

```golang
package main

import "fmt"

func main() {
	/*
		go中的字符串是一个字节的切片
			可以通过将其内容封装在""中来创建字符串，go中的字符串是unicode兼容的，并且是utf-8编码的
		字符串是一个些字节的集合
		语法："" "a" "bd" "中"
		字符：--->对应编码表中的编码值
			A--->65
			B--->66
			a--->97
			...
		字节：byte--->uint8
			utf8
	*/
	//1.定义字符串
	s1 := "hello中文"
	s2 := "hello world"
	fmt.Println(s1)
	fmt.Println(s2)

	//2.字符串的长度：返回的是字节的个数
	fmt.Println(len(s1)) //11 一个中文字符占三个字节
	fmt.Println(len(s2)) //11

	//3.获取某个字节
	fmt.Println(s2[0]) //获取字符串中的第一个字节

	a := 'h'
	b := 104
	fmt.Printf("%c,%c,%c\n", s2[0], a, b) //h,h,h

	//4.字符串的遍历
	for i := 0; i < len(s2); i++ {
		//fmt.Println(s2[i])
		fmt.Printf("%c\t", s2[i]) //h	e	l	l	o	 	w	o	r	l	d
	}
	//for range
	for _, v := range s2 {
		//fmt.Println(i,v)
		fmt.Printf("%c", v)
	}
	fmt.Println()

	//5.字符串是字节的集合
	slice1 := []byte{65, 66, 67, 68}
	s3 := string(slice1)
	fmt.Println(s3) //ABCD

	s4 := "abcdef"
	slice2 := []byte(s4) //根据字符串获取对应的字节切片
	fmt.Println(slice2)  //[97 98 99 100 101 102]

	//6.字符串的不能修改
	fmt.Println(s4)
	//s4[2] = "B"//cannot assign to s4[2]

}
```

### strings包的使用

```golang
package main

import (
	"fmt"
	"strings"
)

func main() {
	/*
		strings包下的关于字符串的函数
	*/
	s1 := "helloworld"
	//1.是否包含指定的内容--->bool
	fmt.Println(strings.Contains(s1, "h")) //true
	//2.是否包含chars中任意一个字符
	fmt.Println(strings.ContainsAny(s1, "abcd")) //true
	//3.统计substr在s1中出现的次数
	fmt.Println(strings.Count(s1, "ll")) //1

	//4.判断字符串是否以指定字符串开头或者结尾
	s2 := "20190525课堂笔记.txt"
	if strings.HasPrefix(s2, "2019") {
		fmt.Println("19年的文件")
	}
	if strings.HasSuffix(s2, ".txt") {
		fmt.Println("这是一个文本文档")
	}

	//索引
	fmt.Println(strings.Index(s1, "l"))       //查账substr在s中的位置，如果不存在返回-1
	fmt.Println(strings.IndexAny(s1, "abcd")) //9 substr任意一个字符在str中的位置
	fmt.Println(strings.LastIndex(s1, "l"))   //查找substr最后出现在str中的位置

	//字符串的拼接
	ss1 := []string{"abc", "world", "haha"}
	s3 := strings.Join(ss1, "*") //以sep拼接切片
	fmt.Println(s3)              //abc*world*haha
	ss2 := strings.Split(s3, "*")
	fmt.Println(ss2) //[abc world haha]

	//重复，自己拼接自己
	s4 := strings.Repeat("hello", 5)
	fmt.Println(s4) //hellohellohellohellohello

	//替换
	s5 := strings.Replace(s1, "l", "*", 1) //he*loworld
	fmt.Println(s5)

	s6 := "helloWORLD **123"
	fmt.Println(strings.ToLower(s6)) //helloworld **123
	fmt.Println(strings.ToUpper(s6)) //HELLOWORLD **123

	//substring(start,end)-->substr
	//str[start:end]
	fmt.Println(s1) //helloworld
	s7 := s1[0:5]
	fmt.Println(s7) //hello
}
```

### strconv（字符串与其他类型之间的转换）

```golang
package main

import (
	"fmt"
	"strconv"
)

func main() {
	/*
		strconv包：字符串和基本类型之间的转换
			string convert
	*/
	//fmt.Println("aa"+100)//cannot convert "aa" (type untyped string) to type int
	//1.bool类型
	s1 := "true"
	b1, err := strconv.ParseBool(s1)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("%T,%t\n", b1, b1) //bool,true

	ss1 := strconv.FormatBool(b1)
	fmt.Printf("%T,%s\n", ss1, ss1) //string,true

	//2.整数、
	s2 := "100"
	i2, err := strconv.ParseInt(s2, 10, 64)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Printf("%T,%d\n", i2, i2) //int64,100

	ss2 := strconv.FormatInt(i2, 10)
	fmt.Printf("%T,%s\n", ss2, ss2) //string,100

	//itoa(),atoi()
	i3,err := strconv.Atoi("-42")//转为int类型
	fmt.Printf("%T,%d\n", i3, i3)//int,-42
	ss3:=strconv.Itoa(-42)
	fmt.Printf("%T,%s\n", ss3, ss3)//string,-42
}
```

### 函数

```golang
package main

import "fmt"

func main() { //程序的主函数
	/*
		函数：function
		一、概念：
			具有特定功能的代码，可以被多次调用执行
		二、意义：
			1.避免重复代码
			2.增强代码的扩展性
		三、使用：
			step1:函数的定义，也叫申明
			step2:函数的调用，就是执行函数中代码的过程
		四、语法：
			1.定义函数的语法：
				func funcName(params){
					//代码处理逻辑
					return value1,value2
				}
				A:func，定义函数的关键字
				B:funcName,函数的名字
				C:(),函数的标志
				D:参数列表：形式参数用于接收外部传入的函数的数据
				E:返回值列表：函数执行后返回给调用者的结果
			2.调用函数的语法
				函数名（实际参数）
			3.注意事项：
				A:函数必须定义后才能使用
					定义了没调用，这个函数没有意义
				B:函数名不能冲突
				C:main(),是一个特殊的函数，作为程序的入口，由系统自动调用
		函数的调用处，就是函数调用的位置
		函数参数分类：
			形式参数：函数定义的时候，用于接收外部传入的参数，函数中某些变量不确定，则定义为形参
			实际参数：函数调用的时候给形参实际赋予的数据
		函数的调用：
			1.函数名：申明函数和调用函数名要统一
			2.实参必须严格匹配形参，顺序，个数，类型，一一对应
	*/
	getSum(10)
	getSum(20)
	getSum(100)

	//求两个整数的和

	getAdd(100, 100)
	getAdd2(100, 100)
	fun1(3.124, 5.1354, "haha")
}

//定义一个函数：用于求1-10的和
func getSum(n int) {
	sum := 0
	for i := 1; i <= n; i++ {
		sum += i
	}
	fmt.Printf("%d\n", sum)
}

func getAdd(a int, b int) {
	sum := a + b
	fmt.Printf("%d+%d=%d\n", a, b, sum)
}

func getAdd2(a, b int) {
	fmt.Printf("a:%d,b:%d", a, b)
}

func fun1(a, b float64, c string) {
	fmt.Printf("a:%.2f,b:%.2f,c:%s", a, b, c)
}
package main

import "fmt"

func main() { //程序的主函数
	/*
		函数：function
		一、概念：
			具有特定功能的代码，可以被多次调用执行
		二、意义：
			1.避免重复代码
			2.增强代码的扩展性
		三、使用：
			step1:函数的定义，也叫申明
			step2:函数的调用，就是执行函数中代码的过程
		四、语法：
			1.定义函数的语法：
				func funcName(params){
					//代码处理逻辑
					return value1,value2
				}
				A:func，定义函数的关键字
				B:funcName,函数的名字
				C:(),函数的标志
				D:参数列表：形式参数用于接收外部传入的函数的数据
				E:返回值列表：函数执行后返回给调用者的结果
			2.调用函数的语法
				函数名（实际参数）
			函数的调用处，就是函数调用的位置
	*/
	getSum()

}

//定义一个函数：用于求1-10的和
func getSum() {
	sum := 0
	for i := 1; i <= 10; i++ {
		sum += i
	}
	fmt.Printf("1-10的和是:%d", sum)
}
```

### 可变参数

```golang
package main

import "fmt"

func main() {
	/*
		可变参数：
			概念：一个函数的参数的类型确定，但是个数不确定，就可以使用可变参数
			语法：
				func myfunc(arg ... int){}
				可以传入多个参数，也可以传入切片
				Println() Print() Printf()
				append()
			注意事项：
				1.如果一个函数的参数是可变参数，同时还有其他参数，可变参数要放在参数列表的最后
				2.一个函数中最多只能有一个可变参数
	*/
	//1.求和
	getChangeSum(1, 2, 3, 4)
	//2.切片
	s1 := []int{1,2,3,4,5}
	getChangeSum(s1...)
}

func getChangeSum(nums ...int) {
	fmt.Printf("%T\n", nums) //[]int
	sum := 0
	for i := 0; i < len(nums); i++ {
		sum += nums[i]
	}
	fmt.Println("总和是：", sum)
}

func hell0(s1,s2 string,nums ...float64){

}
```

### 函数中return语句的使用

```golang
package main

import "fmt"

func main() {
	/*
		函数的返回值：
			一个函数的执行结果，反悔费函数的调用处，执行结果就叫做函数的返回值
		return语句：将结果返回给函数的调用处
			函数返回的结果，必须和函数定义的一致：类型，个数，顺序
			1.将函数执行结果返回
			2.同时结束了该函数的执行
		空白标志符，专门用于舍弃数据：_
	*/

	sum := getSum1(99)
	fmt.Printf("%d\n", sum)

	fmt.Println(getSum2(10))

	res1, res2 := rectangle(10, 20)
	fmt.Printf("周长是：%.2f,面积是：%.2f\n", res1, res2)

	res3, res4 := rectangle(50, 20)
	fmt.Printf("周长是：%.2f,面积是：%.2f\n", res3, res4)

	res5, _ := rectangle(10, 1)
	fmt.Printf("周长：%.f\n", res5)
}

func getSum1(n int) (int) {
	sum := 0
	for i := 1; i <= n; i++ {
		sum += i
	}
	return sum
}

func getSum2(n int) (s int) {
	for i := 1; i <= n; i++ {
		s += i
	}
	return
}

//定义一个函数，用于求矩形的周长后面积
func rectangle(len, wid float64) (float64, float64) {
	perimeter := (len + wid) * 2
	area := len * wid
	return perimeter, area
}

func rectangle2(len, wid float64) (peri float64, area float64) {
	peri = (len + wid) * 2
	area = len * wid
	return
}

func check() (float64, float64, string, int) {
	return 1.0, 2.3, "haha", 10
}
```

### return语句的返回使用

```golang
package main

import "fmt"

func main() {
	/*
		return语句：词义“返回”
			1.一个函数有返回值，那么使用return将返回值返回给调用处
			2.同时意味着结束了函数的执行
		注意点：
			1.一个函数定义了返回值，必须使用return语句将结果返回给调用处，return后的数据必须和函数定义的一致：个数，类型，顺序
			2.可以使用_来舍弃多余的返回值
			3.如果一个函数定义了有返回值，那么函数有分支，无论那个分支，都必须有return
			4.如果一个函数没有定义返回值，那么也可以用return,专门用于结束函数的执行
	*/
	a, b, c := fun3()
	fmt.Println(a, b, c)
	_, _, d := fun3()
	fmt.Println(d)

	fmt.Println(fun4(-30))

	fun5()
}

func fun3() (int, float64, string) {
	return 10, 39.3, "lala"
}

func fun4(age int) int {
	if age >= 0 {
		return age
	} else {
		fmt.Println("年龄不能为负数")
		return 0
	}
}

func fun5() {
	for i := 0; i < 10; i++ {
		if i == 5 {
			return
		}
		fmt.Println(i)
	}
	fmt.Println("func5...over...")
}
```

### 函数作用域（全局变量和局部变量）

```golang
package main

import "fmt"

//全局变量的定义
//num3 := 1000//不支持简短定义的写法
var num3 = 1000

func main() {
	/*
		作用域：变量可以使用的范围
			局部变量：函数内部定义的变量，就叫做局部变量
				在哪里定义只能在哪里使用
			全局变量：函数外部定义的变量，+就叫做全局变量
				所有函数都可以使用，而且共享这一份数据
	*/
	//定义在main函数中，所以n的作用域就是在main函数的范围内
	n := 10
	fmt.Println(n)
	if a := 1; a <= 10 {
		fmt.Println(a)
		fmt.Println(n)
	}
	//fmt.Println(a)//undefined: a 不能访问，出了作用域
	fmt.Println(n)

	if b := 1; b <= 10 {
		n := 20
		fmt.Println(b) //1
		fmt.Println(n) //20
	}
	fmt.Println(n) //10

	fmt.Println(num3)
	num3 = 2000
	fmt.Println(num3)
	fun1()
}

func fun1() {
	//fmt.Println(n)
	num1 := 100
	fmt.Println(num1)

	fmt.Println(num3)
}
```

### 递归函数

```golang
package main

import "fmt"

func main() {
	/*
		递归函数（recursion）:一个函数自己调自己，就叫做递归函数。
			递归函数要有一个出口，逐渐向出口靠近
	*/
	//1.求1-5的和
	fmt.Println(getSum(10))

	//2.fibonacci数列
	fmt.Println(fibonacci(10))

}

func getSum(n int) (int) {
	if n == 1 {
		return 1
	} else {
		return getSum(n-1) + n
	}
}

func fibonacci(n int) (int) {
	/*
		1 2 3 4 5
		1 1 2 3 5
	*/
	if n == 1 || n == 2 {
		return 1
	} else {
		return fibonacci(n-1) + fibonacci(n-2)
	}
}
```

### defer的使用

```golang
package main

import "fmt"

func main() {
	/*
		defer的词义：“延迟，推迟”
		在go语言中，使用defer关键字来延迟一个函数或者方法的执行
			1.defer函数或方法：一个函数或方法的执行被延迟了
			2.defer的用法：
				1.对象.close(),临时文件的删除...
					文件.open()
					defer close()
						读或写
				2.go语言中关于异常的处理，使用panic()和recover()
					panic函数用于引发恐慌，导致程序中断执行
					recover函数用于恢复程序的执行，recover()语法上要求必须在defer中执行
		3.如果多个defer函数(栈：先进后出)：先延迟的后执行
		4.defer函数传递参数的时候:defer函数调用时，就已经传递了参数的数据，只是暂时不执行函数的代码而已
		5.defer函数注意点
			有return语句的时候，先defer,后return
	*/
	//defer fun1("hello")
	//fmt.Println("12345")
	//defer fun1("world")
	//fmt.Println("Tom")

	a := 2
	fmt.Println(a) //2
	defer fun2(a)  //2
	a++
	fmt.Println("main", a) //3

	fmt.Println(fun3())

}

func fun2(a int) {
	fmt.Println("fun2", a)
}

func fun3() int {
	fmt.Println("fun3")
	defer fun1("haha")
	return 0
}

func fun1(s string) {
	fmt.Println(s)
}
```

### 函数类型

```golang
package main

import "fmt"

func main() {
	/*
		go语言的数据类型：
			基本数据类型：
				int float bool string
			复合类型：
				array slice map function pointer struct interface...
		函数的类型：
			func(string, string, int, int) (string, int, float64)
			func(参数类型,...)(返回值类型,...)
	*/
	a := 10
	fmt.Printf("%T\n", a) //int
	b := [4]int{1, 2, 3, 4}
	fmt.Printf("%T\n", b) //[4]int

	fmt.Printf("%T\n", fun1) //func()
	fmt.Printf("%T\n", fun2) //func(int) int
	fmt.Printf("%T\n", fun3) //func(float64, string) (int, int)
	fmt.Printf("%T\n", fun4) //func(string, string, int, int) (string, int, float64)
}
func fun1() {}
func fun2(a int) int {
	return 0
}
func fun3(a float64, b string) (int, int) {
	return 0, 0
}
func fun4(a, b string, c, d int) (string, int, float64) {
	return "haha", 0, 1.1
}
```

### 函数的本质

```golang
package main

import "fmt"

func main() {
	/*
		go语言的数据类型：
			数值类型：整数 浮点
				进行运算操作，加减乘除，打印
			字符串
				可以获取单个字符，截取，遍历，string包下的函数操作
			数组 切片 map
				存储数据，修改数据，获取数据，遍历数据
			函数

		注意点：
			函数作为一种复合数据类型，可以看作是一种特殊的变量
				函数名()：将函数进行调用，函数中的代码会全部执行，然后返回return
				函数名：指向函数体的内存地址
	*/

	//1.函数作为一个变量
	fmt.Printf("%T\n", fun1) //func(int,int)
	fmt.Println(fun1)        //0x492e20

	//2.直接定义一个函数类型的变量
	var c func(int, int)
	fmt.Println(c) //<nil>

	c = fun1       //将fun1地址赋值给c
	fmt.Println(c) //0x492ef0
	fun1(10, 20)   //10 20
	c(100, 200)    //100 200

	res1 := fun2            //将fun2地址赋值给res1
	res2 := fun2(1, 3)      //将fun2的返回值赋值给res2
	fmt.Println(res1, res2) //0x493050 4

}
func fun2(a, b int) int {
	return a + b
}

func fun1(a, b int) {
	fmt.Println(a, b)
}
```

### 匿名函数

```golang
package main

import "fmt"

func main() {
	/*
		匿名：没有名字
			匿名函数：没有名字的函数
		定义一个匿名函数，直接进行调用，通常只能使用一次，也可以使用匿名函数赋值给某个函数，那么就可以调用多次了
		匿名函数：
			go语言支持函数式编程
				1.将匿名函数作为另一个函数的参数，回调函数
				2.将匿名函数作为另一个函数的返回值，可以形成闭包函数
	*/
	fun1()
	fun2 := fun1
	fun2()

	//匿名函数
	func() {
		fmt.Println("匿名函数1")
	}() //匿名函数1

	fun3 := func() {
		fmt.Println("匿名函数2")
	}
	fun3() //匿名函数2 可以调用多次

	//定义带参数的匿名函数
	func(a, b int) {
		fmt.Println(a, b)
	}(1, 2) //1 2

	//定义带返回值的匿名函数
	res1 := func(a, b int) int {
		return a + b
	}(10, 20)               //将执行结果赋值给res1
	fmt.Println(res1)       //30
	res2 := func(a, b int) int {
		return a + b
	}                       //将地址给res2
	fmt.Println(res2)       //0x491520
	fmt.Println(res2(1, 3)) //4
}
func fun1() {
	fmt.Println("fun1")
}
```

### 回调函数

```golang
package main

import "fmt"

func main() {
	/*
		高阶函数：
			根据go语言的数据类型的特点，可以将一个函数是作为另一个函数的参数
		fun1(),fun2()
		将fun1函数作为了fun2这个函数的参数
			fun2函数：就叫高阶函数
				接受另一个函数作为参数的函数，高阶函数
			fun1函数：回调函数
				作为另一个函数的参数的函数，就叫做回调函数
	*/
	fmt.Printf("%T\n", add)  //func(int, int) int
	fmt.Printf("%T\n", oper) //func(int, int, func(int, int) int) int

	res1 := add(1, 2)
	fmt.Println(res1) //3

	res2 := oper(1, 2, add)
	fmt.Println(res2) //3

	res3 := oper(1, 2, sub)
	fmt.Println(res3) //-1

	fun1 := func(a, b int) int {
		return a * b
	}
	res4 := oper(2, 3, fun1)
	fmt.Println(res4) //6

	res5 := oper(100, 8, func(a, b int) int {
		if b == 0 {
			fmt.Println("除数不能为0")
			return 0
		}
		return a / b
	})
	fmt.Println(res5) //12
}

func add(a, b int) int {
	return a + b
}

func sub(a, b int) int {
	return a - b
}

func oper(a, b int, fun func(int, int) int) int {
	fmt.Println(a, b, fun) //打印三个参数
	res := fun(a, b)
	return res
}
```

### 函数闭包

```golang
package main

import "fmt"

func main() {
	/*
		go语言支持函数式编程
			支持将一个函数作为另一个函数的参数
			也支持将一个函数作为另一个函数的返回值
		闭包(closure):
			一个外层函数当中，有内层函数，该内层函数中，会操作外层函数的局部变量，并且该外层函数返回值就是这个内层函数
			这个内层函数和外层函数的局部变量，统称为闭包结构
			局部变量的生命周期会发生改变，正常的变量随着函数调用而创建，随着函数结束而销毁
			但是闭包结构中的外层函数的局部变量并不会随着外层函数的结束而销毁，因为内层函数还要继续使用

	*/
	res1 := increment()
	fmt.Printf("%T\n", res1) //func() int
	v1 := res1()
	fmt.Println(v1) //1
	v2 := res1()
	fmt.Println(v2) //2

	res2 := increment()
	fmt.Println(res2())
	fmt.Println(res2())
}

func increment() func() int {
	//1.定义了一个局部变量
	i := 0
	//2.定义了一个匿名函数
	fun := func() int {
		i++
		return i
	}
	//3.返回匿名函数
	return fun
}
```

## 指针 and 指针的指针

```golang
package main

import "fmt"

func main() {
	/*
		指针：pointer
			存储了另一个变量的内存地址的变量

	*/
	//1.定义一个int类型的变量
	a := 10
	fmt.Println("a的值是：", a)      //a的值是： 10
	fmt.Printf("%T\n", a)        //int
	fmt.Printf("a的地址是:%p\n", &a) //a的地址是:0xc00000a0a8

	//2.创建一个指针变量，用于存储a的地址
	var p1 *int
	fmt.Println(p1) //<nil> 空指针
	p1 = &a
	fmt.Println(p1)                 //0xc00000a0a8
	fmt.Printf("p1自己的地址:%p\n", &p1) //p1自己的地址:0xc000006030

	fmt.Println("p1的数值，是a的地址，改地址存储的数据：", *p1) //p1的数值，是a的地址，改地址存储的数据： 10

	//3.操作变量，更改数值
	a = 100
	fmt.Println("a:", a)            //100
	fmt.Printf("a的地址是:%p\n", &a)    //a的地址是:0xc00000a0a8
	fmt.Println("*p1：", *p1)        //*p1： 100
	fmt.Printf("p1自己的地址:%p\n", &p1) //p1自己的地址:0xc000006030

	//4.通过指针，改变变量的数值
	*p1 = 200
	fmt.Println(a) //200

	//5.指针的指针
	var p2 **int
	fmt.Println(p2) //<nil>
	p2 = &p1
	fmt.Println(p2)                     //0xc000082020
	fmt.Printf("%T,%T,%T\n", a, p1, p2) //int,*int,**int
	fmt.Println(p2)                     //0xc000006030(就是p1的地址)
	fmt.Printf("p2地址：%p\n", &p2)        //p2地址：0xc000082028
	fmt.Println("*p2:", *p2)            //*p2: 0xc00000a0a8
	fmt.Println("**p2:", **p2)          //**p2: 200
}
```

## 指针数组和数组指针

```golang
package main

import "fmt"

func main() {
	/*
		数组指针：首先是一个指针，一个数组的地址
			*[4]Type

		指针数组：首先是一个数组，村粗的数据类型是指针
			[4]*Type

		*[5]float64：5个浮点类型数组的指针---->指针
		*[3]string：3个字符串数组的指针----->指针
		[3]*string：一个数组中放了三个字符串指针----->数组
		[5]*float64：一个数组中放了5个浮点指针----->数组
		*[5]5*float64：指向一个指针数组的指针----->指针
		*[3]*string：指向一个指针数组的指针----->指针
		**[4]string：指向一个数组的指针的指针----->指针
		**[4]*string：指向一个数组指针的指针的指针----->指针
	*/
	//1.创建一个普通的数组
	arr1 := [4]int{1, 2, 3, 4}
	fmt.Println(arr1) //[1 2 3 4]

	//2.创建一个指针，存储该数组的地址--->数组指针
	var p1 *[4]int
	p1 = &arr1
	fmt.Println(p1)         //&[1 2 3 4]
	fmt.Printf("%p\n", p1)  //0xc000054140  数组arr1的地址
	fmt.Printf("%p\n", &p1) //0xc000082020 指针自己的地址

	//3.根据数组指针，操作数组
	(*p1)[0] = 100
	fmt.Println(arr1) //[100 2 3 4]

	p1[1] = 200       //简化写法
	fmt.Println(arr1) //[100 200 3 4]

	//4.指针数组
	a := 1
	b := 2
	c := 3
	d := 4
	arr2 := [4]int{a, b, c, d}
	arr3 := [4]*int{&a, &b, &c, &d}
	fmt.Println(arr2) //[1 2 3 4]
	fmt.Println(arr3) //[0xc0000560c8 0xc0000560d0 0xc0000560d8 0xc0000560e0]
	arr2[0] = 100
	fmt.Println(arr2) //[100 2 3 4]
	fmt.Println(a)    //1
	*arr3[1] = 200
	fmt.Println(arr3) //[0xc00000a100 0xc00000a108 0xc00000a110 0xc00000a118]
	fmt.Println(b)    //200

	c = 1000
	fmt.Println(arr2)     //[100 2 3 4]
	fmt.Println(*arr3[2]) //1000
	for i := 0; i < len(arr3); i++ {
		fmt.Println(*arr3[i])
	}
}
```

## 函数指针和指针函数

```golang
package main

import "fmt"

func main() {
	/*
		函数指针：一个指针，指向一个函数的指针
			因为go语言中，function,默认看作一个指针，没有*
			slice map function
		指针函数：一个函数，该返回值是一个指针
	*/
	var a func()
	a = fun1
	a() //fun1......

	arr1 := fun2()
	fmt.Printf("arr1的类型:%T,地址：%p,数值:%v\n", arr1, &arr1, arr1) //arr1的类型:[4]int,地址：0xc000054140,数值:[1 2 3 4]

	arr2 := fun3()
	fmt.Printf("arr2的类型:%T,地址：%p,数值:%v\n", arr2, &arr2, arr2) //arr2的类型:*[4]int,地址：0xc000006030,数值:&[5 6 7 8]
	fmt.Printf("arr2中存储数值的地址：%p\n", arr2)                     //arr2中存储数值的地址：0xc0000103c0
}

func fun1() {
	fmt.Println("fun1......")
}

func fun2() [4]int { //普通函数
	arr := [4]int{1, 2, 3, 4} //arr的地址:0xc0000103c0
	return arr
}

func fun3() *[4]int {
	arr := [4]int{5, 6, 7, 8}
	fmt.Printf("arr的地址:%p\n", &arr) //0xc0000541c0
	return &arr
}

```

## 指针作为函数的参数

```golang
package main

import "fmt"

func main() {
	/*
		指针作为参数：
		参数的传递：值传递，引用传递

	*/
	a := 10
	fmt.Println("函数调用前，a的值:", a) //函数调用前，a的值: 10
	fun1(a)
	fmt.Println("函数调用后，a的值:", a) //函数调用后，a的值: 10

	fun2(&a)
	fmt.Println("fun2()函数调用后，a:", a) //fun2()函数调用后，a: 200

	arr1 := [4]int{1, 2, 3, 4}
	fmt.Println("fun3()函数调用前：", arr1) //fun3()函数调用前： [1 2 3 4]
	fun3(arr1)
	fmt.Println("fun3()函数调用后：", arr1) //fun3()函数调用后： [1 2 3 4]

	fun4(&arr1)
	fmt.Println("fun4()函数调用后：", arr1) //fun4()函数调用后： [200 2 3 4]
}

func fun1(num int) { //值传递 num = a = 10
	fmt.Println("fun1函数中，num的值：", num) //fun1函数中，num的值： 10
	num = 100
	fmt.Println("fun1()函数中修改num:", num) //fun1()函数中修改num: 100
}

func fun2(p1 *int) { //传递的是a的地址，引用传递
	fmt.Println("fun2()函数中，p1:", *p1) //fun2()函数中，p1: 10
	*p1 = 200
	fmt.Println("fun2函数中修改后的p1:", *p1) //fun2函数中修改后的p1: 200
}

func fun3(arr2 [4]int) { //值传递
	fmt.Println("fun3()函数中的数组的：", arr2) //fun3()函数中的数组的： [1 2 3 4]
	arr2[0] = 100
	fmt.Println("fun3()函数中修改数组:", arr2) //fun3()函数中修改数组: [100 2 3 4]
}

func fun4(p2 *[4]int) { //引用传递
	fmt.Println("fun3()函数中的数组的：", *p2) //fun3()函数中的数组的： [1 2 3 4]
	p2[0] = 200
	fmt.Println("fun3()函数中修改数组:", *p2) //fun3()函数中修改数组: [200 2 3 4]
}

```

### 结构体（相当于python的类）

```golang
package main

import "fmt"

func main() {
	/*
		结构体：是由一系列具有相同类型或不同类型的数据构成的数据结合
			结构体成员是由一系列的成员变量构成，这些成员变量也称为“字段”
	*/
	//1.方法一
	var p1 Person
	fmt.Println(p1) //{ 0  }
	p1.name = "王二狗"
	p1.age = 23
	p1.sex = "男"
	p1.address = "成都市"
	fmt.Printf("姓名：%s,年龄：%d,性别：%s,地址：%s\n", p1.name, p1.age, p1.sex, p1.address) //姓名：王二狗,年龄：23,性别：男,地址：成都市

	//2.方法二
	p2 := Person{}
	p2.name = "Ruby"
	p2.age = 28
	p2.sex = "男"
	p2.address = "北京市"
	fmt.Printf("姓名：%s,年龄：%d,性别：%s,地址：%s\n", p2.name, p2.age, p2.sex, p2.address) //姓名：Ruby,年龄：28,性别：男,地址：北京市

	//3.方法三
	p3 := Person{name: "如花", age: 20, sex: "女", address: "北京市"}
	fmt.Println(p3) //{如花 20 女 北京市}
	p4 := Person{
		name:    "老王",
		age:     20,
		sex:     "男",
		address: "北京市",
	}
	fmt.Println(p4) //{老王 20 男 北京市}

	//4.方法四
	p5 := Person{"李小华", 20, "女", "北极光"}
	fmt.Println(p5) //{李小华 20 女 北极光}
}

//1.定义结构体
type Person struct {
	name    string
	age     int
	sex     string
	address string
}

```

### 结构体指针

```golang
package main

import "fmt"

func main() {
	/*
		数据类型：
			值类型：int float bool string array
			引用类型：slice map function pointer
		通过指针：
			new()创建任意类型的指针,并不是nil，空指针
				指向了新分配的类型的内存空间，里面存储零值
	*/
	//1.结构体是值类型
	p1 := Person{"York", 23, "男", "beijing"}
	fmt.Println(p1)                //{York 23 男 beijing}
	fmt.Printf("%p,%T\n", &p1, p1) //0xc0000520c0,main.Person

	p2 := p1
	fmt.Println(p2)                //{York 23 男 beijing}
	fmt.Printf("%p,%T\n", &p2, p2) //0xc0000521c0,main.Person
	p2.name = "Tom"
	fmt.Println(p2) //{Tom 23 男 beijing}
	fmt.Println(p1) //{York 23 男 beijing}

	//2.定义结构体指针
	var pp1 *Person
	pp1 = &p1
	fmt.Println(pp1)                 //&{York 23 男 beijing}
	fmt.Printf("%p,%T\n", &pp1, pp1) //0xc000006030,*main.Person
	fmt.Println(*pp1)                //{York 23 男 beijing}

	(*pp1).name = "Jerry"
	//pp1.name = "jerry"
	fmt.Println(*pp1) //{Jerry 23 男 beijing}
	fmt.Println(p1)   //{Jerry 23 男 beijing}

	//3.使用内置函数new(),go语言中专门用于创建某种类型的指针的函数
	pp2 := new(Person)
	fmt.Printf("%T\n", pp2) //*main.Person
	fmt.Println(pp2)        //&{ 0  }
	pp2.name = "Jerry"
	pp2.age = 20
	pp2.sex = "男"
	pp2.address = "北京"
	fmt.Println(*pp2) //{Jerry 20 男 北京}

	pp3 := new(int)
	fmt.Println(pp3)  //0xc000056088
	fmt.Println(*pp3) //0

}
type Person struct {
	name    string
	age     int
	sex     string
	address string
}
```

### 匿名结构体和结构体匿名字段

```golang
package main

import "fmt"

func main() {
	/*
		匿名结构体和匿名字段：
		匿名结构体：没有名字的结构体
			在创建匿名结构体时，同时创建对象
			变量名 := struct{
				字段field
			}{
				字段赋值
			}
		匿名字段：一个结构体的字段没有字段名
		匿名函数：
	*/
	s1 := Student{"张三", 18}
	fmt.Println(s1.name, s1.age) //张三 18

	func() {
		fmt.Println("hello world")
	}()

	s2 := struct {
		name string
		age  int
	}{
		name: "李四",
		age:  20,
	}
	fmt.Println(s2.name, s2.age) //李四 20

	//w1 := Worker{name:"王二狗",age:30}
	//fmt.Println(w1.name,w1.age)
	w2 := Worker{"李小花", 32}
	fmt.Println(w2)                //{李小花 32}
	fmt.Println(w2.string, w2.int) //李小花 32
}

type Worker struct {
	//name string
	//age int
	string
	int //匿名字段类型不能冲突，否则报错
}

type Student struct {
	name string
	age  int
}
```

### 结构体嵌套

```golang
package main

import "fmt"

func main() {
	/*
		结构体嵌套：一个结构体中的字段，是另一个结构体类型

	*/
	b1 := Book{}
	b1.bookName = "西游记"
	b1.price = 45.8

	s1 := Student{}
	s1.name = "王二狗"
	s1.age = 18
	s1.book = b1 //值传递
	fmt.Println(b1)
	fmt.Println(s1)
	fmt.Printf("学生姓名:%s,学生年龄：%d,看的书是：《%s》,书的价格是：%.2f\n", s1.name, s1.age, s1.book.bookName, s1.book.price)
	//学生姓名:王二狗,学生年龄：18,看的书是：《西游记》,书的价格是：45.80
	s1.book.bookName = "红楼梦"
	fmt.Println(s1) //{王二狗 18 {红楼梦 45.8}}
	fmt.Println(b1) //{西游记 45.8}

	s2 := Student{name: "李小花", age: 19, book: Book{bookName: "Go语言", price: 89.7}}
	fmt.Println(s2.name, s2.age)                       //李小花 19
	fmt.Println("\t", s2.book.bookName, s2.book.price) //Go语言 89.7

	s3 := Student{
		name: "Jerry",
		age:  17,
		book: Book{
			bookName: "十万个为啥",
			price:    33.3,
		},
	}
	fmt.Printf("学生姓名:%s,学生年龄：%d,看的书是：《%s》,书的价格是：%.2f\n", s3.name, s3.age, s3.book.bookName, s3.book.price)
	//学生姓名:Jerry,学生年龄：17,看的书是：《十万个为啥》,书的价格是：33.3

	b4 := Book{bookName: "呼啸山庄", price: 76.9}
	s4 := Student2{name: "Ruby", age: 18, book: &b4}
	fmt.Println(b4) //{呼啸山庄 76.9}
	fmt.Println(s4) //{Ruby 18 0xc00004c480}

	s4.book.bookName = "雾都孤儿"
	fmt.Println(s4.book.bookName) //雾都孤儿
	fmt.Println(b4.bookName)      //雾都孤儿
}
//1.定义一个书的结构体
type Book struct {
	bookName string
	price    float64
}
//2.定义一个学生的结构体
type Student struct {
	name string
	age  int
	book Book
}
type Student2 struct {
	name string
	age  int
	book *Book //book的地址
}
```

### go语言中的OOP

```golang
package main

import "fmt"

func main() {
	/*
		面向对象：OOP
		go语言的结构体继承：
			1.模拟继承性：is -a
				type A struct{
					field
				}
				type B struct{
					A//匿名字段
				}
			2.模拟聚合关系：has - a
				type C Struct{
					field
				}
				type D struct{
					c C//聚合关系
				}
	*/
	//1.创建父类的对象
	p1 := Person{name: "张三", age: 30}
	fmt.Println(p1)              //{张三 30}
	fmt.Println(p1.name, p1.age) //张三 30

	//2.创建子类的对象
	s1 := Student{Person{"李四", 17}, "云南大学"}
	fmt.Println(s1) //{{李四 17} 云南大学}

	s2 := Student{Person: Person{name: "rose", age: 19}, school: "北京大学"}
	fmt.Println(s2) //{{rose 19} 北京大学}

	var s3 Student
	s3.Person.name = "王五"
	s3.Person.age = 19
	s3.school = "清华大学"
	fmt.Println(s3) //{{王五 19} 清华大学}

	s3.name = "Ruby"
	s3.age = 16
	fmt.Println(s3) //{{Ruby 16} 清华大学}

	fmt.Println(s1.name, s1.age, s1.school) //李四 17 云南大学
	fmt.Println(s2.name, s2.age, s2.school) //rose 19 北京大学
	fmt.Println(s3.name, s3.age, s3.school) //Ruby 16 清华大学

	/*
		s3.Person.name ---> s3.name
		Student结构体将Person结构体作为一个匿名字段了
		那么Person中的字段，对于Student字段来讲，就是提升字段
		Student对象可以直接访问Person中的字段
	*/
}

//1.定义父类
type Person struct {
	name string
	age  int
}

//2.定义子类
type Student struct {
	Person        //模拟继承父类
	school string //子类的新增属性
}
```

### 结构体方法

```golang
package main

import "fmt"

func main() {
	/*
		方法：method
			一个方法就是一个包含了接受者的函数，接受者可以是命名类型或者结构体类型的一个值或者一个指针
			所有给定类型的方法属于该类型的方法集
		语法：
			func (接受者) 方法名（参数列表）（返回值列表）{

			}
		总结：method,通函数类似，区别需要有接受者，也就是调用者

		对比哈数：
			A:意义：
				方法：某个类别的行为功能，需要指定的接受调用者
				函数：一段独立功能的代码，可以直接调用
			B:语法：
				方法：方法名可以相同，只要接收者不同
				函数：命名不能冲突
	*/
	w1 := Worker{name: "王二狗", age: 30, sex: "男"}
	w1.work() //王二狗 在工作

	w2 := &Worker{name: "Ruby", age: 34, sex: "女"}
	fmt.Println("%T\n", w2) //&{Ruby 34 女}
	w2.work()               //Ruby 在工作

	w2.rest() //Ruby 休息了
	w1.rest() //王二狗 休息了

	w2.printInfo()                  //工人姓名：Ruby,工人年龄:34,工人性别:女
	c1 := Cat{color: "白色的", age: 1} //颜色：白色的,年龄:1
	c1.printInfo()

}

//1.定义一个工人结构体
type Worker struct {
	//字段
	name string
	age  int
	sex  string
}

type Cat struct {
	color string
	age   int
}

//2.定义一个方法
func (w Worker) work() {
	fmt.Println(w.name, "在工作")
}

func (p *Worker) rest() { //p=w2 ,p=w1的地址
	fmt.Println(p.name, "休息了")
}

func (p *Worker) printInfo() {
	fmt.Printf("工人姓名：%s,工人年龄:%d,工人性别:%s\n", p.name, p.age, p.sex)
}

func (p *Cat) printInfo() {
	fmt.Printf("颜色：%s,年龄:%d", p.color, p.age)
}
```

### 方法继承

```golang
package main

import "fmt"

func main() {
	/*
		OOP中的继承性：
			如果两个类（class）存在继承关系，其中一个是子类，另一个作为父类，那么：
				1.子类可以直接访问父类的属性和方法
				2.子类可以新增自己的属性和方法
				3.子类可以重写父类的方法（override，就是父类已有的方法，重新实现）

		go语言的结构体嵌套
			1.模拟继承性：is - a
				type A struct{
					field
				}
				type B struct{
					A//匿名字段
				}
			2.模拟聚合关系：has - a
				type C struct{
					field
				}
				type D struct{
					c C//聚合关系
				}
	*/
	//1.创建Person类型
	p1 := Person{name: "王二狗", age: 30}
	fmt.Println(p1.name, p1.age) //父类对象，访问父类属性字段
	p1.eat()                     //父类方法，吃窝窝头了

	//2.创建Student类
	s1 := Student{Person{"Ruby", 18}, "云南大学"}
	fmt.Println(s1.name, s1.age, s1.school) //Ruby 18 云南大学
	s1.eat()                                //父类方法，吃窝窝头了----->子类调用父类方法
	s1.study()                              //子类新增的方法，学生学习了
	s1.eat()                                //子类重写的方法，吃炸鸡---->子类对象访问重写的方法
}

//1.定义一个父类
type Person struct {
	name string
	age  int
}

//2.定义一个子类
type Student struct {
	Person //结构体嵌套，模拟继承性
	school string
}

//3.方法
func (p Person) eat() {
	fmt.Println("父类方法，吃窝窝头了")
}

func (s Student) study() {
	fmt.Println("子类新增的方法，学生学习了")
}

func (s Student) eat() {
	fmt.Println("子类重写的方法，吃炸鸡")
}

```

### 接口

```golang
package main

import "fmt"

func main() {
	/*
		接口：interface
			在go中，接口是一组方法签名

			当某个类型为这个接口中的所有方法提供了方法的实现，他被称为实现接口

			go语言中，接口和类型的实现方式，是非嵌入式

			//其他语言中，要显示的定义
			class Mouser implements USB{}
		1.当需要接口类型的对象时，可以使用任意实现类对象代替
		2.接口对象不能访问实现类中的属性

	*/
	//1.创建Mouser模型
	m1 := Mouse{"逻辑小红"}
	fmt.Println(m1.name) //逻辑小红
	//2.创建FlashDisk
	f1 := FlashDisk{"闪迪64G"}
	fmt.Println(f1.name) //闪迪64G

	testInterface(m1)
	//逻辑小红 鼠标,准备就绪,可以开始工作了,点点点...
	//逻辑小红 结束工作，可以安全退出...

	testInterface(f1)
	//闪迪64G 准备开始工作，可以进行数据的存储...
	//闪迪64G 可以弹出...

	var usb USB
	usb = m1
	usb.start() //逻辑小红 鼠标,准备就绪,可以开始工作了,点点点...
	usb.end()   //逻辑小红 结束工作，可以安全退出...
	fmt.Println()
}

//1.定义接口
type USB interface {
	start() //USB开始工作
	end()   //USB结束工作
}

//2.实现类
type Mouse struct {
	name string
}
type FlashDisk struct {
	name string
}

func (m Mouse) start() {
	fmt.Println(m.name, "鼠标,准备就绪,可以开始工作了,点点点...")
}

func (m Mouse) end() {
	fmt.Println(m.name, "结束工作，可以安全退出...")
}

func (f FlashDisk) start() {
	fmt.Println(f.name, "准备开始工作，可以进行数据的存储...")
}

func (f FlashDisk) end() {
	fmt.Println(f.name, "可以弹出...")
}

//3.测试方法
func testInterface(usb USB) { //usb=m1 f1
	usb.start()
	usb.end()
}

```

### 空接口

```python
package main

import "fmt"

func main() {
	/*
		空接口（interface{}）
			不包含任何的方法，正因为如此，所有的的类型都实现了空接口，因此空接口可以存储任意类型的数值
		fmt包下的print系列函数：
			func Print(a ..interface{}) (n int,err error)
			func Println(a ...interface{}) (n int, err error)
	*/
	var a1 A = Cat{"花猫"}
	var a2 A = Person{"王二狗", 30}
	var a3 A = "haha"
	var a4 A = 100
	fmt.Println(a1) //{花猫}
	fmt.Println(a2) //{王二狗 30}
	fmt.Println(a3) //haha
	fmt.Println(a4) //100
	test1(a1)       //{花猫}
	test1(a2)       //{王二狗 30}
	test1(3.14)     //3.14
	test1("Ruby")   //Ruby

	test2(a3)
	test2(1000)

	//map:key字符串，value任意类型
	map1 := make(map[string]interface{})
	map1["name"] = "李小华"
	map1["age"] = 30
	map1["friend"] = Person{"Jerry", 18}
	fmt.Println(map1) //map[age:30 friend:{Jerry 18} name:李小华]

	//切片，存储任意类型的数据
	slice1 := make([]interface{}, 0, 10)
	slice1 = append(slice1, a1, a2, a3, a4, 100, "hehe") //[{花猫} {王二狗 30} haha 100 100 hehe]
	fmt.Println(slice1)

	test3(slice1)
	//第1个数据:{花猫}
	//第2个数据:{王二狗 30}
	//第3个数据:haha
	//第4个数据:100
	//第5个数据:100
	//第6个数据:hehe

}

func test3(slice2 []interface{}) {
	for i := 0; i < len(slice2); i++ {
		fmt.Printf("第%d个数据:%v\n", i+1, slice2[i])
	}
}

//接口A是空接口,理解为代表了任意类型
func test1(a A) {
	fmt.Println(a)
}

func test2(a interface{}) {
	fmt.Println("----->", a)
}

//空接口
type A interface {
}

type Cat struct {
	color string
}

type Person struct {
	name string
	age  int
}

```

### 接口嵌套

```golang
package main

import "fmt"

func main() {
	/*
		接口的嵌套
	*/
	var cat Cat = Cat{}
	cat.test1() //test1()...
	cat.test2() //test2()...
	cat.test3() //test3()...

	fmt.Println("------------------")
	var a1 A = cat
	a1.test1() //test1()...

	fmt.Println("------------------")
	var b1 B = cat
	b1.test2() //test2()...

	fmt.Println("------------------")
	var c1 C = cat
	c1.test1() //test1()...
	c1.test2() //test2()...
	c1.test3() //test3()...

	var a2 A = c1
	a2.test1() //test1()...
}

type A interface {
	test1()
}

type B interface {
	test2()
}

type C interface {
	A
	B
	test3()
}

type Cat struct { //如果想实现接口c 那不止要实现接口c的方法，还要实现A,B中的方法
}

func (c Cat) test1() {
	fmt.Println("test1()...")
}

func (c Cat) test2() {
	fmt.Println("test2()...")
}

func (c Cat) test3() {
	fmt.Println("test3()...")
}

```

### 接口断言

```golang
package main

import (
	"fmt"
	"math"
)

func main() {
	/*
		接口断言：
			方式一：
				1.instance:=接口对象.(实际类型）//不安全, 会panic()
				2.instance, ok := 接口对象.(实际类型)//安全
			方式二：
				swith instance := 接口对象.(type){
					case 实际类型1：
						...
					case 实际类型2：
						...
				}
	*/
	var t1 Triangle = Triangle{3, 4, 5}
	fmt.Println(t1.peri())        //12
	fmt.Println(t1.area())        //6
	fmt.Println(t1.a, t1.b, t1.c) //3 4 5

	var c1 Circle = Circle{4}
	fmt.Println(c1.peri()) //25.132741228718345
	fmt.Println(c1.area()) //50.26548245743669
	fmt.Println(c1.radius) //4

	var s1 Shape
	s1 = t1
	fmt.Println(s1.peri()) //12
	fmt.Println(s1.area()) //6

	var s2 Shape
	s2 = c1
	fmt.Println(s2.peri()) //25.132741228718345
	fmt.Println(s2.area()) //50.26548245743669

	testShape(t1) //周长：12.00,面积：6.00
	testShape(c1) //周长：25.13,面积：50.27
	testShape(s1) //周长：12.00,面积：6.00

	getType(t1) //是三角形,三边是: 3 4 5
	getType(s1) //是三角形,三边是: 3 4 5
	getType(c1) //是圆形，半径是： 4

	var t2 *Triangle = &Triangle{3, 4, 2}
	getType(t2)
	//ins:*main.Triangle,0xc000082020,0xc0000541c0
	//s:*main.Triangle,0xc0000421f0,0xc0000541c0

	getType2(t2)
	//ins:*main.Triangle,0xc000082028
	//s:*main.Triangle,0xc000042200
	getType2(t1) //是三角形,三边是: 3 4 5
}

func getType2(s Shape) {
	switch ins := s.(type) {
	case Triangle:
		fmt.Println("是三角形,三边是:", ins.a, ins.b, ins.c)
	case Circle:
		fmt.Println("是圆形，半径是：", ins.radius)
	case *Triangle:
		fmt.Printf("ins:%T,%p\n", ins, &ins)
		fmt.Printf("s:%T,%p\n", s, &s)
	default:
		fmt.Println("我也不知道了")
	}
}

func getType(s Shape) {
	//断言
	if ins, ok := s.(Triangle); ok {
		fmt.Println("是三角形,三边是:", ins.a, ins.b, ins.c)
	} else if ins, ok := s.(Circle); ok {
		fmt.Println("是圆形，半径是：", ins.radius)
	} else if ins, ok := s.(*Triangle); ok {
		fmt.Printf("ins:%T,%p,%p\n", ins, &ins, ins)
		fmt.Printf("s:%T,%p,%p\n", s, &s, s)
	} else {
		fmt.Println("我也不知道了")
	}
}

func testShape(s Shape) {
	fmt.Printf("周长：%.2f,面积：%.2f\n", s.peri(), s.area())
}

//1.定义一个接口
type Shape interface {
	peri() float64 //形状的周长
	area() float64 //形状的面积
}

//2.定义实现类：三角形
type Triangle struct {
	//a float64
	//b float64
	//c float64
	a, b, c float64
}

func (t Triangle) peri() float64 {
	return t.a + t.b + t.c
}

func (t Triangle) area() float64 {
	p := t.peri() / 2
	s := math.Sqrt(p * (p - t.a) * (p - t.b) * (p - t.c))
	return s
}

type Circle struct {
	radius float64
}

func (c Circle) peri() float64 {
	return 2 * math.Pi * c.radius
}

func (c Circle) area() float64 {
	return math.Pi * math.Pow(c.radius, 2)
}

```

### type关键字新定义

```golang
package main

import (
	"fmt"
	"strconv"
)

func main() {
	/*
		type:用于类型定义和类型别名
			1.类型定义：type 类型名 Type
			2.类型别名：type 类型名 = Type
	*/
	var i1 myint
	var i2 = 100 //int
	fmt.Println(i1, i2)

	var name mystr
	name = "haha"
	var s1 string
	s1 = "hehe"
	fmt.Println(name, s1)

	//i1 = i2//cannot use i2 (type int) as type myint in assignment

	//name = s1//cannot use s1 (type string) as type mystr in assignment

	fmt.Printf("%T,%T,%T,%T\n", i1, i2, name, s1) //main.myint,int,main.mystr,string

	fmt.Println("-------------------------")
	res1 := fun1()
	fmt.Println(res1(10, 20)) //1020

	fmt.Println("-------------------------")
	var i3 myint2
	i3 = 1000
	fmt.Println(i3) //1000
	i3 = i2
	fmt.Println(i3) //100

	fmt.Printf("%T,%T,%T\n", i1, i2, i3) //main.myint,int,int

}

//1.定义一个新的类型
type myint int
type mystr string

//2.定义函数类型
type myfun func(int, int) (string)

func fun1() myfun {
	fun := func(a, b int) string {
		s := strconv.Itoa(a) + strconv.Itoa(b)
		return s
	}
	return fun
}

//3.类型别名
type myint2 = int //不是重新定义型的数据类型，只是给int起别名，和int可以通用

```

### type别名

```golang
package main

import "time"

func main() {
	/*

	 */

}

//type MyDuration = time.Duration
//func (m MyDuration) SimpleSet(){//cannot define new methods on non-local type time.Duration
//
//}

type MyDuration time.Duration

func (m MyDuration) SimpleSet() {

}

```

### type别名例子

```golang
package main

import "fmt"

type Person struct {
	name string
}

func (p Person) show() {
	fmt.Println("person---->", p.name)
}

//类型别名
type People = Person

//func (p People)show(){//Person.show redeclared in this block
//
//}
func (p People) show2() {
	fmt.Println("people---->", p.name)
}

type Student struct {
	//嵌入两个结构体
	Person
	People
}

func main() {
	var s Student
	//s.name = "haha"//ambiguous selector s.name
	s.Person.name = "Tom"
	//s.show()//ambiguous selector s.show
	s.Person.show() //person----> Tom

	fmt.Printf("%T,%T\n", s.Person, s.People) //main.Person,main.Person

	s.People.name = "Jerry"
	s.People.show()  //person----> Jerry
	s.People.show2() //people----> Jerry
}

```

### error是一种数据类型（Errors.New(),fmt.Errorf()）

```golang
package main

import (
	"errors"
	"fmt"
)

func main() {
	/*
		error:内置的数据类型，内置的接口
			定义方法：Error() string
		使用go语言提供好的包：
			errors包下的函数：New(),创建一个error对象
			fmt下Errorf()函数
				func Errorf(fomat string,a ...interface{}) error
	*/

	//1.创建一个error数据
	err1 := errors.New("自己创建的")
	fmt.Println(err1)        //自己创建的
	fmt.Printf("%T\n", err1) //*errors.errorString

	//2.另一个创建error的方法
	err2 := fmt.Errorf("错误提示吗：%d", 100)
	fmt.Println(err2)        //错误提示吗：100
	fmt.Printf("%T\n", err2) //*errors.errorString

	fmt.Println("----------------------------")
	err3 := checkAge(-30)
	if err3 != nil {
		fmt.Println(err3)
		return
	}
	fmt.Println("程序继续执行")

}

//设计一个函数：验证年龄是否合法，如果为负数，就返回一个error
func checkAge(age int) error {
	if age < 0 {
		//返回error对象
		//return errors.New("年龄不合法")
		err := fmt.Errorf("你给定的年龄是%d,不合法", age)
		return err
	}
	fmt.Println("年龄是:", age)
	return nil
}

```

### error1打开本地文件

```golang
package main

import (
	"fmt"
	"os"
)

func main() {
	f, err := os.Open("text1.txt")
	if err != nil {
		//log.Fatal((err))//2019/09/01 17:49:40 open test.txt: The system cannot find the file specified.
		fmt.Println(err) //open test.txt: The system cannot find the file specified.
		if ins,ok := err.(*os.PathError);ok{
			fmt.Println("1.Op:",ins.Op)
			fmt.Println("2.Path:",ins.Path)
			fmt.Println("3.Err:",ins.Err)
			//1.Op: open
			//2.Path: text1.txt
			//3.Err: The system cannot find the file specified.
		}
		return
	}
	fmt.Println(f.Name(), "文件打开成功了...")
}

```

### err例子2

```golang
package main

import (
	"fmt"
	"net"
)

func main() {
	addr,err := net.LookupHost("www.baidujintinatianqiyoudianre.com")
	fmt.Println(err)//lookup www.baidujintinatianqiyoudianre.com: no such host
	if ins,ok := err.(*net.DNSError);ok{
		if ins.Timeout(){
			fmt.Println("操作超时...")
		}else if ins.Temporary(){
			fmt.Println("临时性错误...")
		}else{
			fmt.Println("通常错误...")
		}
	}
	fmt.Println(addr)//[]

}
```



### err例子3

```golang
package main

import (
	"fmt"
	"path/filepath"
)

func main() {
	files, err := filepath.Glob("[")
	//files,_:=filepath.Glob("[")//忽略了错误
	if err != nil && err == filepath.ErrBadPattern {
		fmt.Println(err) //syntax error in pattern
		return
	}
	fmt.Println("files:", files)
}

```

### 自定义错误类型

```golang
package main

import "fmt"

func main() {
	length, width := -6.7, -9.1
	area, err := rectArea(length, width)
	if err != nil {
		fmt.Println(err)
		if err, ok := err.(*areaError); ok {
			if err.lenghtNegative() {
				fmt.Printf("error:长度，%.2f,小于零\n", err.length)
			}
			if err.widthNegative() {
				fmt.Printf("error:宽度，%.2f,小于零\n", err.width)
			}
		}
		return
	}
	fmt.Println("矩形的面积是：", area)
}

type areaError struct {
	msg    string  //错误的描述
	length float64 //发生错误的时候，矩形的长度
	width  float64 //发生错误的时候，矩形的宽度
}

func (e *areaError) Error() string {
	return e.msg
}

func (e *areaError) lenghtNegative() bool {
	return e.length < 0
}

func (e *areaError) widthNegative() bool {
	return e.width < 0
}

func rectArea(length, width float64) (float64, error) {
	msg := ""
	if length < 0 {
		msg = "长度小于0"
	}
	if width < 0 {
		if msg == "" {
			msg = "宽度小于0"
		} else {
			msg += ",宽度也小于零"
		}
	}
	if msg != "" {
		return 0, &areaError{msg, length, width}
	}
	return length * width, nil
}

```

### panic()和recover()

```golang
package main

import "fmt"

func main() {
	/*
		panic：词义“恐慌”
		recover:"恢复"
		go语言利用panic(),recover(),实现程序中的极特殊的异常的处理
			panic(),让当前的程序进入恐慌，中断程序执行
			recover(),让程序恢复，必须再defer函数中执行
	*/
	defer func() {
		if msg := recover(); msg != nil {
			fmt.Println(msg, "程序恢复了")
		}
	}()

	funA()
	defer myprint("defer main:3.....")
	funB()
	defer myprint("defer main:4.....")
	fmt.Println("main..over")

}

func myprint(s string) {
	fmt.Println(s)
}

func funA() {
	fmt.Println("funA()")
}
func funB() { //外围函数
	fmt.Println("funB()")
	defer myprint("defer funB():1.....")
	for i := 1; i <= 10; i++ {
		fmt.Println("i:", i)
		if i == 5 {
			//让程序中断
			panic("funB函数，恐慌了")
		}
	} //当外围函数的代码发生了运行恐慌，只有其中所有的已经defer的函数全部执行完毕后，该运行恐慌才会真正被扩展至调用处
	defer myprint("defer funB():2.....")
}

```