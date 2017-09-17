# 1. 安装go

## 1.1 在Windows上安装go

通过msi安装。

> Note
> 
> Go默认工作目录为%USERPROFILE%\go

__Hello world__

* 在%USERPROFILE%目录下，创建go目录
* 在go目录下创建src，在src目录下创建hello目录
* 在hello目录下创建hello.go文件，放以下内容到文件中
    
        package main

        import "fmt"
    
        func main() {
            fmt.Printf("hello, world\n")
        }

* 通过go build，编译文件，将在本地目录中生成hello.exe文件
* 运行hello，将打印出"hello, world"。看到这句话，说明go安装成功
* 可以通过go install将hello.exe文件安装到src同级的bin目录中。

## 1.2 CentOS安装go

    wget https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz

默认位置安装:

* 解压文件
* 复制文件到/usr/local目录下
* 添加环境变量/etc/profile
        
        export PATH=$PATH:/usr/local/go/bin

在$HOME下创建go workspace：

    目录结构如下：
    $HOME/go/src/

参考1.1对环境进行测试。



# 2. 如何写go代码

## 2.1 概述

* go程序员把所有代码放在同一个workspace中
* workspace包含了多个版本控制的repository(例如，通过git管理)
* 每个repository包含1个或多个package
* 每个package由存储在目录中的一个或多个go源文件组成
* package目录的路径，指示了它的import路径

这与其它语言不同，在其它语言中每个工程有一个独立的workspace。workspace与版本控制的repository绑定在一起。

## 2.2 工作空间(Workspaces)

一个workspace由三个子目录组成:

* src，包含go源代码
* pkg，包含了包对象
* bin，包含了可执行命令

go工具构建源代码包，并把生成的二进制文件安装到pkg和bin目录中。


workspace示例：

    bin/
        hello                          # command executable
        outyet                         # command executable
    pkg/
        linux_amd64/
            github.com/golang/example/
                stringutil.a           # package object
    src/
        github.com/golang/example/
            .git/                      # Git repository metadata
            hello/
                hello.go               # command source
            outyet/
                 main.go                # command source
                 main_test.go           # test source
            stringutil/
                reverse.go             # package source
                reverse_test.go        # test source
        golang.org/x/image/
            .git/                      # Git repository metadata
            bmp/
                reader.go              # package source
                writer.go              # package source
        ... (many more repositories and packages omitted) ...

上例中显示该workspace包含了两个repository(examplet和image)。example repository包含了两个command(hell和outyet)和一个library(stringutil)。image repository包含了bmp package。

典型的workspace包含了许多的源代码repository，这些repository包含了许多package和command。大多数go程序员把所有的go源代码和依赖放在一个空间中。

Commond和library由不同的source package生成。

## 2.2 GOPATH环境变量

GOPATH指定了workspace的位置。默认是用户home目录中的go目录，Unix是$HOME/go，Plan 9是$home/go，Windows下是%USERPROFILE%\go。如果你想使用不同的workspace位置，可以通过设置GOPATH来实现。

## 2.3 import路径(Import paths)

import path，是一个字符串，唯一标识了一个包。package的import path对应它在workspace中或在远程仓库中的位置。

标准库中的包拥有非常短的import path，如"fmt"和"net/http"。用户必须为自己的包选择一个base path，他不应该与标准库的未来扩展包或其它第三方库冲突。

应该使用仓库名作为包名的一部分。如，你有一个github帐号github.com/user，应该使用它做为base path。

如，在workspace中创建目录，用于存储源代码：

    mkdir -p $GOPATH/src/github.com/user

## 2.4 第一个应用

    # 查看GOPATH环境变量

    go env GOPATH


    # Windows

    mkdir -p /home/liuyanfeng/go/src/github.com/user/hello

    cd /home/liuyanfeng/go/src/github.com/user/hello


hello.go中的代码：

    package main

    import "fmt"
    
    func main() {
    
        fmt.Printf("Hello, world.\n")
    }

