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

## 3.1 Methods

Go没有类，但是可以在类型上定义方法。

方法是带有特殊接收器参数的函数。

接收器出现在func关键字与方法之间它所拥有的参数列表中。

例如，Abs方法有一个Vertex类型，被命名为v的接收器。


    package main

    import (
        "fmt"
        "math"
    )

    type Vertex struct {
        X, Y float64
    }

    func (v Vertex) Abs() float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func main() {
        v := Vertex{3, 4}
        fmt.Println(v.Abs())
    }

## 3.2 Methods are functions

Remember: 方法是带接收器参数的函数。

下面示例中Abs是一个正常的函数，功能上与之前方法没有区别：

    package main

    import (
        "fmt"
        "math"
    )

    type Vertex struct {
        X, Y float64
    }

    func Abs(v Vertex) float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func main() {
        v := Vertex{3, 4}
        fmt.Println(Abs(v))
    }


## 3.3 Method continued

也可以声明non-struct类型的方法。

以下示例中，是一个MyFloat数字类型的Abs方法。

可以只声明一个方法，接收器类型定义在与方法相同的包中。但不能声明一个方法，而接收器类型定义在别的包中(包括内置类型也不行，如int)。

    package main

    import (
        "fmt"
        "math"
    )

    type MyFloat float64

    func (f MyFloat) Abs() float64 {
        if f < 0 {
            return float64(-f)
        }
        return float64(f)
    }

    func main() {
        f := MyFloat(-math.Sqrt2)
        fmt.Println(f.Abs())
    }

## 3.4 Pointer receivers

可以声明带pointer接收器的方法。

这也意味着接收器类型有一个字面语法：对于类型T，是\*T。(T自己不能是指针，如\*int)

例如，Scale方法接收器类型是*Vertex。

带指针接收器的方法可以修改接收器指针的值。由于方法常常需要修改它们的接收器，因此指针接收器比值接收器更通用。

对于值接收器，Scale方法操作Vertex原始的副本。这个行为与其它函数参数行为一致。Scale方法必须有一个指针接收器，来改变声明在main函数中的Vertext的值。


    package main

    import (
        "fmt"
        "math"
    )

    type Vertex struct {
        X, Y float64
    }

    func (v Vertex) Abs() float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func (v *Vertex) Scale(f float64) {
        v.X = v.X * f
        v.Y = v.Y * f
    }

    func main() {
        v := Vertex{3, 4}
        v.Scale(10)
        fmt.Println(v.Abs())
    }


## 3.5 Pointers and functions

在这部分，我们看到Abs和Scale方法被重写成了函数。


    package main

    import (
        "fmt"
        "math"
    )

    type Vertex struct {
        X, Y float64
    }

    func Abs(v Vertex) float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func Scale(v *Vertex, f float64) {
        v.X = v.X * f
        v.Y = v.Y * f
    }

    func main() {
        v := Vertex{3, 4}
        Scale(&v, 10)
        fmt.Println(Abs(v))
    }

## 3.6 Methods and pointer indirection


与之前两段程序相比较，你可能注意到带指针参数的函数，必须接收指针。

    var v Vertex

    ScaleFunc(v) // Compile error
    ScaleFunc(&v) // OK

但是带指针接收器方法，既可以接收值，也可以接收指针：

    var v Vertex

    v.Scale(5) // OK
    p := &v
    p.Scale(10) // OK

对于v.Scale(5)，尽管v是一个值不是指针，带指针接收器的方法也是自动被调用。做为一种便利方式，Go把v.Scale(5)解析成(&v).Scale(5)，因为Scale方法拥有一个指针接收器。


    package main

    import "fmt"

    type Vertex struct {
        X, Y float64
    }

    func (v *Vertex) Scale(f float64) {
        v.X = v.X * f
        v.Y = v.Y * f
    }

    func ScaleFunc(v *Vertex, f float64) {
        v.X = v.X * f
        v.Y = v.Y * f
    }

    func main() {
        v := Vertex{3, 4}
        v.Scale(2)
        ScaleFunc(&v, 10)

        p := &Vertex{4, 3}
        p.Scale(3)
        ScaleFunc(p, 8)

        fmt.Println(v, p)
    }

