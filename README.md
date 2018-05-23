# iparhan_beego
下面介绍Beego项目的创建和初步运用
创建项目
运行项目
创建项目
beego 的项目基本都是通过 bee 命令来创建的，所以在创建项目之前确保你已经安装了 bee 工具和 beego。如果你还没有安装，那么请查阅 beego 的安装 和 bee 工具的安装。

现在一切就绪我们就可以开始创建项目了，打开终端，进入 $GOPATH/src 所在的目录：

➜  src  bee new quickstart
[INFO] Creating application...
/gopath/src/quickstart/
/gopath/src/quickstart/conf/
/gopath/src/quickstart/controllers/
/gopath/src/quickstart/models/
/gopath/src/quickstart/routers/
/gopath/src/quickstart/tests/
/gopath/src/quickstart/static/
/gopath/src/quickstart/static/js/
/gopath/src/quickstart/static/css/
/gopath/src/quickstart/static/img/
/gopath/src/quickstart/views/
/gopath/src/quickstart/conf/app.conf
/gopath/src/quickstart/controllers/default.go
/gopath/src/quickstart/views/index.tpl
/gopath/src/quickstart/routers/router.go
/gopath/src/quickstart/tests/default_test.go
/gopath/src/quickstart/main.go
2014/11/06 18:17:09 [SUCC] New application successfully created!
通过一个简单的命令就创建了一个 beego 项目。他的目录结构如下所示

quickstart
|-- conf
|   `-- app.conf
|-- controllers
|   `-- default.go
|-- main.go
|-- models
|-- routers
|   `-- router.go
|-- static
|   |-- css
|   |-- img
|   `-- js
|-- tests
|   `-- default_test.go
`-- views
    `-- index.tpl	
从目录结构中我们也可以看出来这是一个典型的MVC架构的应用，main.go 是入口文件。

运行项目
beego 项目创建之后，我们就开始运行项目，首先进入创建的项目，我们使用 bee run 来运行该项目，这样就可以做到热编译的效果：

➜  src  cd quickstart
➜  quickstart  bee run
2014/11/06 18:18:34 [INFO] Uses 'quickstart' as 'appname'
2014/11/06 18:18:34 [INFO] Initializing watcher...
2014/11/06 18:18:34 [TRAC] Directory(/gopath/src/quickstart/controllers)
2014/11/06 18:18:34 [TRAC] Directory(/gopath/src/quickstart)
2014/11/06 18:18:34 [TRAC] Directory(/gopath/src/quickstart/routers)
2014/11/06 18:18:34 [TRAC] Directory(/gopath/src/quickstart/tests)
2014/11/06 18:18:34 [INFO] Start building...
2014/11/06 18:18:35 [SUCC] Build was successful
2014/11/06 18:18:35 [INFO] Restarting quickstart ...
2014/11/06 18:18:35 [INFO] ./quickstart is running...
2014/11/06 18:18:35 [app.go:96] [I] http server Running on :8080
这样我们的应用已经在 8080 端口(beego 的默认端口)跑起来了.你是不是觉得很神奇，为什么没有 nginx 和 apache 居然可以自己干这个事情？是的，Go 其实已经做了网络层的东西，beego 只是封装了一下，所以可以做到不需要 nginx 和 apache。让我们打开浏览器看看效果吧：



你内心是否激动了？开发网站如此简单有没有。好了，接下来让我们一层一层的剥离来大概的了解 beego 是怎么运行起来的。


第二个----路由设置
项目路由设置
项目路由设置
前面我们已经创建了 beego 项目，而且我们也看到他已经运行起来了，那么是如何运行起来的呢？让我们从入口文件先分析起来吧：

package main

import (
	_ "quickstart/routers"
	"github.com/astaxie/beego"
)

func main() {
	beego.Run()
}
我们看到main函数是入口函数，但是我们知道Go的执行过程是如下图所示的方式：



这里我们就看到了我们引入了一个包_ "quickstart/routers",这个包只引入执行了里面的init函数，那么让我们看看这个里面做了什么事情：

package routers

import (
	"quickstart/controllers"
	"github.com/astaxie/beego"
)

func init() {
    beego.Router("/", &controllers.MainController{})
}

路由包里面我们看到执行了路由注册beego.Router, 这个函数的功能是映射URL到controller，第一个参数是URL(用户请求的地址)，这里我们注册的是 /，也就是我们访问的不带任何参数的URL，第二个参数是对应的 Controller，也就是我们即将把请求分发到那个控制器来执行相应的逻辑，我们可以执行类似的方式注册如下路由：

beego.Router("/user", &controllers.UserController{})	
这样用户就可以通过访问 /user 去执行 UserController 的逻辑。这就是我们所谓的路由，更多更复杂的路由规则请查询 beego 的路由设置

再回来看看main函数里面的 beego.Run， beego.Run 执行之后，我们看到的效果好像只是监听服务端口这个过程，但是它内部做了很多事情：

解析配置文件

beego 会自动在 conf 目录下面去解析相应的配置文件 app.conf，这样就可以通过配置文件配置一些例如开启的端口，是否开启 session，应用名称等各种信息。

执行用户的hookfunc

beego会执行用户注册的hookfunc，默认的已经存在了注册mime，用户可以通过函数AddAPPStartHook注册自己的启动函数。

是否开启 session

会根据上面配置文件的分析之后判断是否开启 session，如果开启的话就初始化全局的 session。

是否编译模板

beego 会在启动的时候根据配置把 views 目录下的所有模板进行预编译，然后存在 map 里面，这样可以有效的提高模板运行的效率，无需进行多次编译。

是否开启文档功能

根据EnableDocs配置判断是否开启内置的文档路由功能

是否启动管理模块

beego 目前做了一个很帅的模块，应用内监控模块，会在 8088 端口做一个内部监听，我们可以通过这个端口查询到 QPS、CPU、内存、GC、goroutine、thread 等各种信息。

监听服务端口

这是最后一步也就是我们看到的访问 8080 看到的网页端口，内部其实调用了 ListenAndServe，充分利用了 goroutine 的优势

一旦 run 起来之后，我们的服务就监听在两个端口了，一个服务端口 8080 作为对外服务，另一个 8088 端口实行对内监控。

通过这个代码的分析我们了解了 beego 运行起来的过程，以及内部的一些机制。接下来让我们去剥离 Controller 如何来处理逻辑的。