使用go工具，编译并安装程序：

    # pwd - /home/liuyanfeng/go/src

    go install github.com/user/hello

> Note
> 
> 可以在系统的任何位置运行go install github.com/user/hello，go工具将在GOPATH指定的工作空间中查找github.com/user/hello包。
> 
> 如果在包目录中运行go install，可以忽略包路径：
> 
>     cd /home/liuyanfeng/go/src/github.com/user/hello
>     go install

go install将构建hello命令，并生成一个可执行二进制文件。然后将该二进制文件(hello，Windows中就是hello.exe)安装到workspace的bin目录中。在我们的例子中，就是$GOPATH/bin/hello，如果没有设置$GOPATH，也就是$HOME/go/bin/hello。

go工具错误发生时才打印输出，因此如果这些命令没有产生输出，则表示它们被执行成功。

现在可以通过输入命令的全路径来运行命令：

    # $HOME/go/bin/hello

    $GOPATH/bin/hello

或，如果你把$GOPATH/bin加入到PATH中，可以直接输入二进制文件名字：

    hello

如果你正使用源代码管理系统，现在正好可以初始化repository，add the files, and commit your first change。

    cd $HOME/go/src/github.com/user/hello
    git init
    git add .
    git commit -m "initial commit"
    git push origin develop


## 2.5 第一个library

我们写个库，然后在hello程序中使用它。

创建包空间目录，如: $HOME/src/github.com/user/stringutil

    mkdir $HOME/go/src/github.com/user/stringutil

创建一个reverse.go文件，文件中包含以下内容：

    // Package stringutil contains utility functions for woking with strings.
    package stringutil

    // Reverse returns its argument string reversed rune-wise left to right.
    func Reverse(s string) string {
    
        r := []rune(s)
        for i, j := 0, len(r) - 1; i < len(r) / 2; i, j = i + 1, j - 1 {
            r[i], r[j] = r[j], r[i]
        }
        
        return string(r)
    }

编译：

    go build github.com/user/stringutil
    go install github.com/user/stringutil

执行go install后，将会把生成的文件安装到工作空间的pkg目录中。


修改hello.go文件。

    vi $HOME/go/src/github.com/user/hello/hello.go

    package main

    import (
        "fmt"
        "github.com/user/stringutil"
    )
    
    func main() {
    
        fmt.Printf(stringutil.Reverse("!oG ,olleH"))
    }

编译：

    go install github.com/user/hello

输出结果：

    liuyanfeng@iZ259skw0ciZ:[/home/liuyanfeng/go/bin]./hello 
    Hello, Go!

在执行过上面的操作后，工作空间目录应该如下：

    liuyanfeng@iZ259skw0ciZ:[/home/liuyanfeng/go]tree
    .
    ├── bin
    │   └── hello
    ├── pkg
    │   └── linux_amd64
    │       └── github.com
    │           └── user
    │               └── stringutil.a
    └── src
        └── github.com
            └── user
                ├── hello
                │   └── hello.go
                └── stringutil
                    └── reverse.go
    
    10 directories, 4 files

> NOTE
> 
> go install将stringutil.a对象安装到pkg/linux_amd64目录下。

Go命令执行文件是被静态链接的；在Go运行时package对象不必存在。

## 2.6 包名(Package names)

在Go源文件第一个语句必须是：

    package name

name是导入时，包的默认名字。(在包中的所有文件必须使用相同的名字。)

Go中约定，包名是导入路径的最后一个元素：当"crypto/rot13"导入时的名字应该是rot13。

执行命令文件必须使用package main。

NOTE: 被链接到同一个二进制文件中，不要求包名是唯一的。但是导入路径(他们的全文件名)必须是唯一的。

更多命名规范，请查阅[https://golang.org/doc/effective_go.html#names](https://golang.org/doc/effective_go.html#names)

# X.

## X.1 rm -foo

    To remove a file whose name starts with a `-', for example `-foo',
    use one of these commands:
        rm -- -foo
    
        rm ./-foo