## 3.7 Methods and pointer indirection (2)

同样的事情发生在相反的一面。

传入函数的参数必须是一个特定类型的值：

    var v Vertex
    
    fmt.Println(AbsFunc(v)) // OK
    fmt.Println(AbsFunc(&v)) // Compile error

带值接收器的方法，既可以是一个值也可以是一个指针：

    var v Vertex

    fmt.Println(v.Abs()) // OK
    p := &v

    fmt.Println(p.Abs()) // OK

在这种情况下，p.Abs()会被解析成(*p).Abs()。


    package main

    import (
        "fmt"
        "math"
    )

    type Vertex struct {
        X, Y float64
    }

    func (v Vertex) Abs() float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func AbsFunc(v Vertex) float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func main() {
        v := Vertex{3, 4}
        fmt.Println(v.Abs())
        fmt.Println(AbsFunc(v))

        p := &Vertex{4, 3}
        fmt.Println(p.Abs())
        fmt.Println(AbsFunc(*p))
    }


## 3.8 Choosing a value or pointer receiver

有两个原因要使用指针接收器：

* 为了方法可以修改接收器指针指向的值
* 避免在方法调用时复制值。如果接收器是一个大的struct结构，可能更高效。

在这个示例中，*Vertex类型的Scale和Abs，尽管Abs方法不需要修改它的接收器。

通常情况下，类型中所有的方法应该统一是值接收器或指针接收器，而不是二者混合。


    package main

    import (
        "fmt"
        "math"
    )

    type Vertex struct {
        X, Y float64
    }

    func (v *Vertex) Scale(f float64) {
        v.X = v.X * f
        v.Y = v.Y * f
    }

    func (v *Vertex) Abs() float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    func main() {
        v := &Vertex{3, 4}
        fmt.Printf("Before scaling: %+v, Abs: %v\n", v, v.Abs())
        v.Scale(5)
        fmt.Printf("After scaling: %+v, Abs: %v\n", v, v.Abs())
    }


## 3.9 Interfaces

接口类型是定义了一序列的方法签名。

接口类型的值可以持有任何实现这些方法的值。

> NOTE
> 
> 在示例的22行有一个错误。Vertex(值类型)不能实现Abser，因为Abs方法是以*Vertext(指针类型)定义的。


    package main

    import (
        "fmt"
        "math"
    )

    type Abser interface {
        Abs() float64
    }

    func main() {
        var a Abser
        f := MyFloat(-math.Sqrt2)
        v := Vertex{3, 4}

        a = f  // a MyFloat implements Abser
        a = &v // a *Vertex implements Abser

        // In the following line, v is a Vertex (not *Vertex)
        // and does NOT implement Abser.
        a = v

        fmt.Println(a.Abs())
    }

    type MyFloat float64

    func (f MyFloat) Abs() float64 {
        if f < 0 {
            return float64(-f)
        }
        return float64(f)
    }

    type Vertex struct {
        X, Y float64
    }

    func (v *Vertex) Abs() float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }



错误信息：

    tmp/sandbox350658444/main.go:22:4: cannot use v (type Vertex) as type Abser in assignment:
	Vertex does not implement Abser (Abs method has pointer receiver)


## 3.10 Interfaces are implemented implicitly

类型实现一个接口，就是实现它的方法。这儿没有明确的声明，没有"implements"关键字。

隐式接口从它的实现中解耦了接口定义，可以出现在任何包中而不需要预先设定。


    package main

    import "fmt"

    type I interface {
        M()
    }

    type T struct {
        S string
    }

    // This method means type T implements the interface I,
    // but we don't need to explicitly declare that it does so.
    func (t T) M() {
        fmt.Println(t.S)
    }

    func main() {
        var i I = T{"hello"}
        i.M()
    }


