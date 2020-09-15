## go环境和基础知识

    |--------------------------------------------
    
     Author:       intro                        
     CreateAt:     2020-09-15 23:51:43        
      
    |--------------------------------------------



>go mod 是什么?它的出现是为了解决什么问题？

我们知道在1.12之前的版本里，我们项目代码只能放在 $GOPATH/src目录下。包括go get 回来的第三方包。比如我们看到的  gitbub.com 目录。这里有提到 \$GOPATH

有必要理解一下，\$GOPATH 和​ $GOROOT.

#### GOROOT

GOROOT就是go的安装路径, go的可执行目录。如 go命令，即在\$GOROOT/bin 目录下。类似的还有 godoc,gofmt等，所以,第一步，需要将GOROOT/bin添加到环境变量中。

#### GOPATH

$GOPATH 目录主要用于存放用户相关的文件。如 go get 回来的包。

GOPATH之下主要包含三个目录: bin、pkg、src
bin目录主要存放可执行文件; pkg目录存放编译好的库文件, 主要是*.a文件; src目录下主要存放go的源文件



知道了GOPATH的作用后，再了解一下，go 的 **import**

go文件中 通过import来“导入一个包 ”。注意这里说的是导入一个“**包**”，这里通常会让人容易理解错误。

import **？**  这个 **？**到底是什么？

为了得出这个答案，下面写一点代码测试一下

在 GOPATH/src 目录下创建 testpath 项目

testpath
├── abc
│   ├── abc.go
│   └── api
│       └── api.go
└── api
    └── main.go

其中 main.go 的代码为：

```go
package main


import (
	"testpath/abc"
	api123 "testpath/abc/api"
)

func main()  {
	api123.TestApi()
	abc.TestAbc()
}
```



abc.go 的代码为：

```go
package abc

func TestAbc()  {

}

```

api.go的代码为：

```go
package api123

func TestApi()  {

}
```



从上面 main.go 文件的 import 代码可以看出。 "testpath/abc/api" 对应的，并不是 pachage api 的包。而是 目录。

也就是说，**import 导入的是目录**。

其中 	api123 "testpath/abc/api"   api123只是一个别名，可以换成 api12345或别的，只要不与已经 import 进的目录下存在同名包即可。

另一个问题。。我们使用包的时候，是用的 package 里定义的名称，还是目录名？

我们把 abc.go 文件改一下：

```go
package abcd

func TestAbc()  {

}

```

发现 main.go 文件里想要正常引用也必须改成：

```go
package main

import (
	"testpath/abc"
	api123 "testpath/abc/api"
)

func main()  {
	api123.TestApi()
	abcd.TestAbc()
}

```

所以，我们知道这里引用的是 **package 定义的名称**，并非目录名。

到这里，又会产生另一个疑问：

> 那别名，到底是给目录取的，还是package 定义的名称的别名呢？

我们不妨再把main.go 文件里的 **api123 "testpath/abc/api"** 改成 **api1234 "testpath/abc/api"**

这时候发现，要想正常的引用，main 方法将必须改成：

```go
package main

import (
	"testpath/abc"
	api1234 "testpath/abc/api"
)

func main()  {
	api1234.TestApi()
	abcd.TestAbc()
}

```

由此是可以看出，别名，**不是针对的目录，而是目录下 package 定义的包名**。



同一个目录下，同级只能有一个包名。不能存在多个。

这里之所以会有一些坑，而平常都不太注意，是因为我们默认都是包名与目录同名，甚至与文件同名。这里需要理解的是：

> go 的 import，是导入一个目录，引用的是目录里定义的包名。一个包，可以拆分成**同级目录**下多个不同名称的文件。



ok,到这里基本都搞清楚了 import 是怎么工作的了。细心的你会发现。import的目录，是和项目同名的目录开始的，这是巧合，还是别的规则呢？

其实，import 导入的目录，分两种，一个是系统包，如“os,fmt,time”等。另一些是用户自定义的，如项目内的包和第三方包。

系统的包在：$GOROOT/src 目录下

第三方或用户自定义的包在：$GOPATH/src 下。

也就是说，import 目录时，先开始在 $GOROOT/src目录下找，如果找不到对应的目录，再会到 \$GOPATH/src 下来找。如果还是找不到，就会报错。

上面那个问题，我们项目放在了$GOPATH/src下，所以，import 的时候就要从 项目名（其实也是一个目录）开始引入项目内的包了。第三方如“github.com/aiqiu506/x”，也指的是**\$GOPATH/src/github.com/aiqiu506/x**目录



讲了一大堆，铺垫了那么长，终于到我们的主角 go mod 了。

#### go mod

go mod ，指的是 go module。

再介绍go mod 之前，可以发现之前的模式，存在一定的问题，第一，项目代码必须放在**\$GOPATH/src** 下，不自由。其实这还能忍，主要的是版本的问题。如果我 A 项目用的是 B 包的 v1.0.0版本，而 C项目用的是B 包的 v1.0.1版本，之前的模式就出问题了。而 go mod 的出现，就是为了解决这个问题。

