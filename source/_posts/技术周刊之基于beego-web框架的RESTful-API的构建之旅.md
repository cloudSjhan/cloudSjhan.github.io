---
title: 技术周刊之基于beego web框架的RESTful API的构建之旅
tags: [golang beego]
copyright: true
date: 2018-10-14 15:44:03
permalink:
categories: 技术周刊
description: 本文介绍通过使用golang web开发框架beego搭建RESTFUL风格的API
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 前言

​	beego是一个快速开发GO应用的http框架，作者是go语言方向的大牛，astaxie。beego可以用来快速开发API、web、后端服务等应用，是一个RESTFul风格的框架，主要的设计灵感来自于Python web开发框架tornado、flask、sinstra，很好的结合了Go语言本身的一些特性（interface，struct继承等）。

​	beego是基于八大独立模块来实现的，很好的实现了模块间的解耦，即使用户不使用http的逻辑，也可以很好的使用其中的各个模块。作者自己说，他的这种思想来自于乐高积木，设计beego的时候，这些模块就是积木，而最终搭建好的机器人就是beego。

​	这篇博文通过使用beego来构建API，讲解实现过程中的细节以及遇到的一些坑，让我们马上开始beego的API构建之旅吧！

### 项目创建

- 进入到你的$GOPATH/src
- 安装beego开发包自己快速开发工具bee

```go
go get github.com/astaxie/beego
go get github.com/astaxie/beego/orm
go get github.com/beego/bee
```

- 使用快速开发工具bee，创建我们的API项目

```go
bee new firstAPI
```

我们得到的项目结构如下图所示：