## 3.11 Interface values

下面会解析，接口值可以被认为是值元组或具体类型的元组。

    (value, type)

接口值持有一个特定底层具体类型的值。

调用接口值上的方法，将执行底层类型上的同名方法。


    package main

    import (
        "fmt"
        "math"
    )

    type I interface {
        M()
    }

    type T struct {
        S string
    }

    func (t *T) M() {
        fmt.Println(t.S)
    }

    type F float64

    func (f F) M() {
        fmt.Println(f)
    }

    func main() {
        var i I

        i = &T{"Hello"}
        describe(i)
        i.M()

        i = F(math.Pi)
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }


## 3.12 Interface values with nil underlying values

如果接口中具体值是nil，方法将用一个nil接收器调用。

在一些语言中，这将触发一个空指针异常，但是在Go中，通用的方法编写试是使用一个nil接收器来处理。

NOTE：接口值持有一个nil，但是具体自己可以是non-nil。


    package main

    import "fmt"

    type I interface {
        M()
    }

    type T struct {
        S string
    }

    func (t *T) M() {
        if t == nil {
            fmt.Println("<nil>")
            return
        }
        fmt.Println(t.S)
    }

    func main() {
        var i I

        var t *T
        i = t
        describe(i)
        i.M()

        i = &T{"hello"}
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }


## 3.13 Nil interface values

nil接口值可以持有值，也可以持有具体类型。

调用nil接口上的方法会有一个run-time error，因为接口元组内部没有类型来指定具体被调用的方法。


    package main

    import "fmt"

    type I interface {
        M()
    }

    func main() {
        var i I
        describe(i)
        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }


错误信息：

    (<nil>, <nil>)
    panic: runtime error: invalid memory address or nil pointer dereference
    [signal SIGSEGV: segmentation violation code=0xffffffff addr=0x0 pc=0xd4b87]
    
    goroutine 1 [running]:
    main.main()
    	/tmp/sandbox151068630/main.go:12 +0x27

## 3.14 The empty interface


不包含任何方法的接口类型被称为empty interface。

    interface{}

空接口可以持有任意类型的value。(每个类型至少实现零个方法。)

空接口被用来处理未知类型的值。例如，fmt.Print任意个interface{}类型的参数。

    package main

    import "fmt"

    func main() {
        var i interface{}
        describe(i)

        i = 42
        describe(i)

        i = "hello"
        describe(i)
    }

    func describe(i interface{}) {
        fmt.Printf("(%v, %T)\n", i, i)
    }


## 3.15 Type assertions


type assertion提供了访问接口类型的底层具体值的方式。

    t := i.(T)

这个语句段言接口值i持有的具体类型T，分配底层T的值给变量t。

如果i不持有T，该语句将产生一个panic。

为了测试是否接口值持有一个特定类型，type assertion可以返回两个值：底层的值和一个报告是否断言成功的一个boolean值。

    t, ok := i.(T)

如果i持有T，t将是底层的值，ok将是true。

否则，ok为false，t将是T类型的zero value，不会产生panic。

这与map的语法规则类似。


    package main

    import "fmt"

    func main() {
        var i interface{} = "hello"

        s := i.(string)
        fmt.Println(s)

        s, ok := i.(string)
        fmt.Println(s, ok)

        f, ok := i.(float64)
        fmt.Println(f, ok)

        f = i.(float64) // panic
        fmt.Println(f)
    }


## 3.16 Type switches

type switch是一个概念，允许多个type assertion串行执行。

类型切换和正常switch语句一样，但类型切换是针对类型的(不是值)，这些值是与接口值所持有的值类型进行比较。

    switch v := i.(type) {

    case T:
        // here v has type T
    case S:
        // here v has type S
    default:
        // no match; here v has the same type as i
    }

