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

## 2.4 