![](https://ws4.sinaimg.cn/large/006tNbRwly1fw7uxgc1sqj314q03gwev.jpg)

可以看出这是一个典型的MVC架构的应用，beego把我们项目所需要的一些都准备好了，例如配置文件conf，测试文件tests等，我们只需要专注于API代码的编写即可。

### 运行项目并获得API自动化文档

```go
bee run -gendoc=true -downdoc=true
```

运行上述代码输出如下图所示：

![](https://ws1.sinaimg.cn/large/006tNbRwly1fw7v5dfquwj31kw0skn3a.jpg)

我们在浏览器中访问：本机IP：8080/swagger，就会看到swagger的API文档，我们代码更新后，该文档就会自动更新，非常方便。

### models设计

- 对 数据库object 操作有四个方法 Read / Insert / Update / Delete

```go
示例代码：
o := orm.NewOrm()
user := new(User)
user.Name = "slene"

fmt.Println(o.Insert(user))

user.Name = "Your"
fmt.Println(o.Update(user))
fmt.Println(o.Read(user))
fmt.Println(o.Delete(user))
```

还有其他的方法可以参阅beego[官方文档](https://beego.me/docs/mvc/model/object.md)，里面对orm操作有着详细的介绍。

- 创建一个数据库并设计一张数据库表

```mysql
CREATE TABLE IF NOT EXISTS `student` (
`Id` int(11),
`Name` varchar(255),
`Birthdate` varchar(255),
`Gender` bool,
`Score` int(11)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

- 在models文件夹下新建一个文件Student.go,并实现以下代码，代码中关键点都有注释

```go
package models

import (
	"fmt"
	"github.com/astaxie/beego/orm"
)


//在models模块中创建一个struct，目的是使用beego的orm框架，使struct与数据库中的字段产生对应关系
type Student struct {
	Id int`orm:"column(Id)"` //column()括号中的字段就是在定义数据库时的相应字段，这一段必须严格填写，不然在API读写数据时就会出现读不到或者写不进去的问题
	Name string  `orm:"column(Name)"`
	BirthDate string `orm:"column(Birthdate)"`
	Gender bool `orm:"column(Gender)"`
	Score int `orm:"column(Score)"`
}


//该函数获得数据库中所有student的信息，返回值是一个结构体数组指针
func GetAllStudents() []*Student {
	o := orm.NewOrm() //产生一个orm对象
	o.Using("default") //这句话的意思是使用定义的默认数据库，与main.go中的orm.RegisterDataBase()对应
	var students []*Student //定义指向结构体数组的指针
	q := o.QueryTable("student")//获得一个数据库表的请求
	q.All(&students)//取到这个表中的所有数据

	return students

}


//该函数根据student中的Id，返回该学生的信息
func GetStudentById(id int) Student {
	u := Student{Id:id}//根据所传入的Id得到对应student的对象
	o := orm.NewOrm()//new 一个orm对象
	o.Using("default")//使用最开始定义的default数据库
	err := o.Read(&u)//读取Id=id的student的信息

	if err == orm.ErrNoRows {
		fmt.Println("查询不到")//对应操作，不一定是print
	} else if err == orm.ErrMissPK {
		fmt.Println("没有主键")
	}

	return u
}


//添加一个学生的信息到数据库中，参数是指向student结构题的指针
func AddStudent(student *Student) Student {
	o := orm.NewOrm()
	o.Using("default")
	o.Insert(student)//插入数据库

	return *student
}

func UpdateStudent(student *Student) {
	o := orm.NewOrm()
	o.Using("default")
	o.Update(student)//更新该student的信息
}

func DeleteStudent(id int) {
	o := orm.NewOrm()
	o.Using("default")
	o.Delete(&Student{Id:id})//删除对应id的student的信息
}

func init()  {
	orm.RegisterModel(new(Student))//将数据库注册到orm
}
```

- model这一层主要是定义struct，并为上层编写读写数据库。处理数据的代码。

### controller层实现

基于 beego 的 Controller 设计，只需要匿名组合 `beego.Controller` 就可以了，如下所示：

```go
type xxxController struct {
    beego.Controller
}
```

`beego.Controller` 实现了接口 `beego.ControllerInterface`，`beego.ControllerInterface` 定义了如下函数：

- Init(ct *context.Context, childName string, app interface{})

  这个函数主要初始化了 Context、相应的 Controller 名称，模板名，初始化模板参数的容器 Data，app 即为当前执行的 Controller 的 reflecttype，这个 app 可以用来执行子类的方法。

- Prepare()

  这个函数主要是为了用户扩展用的，这个函数会在下面定义的这些 Method 方法之前执行，用户可以重写这个函数实现类似用户验证之类。

- Get()

  如果用户请求的 HTTP Method 是 GET，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Get 请求。

- Post()

  如果用户请求的 HTTP Method 是 POST，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Post 请求。

- Delete()

  如果用户请求的 HTTP Method 是 DELETE，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Delete 请求。

- Put()

  如果用户请求的 HTTP Method 是 PUT，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Put 请求.

- Head()

  如果用户请求的 HTTP Method 是 HEAD，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Head 请求。

- Patch()

  如果用户请求的 HTTP Method 是 PATCH，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Patch 请求.

- Options()

  如果用户请求的HTTP Method是OPTIONS，那么就执行该函数，默认是 405，用户继承的子 struct 中可以实现了该方法以处理 Options 请求。

- Finish()

  这个函数是在执行完相应的 HTTP Method 方法之后执行的，默认是空，用户可以在子 struct 中重写这个函数，执行例如数据库关闭，清理数据之类的工作。

- Render() error

  这个函数主要用来实现渲染模板，如果 beego.AutoRender 为 true 的情况下才会执行。

所以通过子 struct 的方法重写，用户就可以实现自己的逻辑。

### routers层实现

什么是路由设置呢？前面介绍的 MVC 结构执行时，介绍过 beego 存在三种方式的路由:固定路由、正则路由、自动路由，与RESTFul API相关的就是固定路由和正则路由。

下面就是固定路由的例子

```go
beego.Router("/", &controllers.MainController{})
beego.Router("/admin", &admin.UserController{})
beego.Router("/admin/index", &admin.ArticleController{})
beego.Router("/admin/addpkg", &admin.AddController{})
```

下面是正则路由的例子：

- beego.Router(“/api/?:id”, &controllers.RController{})

  默认匹配 //例如对于URL”/api/123”可以匹配成功，此时变量”:id”值为”123”

- beego.Router(“/api/:id”, &controllers.RController{})

  默认匹配 //例如对于URL”/api/123”可以匹配成功，此时变量”:id”值为”123”，但URL”/api/“匹配失败

- beego.Router(“/api/:id([0-9]+)“, &controllers.RController{})

  自定义正则匹配 //例如对于URL”/api/123”可以匹配成功，此时变量”:id”值为”123”

- beego.Router(“/user/:username([\\w]+)“, &controllers.RController{})

  正则字符串匹配 //例如对于URL”/user/astaxie”可以匹配成功，此时变量”:username”值为”astaxie”

- beego.Router(“/download/*.*”, &controllers.RController{})

  *匹配方式 //例如对于URL”/download/file/api.xml”可以匹配成功，此时变量”:path”值为”file/api”， “:ext”值为”xml”

- beego.Router(“/download/ceshi/*“, &controllers.RController{})

  *全匹配方式 //例如对于URL”/download/ceshi/file/api.json”可以匹配成功，此时变量”:splat”值为”file/api.json”

- beego.Router(“/:id:int”, &controllers.RController{})

  int 类型设置方式，匹配 :id为int 类型，框架帮你实现了正则 ([0-9]+)

- beego.Router(“/:hi:string”, &controllers.RController{})

  string 类型设置方式，匹配 :hi 为 string 类型。框架帮你实现了正则 ([\w]+)

- beego.Router(“/cms_:id([0-9]+).html”, &controllers.CmsController{})

  带有前缀的自定义正则 //匹配 :id 为正则类型。匹配 cms_123.html 这样的 url :id = 123

个人觉得，最方便的还是类似于Python框架flask的注解路由，也是在这个项目中使用的：

- 在routers/routers.go里面添加你所希望的API

- ```go
  package routers
  
  import (
  	"firstAPI/controllers"
  
  	"github.com/astaxie/beego"
  )
  
  func init() {
  	ns := beego.NewNamespace("/v1",
  		beego.NSNamespace("/object",
  			beego.NSInclude(
  				&controllers.ObjectController{},
  			),
  		),
  		beego.NSNamespace("/user",
  			beego.NSInclude(
  				&controllers.UserController{},
  			),
  		),
  		beego.NSNamespace("/student",
  			beego.NSInclude(
  				&controllers.StudentController{},
  			),
  		),
  	)
  	beego.AddNamespace(ns)
  }
  
  ```


以上代码实现了如下的API：

/v1/object

/v1/user

/v1/student

非常清晰明了。

### main.go的数据库配置

```go
package main

import (
	_ "firstAPI/routers"
	"github.com/astaxie/beego"
	"github.com/astaxie/beego/orm"
	_ "github.com/go-sql-driver/mysql"
)
func init() {
	orm.RegisterDriver("mysql", orm.DRMySQL)//注册MySQL的driver
	orm.RegisterDataBase("default", "mysql", "root:test@tcp(127.0.0.1:3306)/restapi_test?charset=utf8")//本地数据库的账号。密码等
	orm.RunSyncdb("default", false, true)

}
func main() {

	if beego.BConfig.RunMode == "dev" {
		beego.BConfig.WebConfig.DirectoryIndex = true
		beego.BConfig.WebConfig.StaticDir["/swagger"] = "swagger"//静态文档
	}

	beego.Run()
}

```

关键点都在代码中以注释的形式展现。

### postman测试

bee run 运行代码后，我们使用postman测试一下我们所构建的API效果如何。

![](https://ws1.sinaimg.cn/large/006tNbRwly1fw7w1tcnesj31kw0y90y6.jpg)

这里节省篇幅，只测试一个接口。

到此为止，我们基于beego就实现了简单API接口的构建，是不是既清晰又简单呢？赶快自己动手试试吧！

本期技术周刊结束，代码已上传到[GitHub](https://github.com/hantmac/beego_api_demo)，可以查阅，我们下期再会！

<hr />