类型切换的声明与类型assertion语法是一样的：i.(T)，但是特定类型T是可以用关键字type替换的。

这个切换语句测试是否接口值i持有一个类型T或S的值。在T或S的情况中，变量v是T类型或S类型，并持有被i所持有的值。在默认情况下(如果不匹配)，变量v的类型与接口类型相同，值是i所持有的值。


    package main

    import "fmt"

    func do(i interface{}) {
        switch v := i.(type) {
        case int:
            fmt.Printf("Twice %v is %v\n", v, v*2)
        case string:
            fmt.Printf("%q is %v bytes long\n", v, len(v))
        default:
            fmt.Printf("I don't know about type %T!\n", v)
        }
    }

    func main() {
        do(21)
        do("hello")
        do(true)
    }

## 3.17 Stringers

最普遍存在的接口之一是定义在fmt包中的Stringer。

    type Stringer interface {

        String() string
    }

Stringer是可以把他自己描述成string的一种类型。fmt包(许多别的包)通过查找这个接口，来输出值。


    package main

    import "fmt"

    type Person struct {
        Name string
        Age  int
    }

    func (p Person) String() string {
        return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
    }

    func main() {
        a := Person{"Arthur Dent", 42}
        z := Person{"Zaphod Beeblebrox", 9001}
        fmt.Println(a, z)
    }


## 3.18 Errors

Go程序使用error值来表示error状态。

error类型是一个与fmt.Stringer类似的内置接口：

    type error interface {

        Error() string
    }

正如fmt.Stringer，在打印错误值时，fmt包会查找error接口。

函数常常返回error值，调用代码通过测试是否error是nil来处理错误。

    i, err := strconv.Atoi("42")
    if err != nil {

        fmt.Printf("couldn't convert number: %v\n", err)
        return
    }
    fmt.Println("Converted integer: ", i)

nil error表示成功；non-nil error表示失败。


    package main

    import (
        "fmt"
        "time"
    )

    type MyError struct {
        When time.Time
        What string
    }

    func (e *MyError) Error() string {
        return fmt.Sprintf("at %v, %s",
        e.When, e.What)
    }

    func run() error {
        return &MyError{
            time.Now(),
            "it didn't work",
        }
    }

    func main() {
        if err := run(); err != nil {
            fmt.Println(err)
        }
    }


## 3.19 Readers

io包描述了io.Reader接口，实现了读取数据流。

