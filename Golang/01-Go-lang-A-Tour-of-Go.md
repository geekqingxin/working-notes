[A Tour of Go](https://tour.golang.org/)

一个以交互形式组织的一个Go介绍，由三部分组成。第一部分涵盖了基本语法和数据结构；第二部分讨论了方法和接口；第三部分介绍了Go并发原语。每部分结束部分都有一些练习，因此你可以练习你所学到的内容。你可以[在线使用](https://tour.golang.org/)，也可以在本地安装。

本地安装方法：

    go get golang.org/x/tour/gotour

这将安装gotour到工作空间的bin目录中。


# 1. Hello，世界

## 1.1 Go离线环境

    go tool tour

# 2. 基础

从现在开始Go语言的基础学习。

声名变量，函数调用，所有在接下来学习需要了解的内容。

## 2.1 Packages, variables and functions

### 2.1.1 Packages

每个Go程序都是由包组成的。

程序从main包开始执行。

    package main

    import (
        "fmt"
        "math/rand"
    )

    func main() {
        fmt.Println("My favorite number is", rand.Intn(10))
    }

按照约定，包名与import路径的最后元素相同。例如，"math/rand"包由以package rand开始语句的文件组成。


### 2.1.2 Imports

以下代码，通过()来分组导入路径。


    package main
    
    import (
        "fmt"
        "math"
    )
    
    func main() {
        fmt.Printf("Now you have %g problems.", math.Sqrt(7))
    }

同样，你也可以像现在这样来导入packages：

    import "fmt"
    import "math"

通常情况下，使用()来划分导入路径是一种更好的方式。


### 2.1.3 Exported names

在Go语言中，如果名字的首字母大写，他对外则是可见的。如Pizza就是一个被导出的名字。同样的Pi，他是从math包中被导出的。

然而，pizza和pi都不是首字母大写，同他们不会被导出。

当导入一个包，你可以只能引用它导出的名字。任何没有被导出的名字，对外都是不可见的。


    package main
    
    import (
        "fmt"
        "math"
    )
    
    func main() {
        fmt.Println(math.Pi)
    }


### 2.1.4 Functions

函数可以有0到多个参数。

NOTE: 类型位于变量名后面。

    package main

    import "fmt"

    func add(x int, y int) int {
        return x + y
    }

    func main() {
        fmt.Println(add(42, 13))
    }


### 2.1.5 Functions continued

一次声明多个同类型的参数：

    package main

    import "fmt"

    func add(x, y int) int {
        return x + y
    }

    func main() {
        fmt.Println(add(42, 13))
    }
### 2.1.6 Multiple results

一次返回多个值的函数：

    package main

    import "fmt"

    func swap(x, y string) (string, string) {
        return y, x
    }

    func main() {
        a, b := swap("hello", "world")
        fmt.Println(a, b)
    }

### 2.1.7 Named return values

Go的返回值可以被命名。

    func split(sum int) (x, y int)

声明方式如下：


    package main

    import "fmt"

    func split(sum int) (x, y int) {
        x = sum * 4 / 9
        y = sum - x
        return
    }

    func main() {
        fmt.Println(split(17))
    }

### 2.1.8 变量

通过var声明变量。如果是用在函数参数列表时，可以放在变量后面。

    package main

    import "fmt"

    var c, python, java bool

    func main() {
        var i int
        fmt.Println(i, c, python, java)
    }

var语句可以用在包或函数上。


### 2.1.9 声明变量并初始化

var声明可以直接初始化。

    var i, j int = 1, 2

如果直接初始化，类型可以被省略；变量将使用初始化的类型。


### 2.1.10 短变量声明方式

在函数内部，:=短分配语句可以被用来代替var隐式类型声明。

var i, j int = 1, 2
k := 3

在函数外面，每个语句由var，func开头，因此:=不可用。


### 2.1.11 基本类型

Go基本类型：

    bool
    
    string
    
    int int8 int16 int32 int64
    
    uint uint8 uint16 uint32 uint64 uintptr
    
    byte // alias for uint8
    rune // alias for int32, represents a Unicode code point

    float32 float64

    complex64 complex128

示例：

    package main

    import (
        "fmt"
        "math/cmplx"
    )

    var (
        ToBe bool = false
        MaxInt uint64 = 1 << 64 - 1
        z complex128 = cmplx.Sqrt(-5 + 12i)
    )

    func main() {
        fmt.Printf("Type: %T Value: %v\n", ToBe, ToBe)
        fmt.Printf("Type: %T Value: %v\n", MaxInt, MaxInt)
        fmt.Printf("Type: %T Value: %v\n", z, z)
    }

int, uint和uintptr在32位机器上是32位；在64位机器上是64位。

当需要integer值时，你应该使用int；除非你有特定原因可以使用指定大小的int或无符号的integer。


### 2.1.12 Zero values

声明变量时，如果没有明确的初始值，值为zero value。

* 0 for numeric types,
* false for the boolean type
* ""(the empty string) for strings


    package main

    import "fmt"

    func main() {
        var i int
        var f float64
        var b bool
        var s string
        fmt.Printf("%v %v %v %q\n", i, f, b, s)
    }

### 2.1.13 Type conversions

通过T(v)表达式，可以将v值转换成T类型。

数字转换：

    var i int = 42;
    var f float64 = float64(i)
    var u uint = uint(f)

或，用更简单的形式转换：

    i := 42
    f := float64(i)
    u := uint(f)

不像在C语言中，Go在不同类型间转换要求明确的转换。


### 2.1.14 Type inference

当声明变量时没有明确的指定类型(既没有使用:=，也没有使用var = 表达式)，变量类型会从右边的值推断出来。

当声明的右边是类型化的，新变量的类型与之一致：

    var i int
    j := i // j is an int

当右边包含无类型的数字常量，新变量可能是int，float64或complex128；这依赖于常量的精度：

    i := 42 // int
    f :=3.142 // float64
    g := 0.867 + 0.5i // complex128


示例：

    package main

    import "fmt"

    func main() {
        v := 42 // change me!
        fmt.Printf("v is of type %T\n", v)
    }


### 2.1.15 Constants

常量与变量声明方式类似，除了是用const关键字。

常量可以是字符, string, boolean或数字。

常量不能使用:=来声明。

    package main

    import "fmt"

    const Pi = 3.14

    func main() {
        const World = "世界"
        fmt.Println("Hello", World)
        fmt.Println("Happy", Pi, "Day")

        const Truth = true
        fmt.Println("Go rules?", Truth)
    }


### 2.1.16 数值常量

数字常量是高精度的值。

无类型的常量，类型依赖于它的上下文。

(int最大可以存储64-bit integer，有时会少一些。)



    package main

    import "fmt"

    const (
        // Create a huge number by shifting a 1 bit left 100 places.
        // In other words, the binary number that is 1 followed by 100 zeroes.
        Big = 1 << 100
        // Shift it right again 99 places, so we end up with 1<<1, or 2.
        Small = Big >> 99
    )

    func needInt(x int) int { return x*10 + 1 }
    func needFloat(x float64) float64 {
        return x * 0.1
    }

    func main() {
        fmt.Println(needInt(Small))
        fmt.Println(needFloat(Small))
        fmt.Println(needFloat(Big))
    }


## 2.2 Flow control statements: for, if, else, switch and defer


### 2.2.1 For

Go只有一种循环结构，for loop。

基本的for循环有三个组件，分号分隔：

* 初始化语句(the init statement)：第一次循环前执行
* 条件表达式(the condition expression)：在每次循环后计算
* 步长语句(the post statement)：在每次循环后执行

初始化语句常常使用短变量声明，被声明的变量只在for语句的scope中可见。

如果布尔条件为false，循环将停止执行。

> Note
> 
> 不像在C、Java或JavaScript中，在不需要小括号来括住这三个组件，但是{}是必须的。


    package main

    import "fmt"

    func main() {
        sum := 0
        for i := 0; i < 10; i++ {
            sum += i
        }
        fmt.Println(sum)
    }


### 2.2.2 For continued

init和post语句可选：

    package main

    import "fmt"

    func main() {
        sum := 1
        for ; sum < 1000; {
            sum += sum
        }
        fmt.Println(sum)
    }

### 2.2.3 For is Go's "while"

可以只包含condition expression部分。

    package main

    import "fmt"

    func main() {
        sum := 1
        for sum < 1000 {
            sum += sum
        }
        fmt.Println(sum)
    }

### 2.2.4 Forever

如果省略循环条件，它将无限循环。

    for {
    }


### 2.2.5 If

if像for一样，条件不需要放在小括号中，但是{}是必须的。


### 2.2.6 If with a short statement

像for一样，if语句可以包含一个短声明语句。

语句中声明的变量只在if上下文中有效。

    package main

    import (
        "fmt"
        "math"
    )

    func pow(x, n, lim float64) float64 {
        if v := math.Pow(x, n); v < lim {
            return v
        }
        return lim
    }

    func main() {
       fmt.Println(
            pow(3, 2, 10),
            pow(3, 3, 20),
	   )
    }


### 2.2.7 If and else

在if中声明的短语句在else块中也是可见的。

    package main

    import (
        "fmt"
        "math"
    )

    func pow(x, n, lim float64) float64 {
        if v := math.Pow(x, n); v < lim {
            return v
        } else {
            fmt.Printf("%g >= %g\n", v, lim)
        }
        // can't use v here, though
        return lim
    }

    func main() {
        fmt.Println(
            pow(3, 2, 10),
            pow(3, 3, 20),
        )
    }


### 2.2.8 Switch

case体自动break，除非它以fallthrough语句结尾。


    package main

    import (
        "fmt"
        "runtime"
    )

    func main() {
        fmt.Print("Go runs on ")
        switch os := runtime.GOOS; os {
        case "darwin":
            fmt.Println("OS X.")
        case "linux":
            fmt.Println("Linux.")
        default:
            // freebsd, openbsd,
            // plan9, windows...
            fmt.Printf("%s.", os)
        }
    }


### 2.2.9 Switch evaluation order

switch计算case从上到下，遇到匹配的case，则停止。

    switch i {
        case 0:
        case f():
    }

以上代码if i == 0时，不会调用f函数。


### 2.2.10 Switch with no condition

没有条件的switch等同与switch true。

这个结构可以代替多分支的if-then-else。

    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        t := time.Now()
        switch {
        case t.Hour() < 12:
            fmt.Println("Good morning!")
        case t.Hour() < 17:
            fmt.Println("Good afternoon.")
        default:
            fmt.Println("Good evening.")
        }
    }


### 2.2.11 Defer

defer语句，延迟函数执行，只到外围函数返回后才再执行。

    package main
    
    import "fmt"

    func main() {

        defer fmt.Println("world")

        fmt.Println("hello")
    }

延迟的调用参数是被立即计算的，但函数只到外围函数执行后他才执行。


### 2.2.12 Stacking defers

延迟函数调用是被放在一个stack中。当函数返回后，他以last-in-first-out顺序执行延迟调用。


    package main

    import "fmt"

    func main() {
        fmt.Println("counting")

        for i := 0; i < 10; i++ {
            defer fmt.Println(i)
        }

        fmt.Println("done")
    }


To learn more about defer statements read this [blog post](https://blog.golang.org/defer-panic-and-recover).


## 2.3 More types: structs, slices and maps

### 2.3.1 Pointers

Go有指针，指针持有一个值的内存地址。

*T是一个指向T值的指针。它的zero value是nil。

    var p *int

&操作符将会产生一个指向操作数的指针。

    i := 42
    p = &i

*操作符表示指针底层的值。

    fmt.Println(*p)
    *p = 21

这也被称为非关联化(dereferencing)或间接或迂回(indirecting)。

不像C，Go没有指针运算。


    package main

    import "fmt"

    func main() {
        i, j := 42, 2701

        p := &i // point to i
        fmt.Println(*p) // read i through the pointer
        *p = 21 // set i through the pointer
        fmt.Println(i)  // see the new value of i

        p = &j // point to j
        *p = *p / 37 // divide j through the pointer
        fmt.Println(j) // see the new value of j
    }


### 2.3.2 Structs


一个struct是一个字段的(fields)集合。

(增加一个type声明，实现你期望的功能。)


    package main

    import "fmt"

    type Vertex struct {
        X int
        Y int
    }

    func main() {
        fmt.Println(Vertex{1, 2})
    }


### 2.3.3 Struct Fields

struct字段通过使用.来访问。

    package main

    import "fmt"

    type Vertex struct {
        X int
        Y int
    }

    func main() {
        v := Vertex{1, 2}
        v.X = 4
        fmt.Println(v.X)
    }


### 2.3.4 Pinters to structs

struct fields可以通过struct指针进行访问。

为了访问struct的X字段，当我们有一个struct指针p时，我们可以写(*p).X。无论如何，符号是很讨厌的，因此语言允许我们使用p.X来代替，不需要明确的解引用(dereference)。

    package main

    import "fmt"

    type Vertex struct {
        X int
        Y int
    }

    func main() {
        v := Vertex{1, 2}
        p := &v
        p.X = 1e9
        fmt.Println(v)
    }


### 2.3.5 Struct Literals

struct字面量表示使用列出的字段值来分配struct的值。

可以通过Name:语法列出字段的子集。(与被命名字段的顺序无关)

特殊前缀&返回一个指向struct值的指针。


    package main

    import "fmt"

    type Vertex struct {
        X, Y int
    }

    var (
        v1 = Vertex{1, 2}  // has type Vertex
        v2 = Vertex{X: 1}  // Y:0 is implicit
        v3 = Vertex{}      // X:0 and Y:0
        p  = &Vertex{1, 2} // has type *Vertex
    )

    func main() {
        fmt.Println(v1, p, v2, v3)
    }

### 2.3.6 Arrays

[n]T表示一个n个T类型值的集合。

表达式：

    var a [10]int

声明一个包含10个数字的数组a。

数组的长度是类型的一部分，因此数组不能被再分配。这似乎是有一些限制，但不用担心；Go提供了一种使用数组的便利方式。

    package main

    import "fmt"

    func main() {
        var a [2]string
        a[0] = "Hello"
        a[1] = "World"
        fmt.Println(a[0], a[1])
        fmt.Println(a)

        primes := [6]int{2, 3, 5, 7, 11, 13}
        fmt.Println(primes)
    }


### 2.3.7 Slices


数组有固定的长度。Slice是动态大小的，是数组元素的动态视图。实践中，slice比数组更通用。

[]T是一个包含T类型元素的slice。

这个表达式创建了数组a前五个元素的slice：

    a[0:5]

示例：

    package main

    import "fmt"

    func main() {
        primes := [6]int{2, 3, 5, 7, 11, 13}

        var s []int = primes[1:4]
        fmt.Println(s)
    }


### 2.3.8 Slices are like references to arrays

slice不存在任何数据，他仅仅描述了底层数组的一部分。

修改slice元素，也将修改底层数组中对应的元素。

其它slice共享相同底层数组的话，也将看见这些变化。


    package main

    import "fmt"

    func main() {
        names := [4]string{
            "John",
            "Paul",
            "George",
            "Ringo",
        }
        fmt.Println(names)

        a := names[0:2]
        b := names[1:3]
        fmt.Println(a, b)

        b[0] = "XXX"
        fmt.Println(a, b)
        fmt.Println(names)
    }


### 2.3.9 Slice literals

slice字面量像一个没有长度的数组字面量。

数组字面量：

    [3]bool{true, true, false}

创建一个slice引用上面的数组：

[]bool{true, true, false}


示例：

    package main

    import "fmt"

    func main() {
         q := []int{2, 3, 5, 7, 11, 13}
         fmt.Println(q)

         r := []bool{true, false, true, true, false, true}
         fmt.Println(r)

         s := []struct {
             i int
             b bool
         }{
             {2, true},
             {3, false},
             {5, true},
             {7, true},
             {11, false},
             {13, true},
        }
        fmt.Println(s)
    }


### 2.3.10 Slice defaults

当slice时，你可以忽略它的高或低边界，使用默认值替代。

低边界默认值是0，高边界是slice的长度。

对于数组：
    
    var a [10]int

以下表达式是相同的：

    a[0:10]
    a[:10]
    a[0:]
    a[:]

### 2.3.11 Slice length and capacity

slice有长度和容量。

slice的长度是它所包含的元素的个数。

slice的容量是底层数组从slice中第一个元素开始计算，到数组最后一个元素的个数。

slice的长度和容量可以通过以下表达式获取：len(s)，cap(s)。

可以通过re-slicing来扩展slice的长度，给他提供一个有效的容量。


    package main

    import "fmt"

    func main() {
        s := []int{2, 3, 5, 7, 11, 13}
        printSlice(s)

        // Slice the slice to give it zero length.
        s = s[:0]
        printSlice(s)

        // Extend its length.
        s = s[:4]
        printSlice(s)

        // Drop its first two values.
        s = s[2:]
        printSlice(s)
    }

    func printSlice(s []int) {
        fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
    }

### 2.3.12 Nil slices

slice的zero value是nil。

nil slice的长度和容量是0，并且没有底层数组。


    package main

    import "fmt"

    func main() {
        var s []int
        fmt.Println(s, len(s), cap(s))
        if s == nil {
            fmt.Println("nil!")
        }
    }


### 2.3.13 Creating a slice with make

可以内置的make函数创建slice；通过这个，你也可以创建动态数组。

make分配一个zeroed数组，并返回一个引用该数组的slice：

    a := make([]int, 5) // len(a) = 5

为了指定一个容量，可以向make传入第3个值：

    b := make([]int, 0, 5) // len(b) = 0, cap(b) = 5

    b = b[:cap(b)] // len(b) = 5, cap(b) = 5

    b = b[1:] // len(b) = 4, cap(b) = 4

### 2.3.14 Slices of slices

Slices可以包含任意类型，包括别的slices。

    package main

    import (
        "fmt"
        "strings"
    )

    func main() {
        // Create a tic-tac-toe board.
        board := [][]string{
            []string{"_", "_", "_"},
            []string{"_", "_", "_"},
            []string{"_", "_", "_"},
        }

        // The players take turns.
        board[0][0] = "X"
        board[2][2] = "O"
        board[1][2] = "X"
        board[1][0] = "O"
        board[0][2] = "X"
    
        for i := 0; i < len(board); i++ {
            fmt.Printf("%s\n", strings.Join(board[i], " "))
        }
    }

### 2.3.15 Appending to a slice

追加新元素到slice，Go提供了内置的append函数。[内置package的文档](https://golang.org/pkg/builtin/#append)对append有描述。

    func append(s []T, vs ...T) []T

append的第一个参数s是T类型的slice，剩下的是追加到slice的T值。

append的最终值是一个包含了所有原始slice中元素以及新增加的元素。

如果s后面的数组太小，不能存放所有给定的值；一个更大的数组将被分配。返回的slice将指向新分配的数组。

To learn more about slices, read the [Slices: usage and internals](https://blog.golang.org/go-slices-usage-and-internals) article.

示例：

    package main

    import "fmt"

    func main() {
        var s []int
        printSlice(s)

        // append works on nil slices.
        s = append(s, 0)
        printSlice(s)

        // The slice grows as needed.
        s = append(s, 1)
        printSlice(s)

        // We can add more than one element at a time.
        s = append(s, 2, 3, 4)
        printSlice(s)
    }

    func printSlice(s []int) {
        fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
    }


### 2.3.16 Range

for循环的range形式，用来迭代一个slice或map。

当在slice上执行range，每次迭代返回两个值。第一个是索引，第二个是该索引位置上的元素副本。

    package main

    import "fmt"

    var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

    func main() {

        for i, v := range pow {

            fmt.Printf("2**%d = %d\n", i, v)
        }
    }


### 2.3.17 Range continued

你可以跳过index或value，通过赋值给_。

如果你只想要索引，可以把", value"整个删除。

    package main

    import "fmt"

    func main() {
        pow := make([]int, 10)
        for i := range pow {
            pow[i] = 1 << uint(i) // == 2**i
        }
        for _, value := range pow {
            fmt.Printf("%d\n", value)
        }
}


### 2.3.18 Maps

map映射key和value。

map的zero value是nil。nil的map没有key，也不能添加key。

make函数返回一个给定类型的map。

    package main

    import "fmt"

    type Vertex struct {
        Lat, Long float64
    }

    var m map[string]Vertex

    func main() {
        m = make(map[string]Vertex)
        m["Bell Labs"] = Vertex{
            40.68433, -74.39967,
        }
        fmt.Println(m["Bell Labs"])
    }


### 2.3.19 Map字面量

Map字面量像struct字面量，但是必须要有key。

    package main

    import "fmt"

    type Vertex struct {
        Lat, Long float64
    }

    var m = map[string]Vertex{
        "Bell Labs": Vertex{
            40.68433, -74.39967,
        },
    "Google": Vertex{
            37.42202, -122.08408,
        },
    }

    func main() {
        fmt.Println(m)
    }


### 2.3.20 Map literals continued

如果顶级类型是一个类型名，可以省略字面量的类型。

    package main

    import "fmt"

    type Vertex struct {
        Lat, Long float64
    }

    var m = map[string]Vertex{
        "Bell Labs": {40.68433, -74.39967},
        "Google":    {37.42202, -122.08408},
    }

    func main() {
        fmt.Println(m)
    }


### 2.3.21 Mutating Maps

插入或更新Map m中的元素：

    m[key] = elem

获取一个元素：

    elem = m[key]

删除元素：

    delete(m, key)


测试把一个存在的key，赋值给两个变量：

    elem, ok = m[key]

如果key在m中，ok为true。如果不在，ok是false。

如果key不在map中，elem根据map元素的类型取默认的zero value。

NOTE: 如果elem或ok没有被声明，可以使用短声明形式：

    elem, ok := m[key]


### 2.3.22 Function values

function也是值。它们也可以像其它值一样被传递。

function值可以被用作函数参数或返回值。


    package main

    import (
        "fmt"
        "math"
    )

    func compute(fn func(float64, float64) float64) float64 {
        return fn(3, 4)
    }

    func main() {
        hypot := func(x, y float64) float64 {
            return math.Sqrt(x*x + y*y)
        }
        fmt.Println(hypot(5, 12))

        fmt.Println(compute(hypot))
        fmt.Println(compute(math.Pow))
    }


### 2.3.23 Function closures

Go函数可以是闭包。闭包是一个引用函数外部的变量的函数值。函数可以访问和赋值给引用变量；在这种情况下，function对变量负责。

举例，adder函数返回一个闭包。每个闭包都对它所拥有的变量sum负责。

    package main

    import "fmt"

    func adder() func(int) int {
        sum := 0
        return func(x int) int {
            sum += x
            return sum
        }
    }

    func main() {
        pos, neg := adder(), adder()
        for i := 0; i < 10; i++ {
            fmt.Println(
                pos(i),
                neg(-2*i),
            )
        }
    }


# 3. Methods and interfaces

## 3.1


# 4. Concurrency