且看它是如何解决这个问题的。

首先，要使用 go mod 之前需要先指定一个环境变量：**GO111MODULE**，设置`GO111MODULE=on` 即，我们说的开启 go module.

默认GO111MODULE设置的是`GO111MODULE=auto`，这个意思是说，**\$GOPATH/src** 下的项目，将默认不启用 go mod 模式。如果不在**\$GOPATH/src** 下创建项目，这个变量其实也可以不设置。

通过：`go mod init 项目名` 来初始化一个 go mod 项目。这个命令会在当前目录下创建 **项目名**的目录，并且在目录里创建 go.mod 文件。

如： `go mod init testmod`  （当前目录不在**\$GOPATH/src** 下）

下 testmod 目录下的go.mod 文件内容如下：

```go
module testmod

go 1.12

```

为项目添加了几个文件，目录文件如下：

```
testmod
├── go.mod
├── main
│   └── main.go
└── pkgs
    └── pkg.go

```

main.go 文件内容如下：

```go
package main

import "testmod/pkgs"

func main()  {

	pkgs.TestPkg()
}

```

pkg.go文件内容如下：

```go
package pkgs

func TestPkg()  {

}
```

项目可以跑起来。说明：

1、所有自定义包和第三方包都需要放在 **\$GOPATH/src**下的带来的第一个问题，已经解决。

2、那使用了 go mod 的模式，如何 import 文件的呢？是不是还是需要将包下载到**\$GOPATH/src** 下？

第二个问题。其实go mod 是这么解决的。

go1.12 会在用户的 $home 目录下创建一个 go 文件夹作为默认的 GOPATH。（可以不显示地定义 gopath）

使用 go mod 的开发，与之前并没什么不同，同样使用 go get 来下载第三方包。

而开启了 go mod 后，go get 会将远程的软件包下载到 **$home/go/pkg/mod** 目录里。注意不再是GOPATH/src下了。

到这里是不是会想，不就是换了一个目录放吗？那该存在的版本问题，还是没有解决呀。别急，往下看。

我们往 main.go 里添加点东西：

```go
package main

import (
	"testmod/pkgs"
	"github.com/aiqiu506/x"
)

func main()  {
  pkgs.TestPkg()
	app:=X.NewX("config.yaml")
	app.Start()

}


```

执行 go run main.go 的时候，会自动下载所依赖的包（稍后分析下载过程）

这个时候会发现go.mod 文件里内容自动变了：

```go
module testmod

go 1.12

require github.com/aiqiu506/x v0.0.0-20200618004349-df3a0a36a293 // indirect

```

然后在 **$home/go/pkg/mod**目录下，多了github.com/aiqiu506/x@v0.0.0-20200618004349-df3a0a36a293 目录。

这个目录名怎么搞这么长？因为加了版本号

也就是说，我们可以通过指定 go.mod 里 对应包的版本号，来下载对应的包。多个项目依赖同一个包的不同版本的问题，也完美地解决了。

#### 使用 go mod 模式，执行go get 时的过程

在执行go run main.go 后，首先会去检查依赖，导入需要的包。如同 go get 一样。

 检测到import 语句。并尝试根据 go.mod 的依赖引用关系导入三方包。如果发现本地cache没有，就会从远程拉取。就像是 go get。当 go module下载了远程包后，同时会自动更新 go.mod 。

#### 包版本的问题

go.mod 文件中的 require，必须加入版本号。那这个版本号是什么？

V0.0.0 这个一个默认版本格式。通常 go的版本是 v1.0.0开始的。据说是因为很多包永远只有一个版本。这三段分别表达的是

- MAJOR：主版本，通常是大版本升级，导致向前不兼容

- MINOR：次版本，通常是向下兼容的 feture

- PATCH：修订版本，如一些bugfix。

  

如果我们包代码托管在 github 上。那么 git的 tag，将可以成为 包的版本号。如果没有打 tag时。这个版本号将是什么？像这样`v0.0.0-20200618004349-df3a0a36a293`  第一段 v0.0.0就是一个默认版本号，第三段，是最后一次提交的时间，第三段，是最后提交的 git版本。



#### 本地包引用问题

使用了 go mod 后，会有一个新的问题。原来不管多少项目，都放在 gopath 下，可以跨项目调用。现在反而不进了。那针对这个问题，是如何解决的呢？

```go
module testmod

go 1.12

require github.com/aiqiu506/x v0.0.0-20200618004349-df3a0a36a293 // indirect

require moul.io/http2curl v0.0.0

replace moul.io/http2curl v0.0.0 => ../../go/src/moul.io/http2curl
```

其中 replace 就是干这个的。

> replace	目标目录 版本号 =>源目录

这个地方要注意，只能引入同样是 go mod 模式的包。而且源目录，是相对于当前项目的。

目标目录，必须是 源目录下 **go.mod 定义的 module 名称**，版本可以随便，只要符合格式。

require 时 与` 目标目录 版本号` 保持一致即可。







参考：https://www.jianshu.com/p/07ffc5827b26