Go标准库包含了这个接口的[许多实现](https://golang.org/search?q=Read#Global)，包括文件，网络连接，压缩器(compressor)，加密(cipher)及其它。

io.Reader接口有一个Read方法：

    func (T) Read(b []byte) (n int, err error)

Read用数据填充给定的字节片，返回填充的字节数及error值。当流结束，它会返回一个io.EOF error。

下面的示例创建了一个strings.Reader，一次处理输出中的8个字节。


    package main

    import (
        "fmt"
        "io"
        "strings"
    )

    func main() {
        r := strings.NewReader("Hello, Reader!")

        b := make([]byte, 8)
        for {
            n, err := r.Read(b)
            fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
            fmt.Printf("b[:n] = %q\n", b[:n])
            if err == io.EOF {
                break
            }
        }
    }


## 3.20 Images

[image包](https://golang.org/pkg/image/#Image)定义了Image接口：

    package image

    type Image interface {

        ColorModel() color.Model
        Bounds() Rectangle
        At(x, y int) color.Color
    }


> NOTE
> 
> Bounds方法的Rectangle返回值实际上是一个image.Rectangle，是被定义在image包中的。


[更多细节，请查阅](https://golang.org/pkg/image/#Image)

color.Color和color.Model类型也是接口，但是我们会忽略它们，而使用预置实现：color.RGBA和color.RGBAModel。这些接口或类型由[image/color package](https://golang.org/pkg/image/color/)来定义的。


# 4. Concurrency

## 4.1 Goroutines

goroutine是一个由Go runtime管理的轻量级线程。

    go f(x, y, z)

开启一个新的goroutine

    f(x, y, z)

f, x, y和z的计算发生在当前goroutine，f的执行发生在新的goroutine中。

Goroutine运行在相同的地址空间，因此访问共享内存时需要同步。sync包提供了有用的原语，尽管在Go中有其它原语，大多数情况中你不需要这些原语。


    package main

    import (
        "fmt"
        "time"
    )

    func say(s string) {
        for i := 0; i < 5; i++ {
            time.Sleep(100 * time.Millisecond)
            fmt.Println(s)
        }
    }

    func main() {
        go say("world")
        say("hello")
    }


## 4.2 Channels

Channel是一个有类型导管，你可以管道操作符(<-)来发送或接收值。


    ch <- v // Send v to channel ch.
    v := <-ch // Receive from ch, and assign value to v

箭头方向表达了数据流的方向。

像map和slice一样，channel必须在使用前被创建：

    ch := make(chan int)

默认情况，发送与接收块等待直到另一端准备好。这允许goroutine之间同步，不需要明确的锁或条件变量。

示例代码sum在slice中的数字，在两个goroutine之间分配工作。一旦两个goroutine完成估算，它将计算最终的结果。


    package main

    import "fmt"

    func sum(s []int, c chan int) {
        sum := 0
        for _, v := range s {
            sum += v
        }
        c <- sum // send sum to c
    }

    func main() {
        s := []int{7, 2, 8, -9, 4, 0}

        c := make(chan int)
        go sum(s[:len(s)/2], c)
        go sum(s[len(s)/2:], c)
        x, y := <-c, <-c // receive from c

        fmt.Println(x, y, x+y)
    }


## 4.3 Buffered Channels

channel可以是buffered。可以通过make函数的第二个参数来指定buffer长度，初始化buffered channel：

    ch := make(chan int, 100)

只有当buffer满时，才会发送到buffered channel块。当buffer是空时，才会接收块。


    package main

    import "fmt"

    func main() {
        ch := make(chan int, 2)
        ch <- 1
        ch <- 2
        fmt.Println(<-ch)
        fmt.Println(<-ch)
    }


## 4.4 Range and Close

sender可以close通道，来指示没有更多的值要发送。Receiver可以通过在接收表达式上声明第二个参数，来测试是否channel被关闭。

    v, ok := <-ch

ok是false，表示没有更多的值被接收，并且channel也被关闭了。

for i := range c循环从通道中重复的接收值，只到它被关闭。

NOTE: 应该只有发送者来关闭通道，决不应该是接收者。发送到一个关闭的通道，将造成一个panic。

Another note：channel不像文件，你通常不需要关闭它。当接收者被告知这儿没有更多的值进来，关闭才是必要的，例如，为了终止一个range循环。


    package main

    import (
        "fmt"
    )

    func fibonacci(n int, c chan int) {
        x, y := 0, 1
        for i := 0; i < n; i++ {
            c <- x
            x, y = y, x+y
        }
        close(c)
    }

    func main() {
        c := make(chan int, 10)
        go fibonacci(cap(c), c)
        for i := range c {
            fmt.Println(i)
        }
    }


## 4.5 Select

select语句让goroutine在多个通讯操作上等待。

select块一直等到它的某个case可运行，然后它执行这个case。如果多个都ready，它将随机选择一个。


    package main

    import "fmt"

    func fibonacci(c, quit chan int) {
        x, y := 0, 1
        for {
            select {
            case c <- x:
                x, y = y, x+y
            case <-quit:
                fmt.Println("quit")
            return
            }
        }
    }

    func main() {
        c := make(chan int)
        quit := make(chan int)
        go func() {
            for i := 0; i < 10; i++ {
                fmt.Println(<-c)
            }
            quit <- 0
        }()
        fibonacci(c, quit)
    }

## 4.6 Default Selection

在select中如果没有别的case是ready，default将被执行。

使用default来尝试做无阻塞的发送或接收：

    select {
    case i := <-c:
        // use i
    default:
        // receiving from c would block
    }

示例代码：


    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        tick := time.Tick(100 * time.Millisecond)
        boom := time.After(500 * time.Millisecond)
        for {
            select {
            case <-tick:
                    fmt.Println("tick.")
            case <-boom:
                    fmt.Println("BOOM!")
                    return
            default:
                    fmt.Println("    .")
                    time.Sleep(50 * time.Millisecond)
            }
        }
    }


## 4.7 syc.Mutex

我们看到了channel是善长在goroutine之间进行通信的。

但是如果我不需要通信呢？我们只想保证在同一时间只有一个goroutine可以访问某个变量，避免冲突？

这个概念称为mutual exclusion(互相排它)，数据结构规范称为：mutex(互斥)。

Go标准库通过sync.Mutex实现了mutual exclusion，它有两个方法：

* Lock
* Unlock

我们可以定义一个代码块只能在mutual exclusion中被执行，通过在它外面增加Lock和Unlock，就像下面示例中Inc方法展示的那样。

我们也可以使用defer确保mutex将被unlocked，就像示例中Value方法展示的那样。


    package main

    import (
        "fmt"
        "sync"
        "time"
    )

    // SafeCounter is safe to use concurrently.
    type SafeCounter struct {
        v   map[string]int
        mux sync.Mutex
    }

    // Inc increments the counter for the given key.
    func (c *SafeCounter) Inc(key string) {
        c.mux.Lock()
        // Lock so only one goroutine at a time can access the map c.v.
        c.v[key]++
        c.mux.Unlock()
    }

    // Value returns the current value of the counter for the given key.
    func (c *SafeCounter) Value(key string) int {
        c.mux.Lock()
        // Lock so only one goroutine at a time can access the map c.v.
        defer c.mux.Unlock()
        return c.v[key]
    }

    func main() {
        c := SafeCounter{v: make(map[string]int)}
        for i := 0; i < 1000; i++ {
            go c.Inc("somekey")
        }

        time.Sleep(time.Second)
        fmt.Println(c.Value("somekey"))
    }


# 5. Where to Go from here ...

[安装Go](https://golang.org/dl/)

一旦你安装完Go，[Go的文档](https://golang.org/doc/)是继续学习的好地方。它包含references，tutorials，videos and more。

学习怎么样组织和开始写Go代码，看[录像](https://www.youtube.com/watch?v=XCsL89YtqCs)或读[How to Write Go Code](https://golang.org/doc/code.html)

如果你需要标准库的帮助，[请查看](https://golang.org/pkg/)。

如果是语言本身，可以查阅[语言规范](https://golang.org/ref/spec)。

进一步了解Go的并发模型，请看[Go并发模式](https://www.youtube.com/watch?v=f6kdp27TYZs)([slides](https://talks.golang.org/2012/concurrency.slide))、[高级Go并发模式](https://www.youtube.com/watch?v=QDDwwePbDtw)([slides](https://talks.golang.org/2013/advconc.slide))和[通过共享内存进行通信](https://golang.org/doc/codewalk/sharemem/)。

开始写web应用，请看[简单的编程环境](https://vimeo.com/53221558)([slides](https://talks.golang.org/2012/simple.slide))和读[写Web应用](https://golang.org/doc/articles/wiki/)的tutorial。

[First Class Functions in Go](https://golang.org/doc/codewalk/functions/)给出了一个关于Go函数类型的有趣观点。

[Go blog](https://blog.golang.org/)有大量的归档的Go文档。

更多的资料，请访问[golang.org](https://golang.org/)。




