---
title: 关于 golang 错误处理的一些思考
tags: [golang]
copyright: true
date: 2020-06-01 15:03:21
permalink:
categories: golang
description: 关于 golang 错误处理的一些思考
image:
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

> 写在前面：你是还没在error上栽跟头，当你栽了跟头时才会哭着想起来，当年为什么没好好思考和反省错误处理这么一个宏大的话题

## 关于 Golang 错误处理的实践

> Golang有很多优点，这也是它如此流行的主要原因。但是 Go 1 对错误处理的支持过于简单了，以至于日常开发中会有诸多不便利，遭到很多开发者的吐槽。
这些不足催生了一些开源解决方案。与此同时， Go 官方也在从语言和标准库层面作出改进。
这篇文章将给出几种常见创建错误的方式并分析一些常见问题，对比各种解决方案，并展示了迄今为止(go 1.13)的最佳实践。

## 几种创建错误的方式
> 首先介绍几种常见的创建错误的方法

1. 基于字符串的错误

```go

err1 := errors.New("math: square root of negative number")

err2 := fmt.Errorf("math: square root of negative number %g", x)
```
2. 带有数据的自定义错误
```go 
package zError

import (
	"fmt"
	"github.com/satori/go.uuid"
	"log"
	"runtime/debug"
	"time"
)

type BaseError struct {
	InnerError error
	Message    string
	StackTrace string
	Misc       map[string]interface{}
}

func WrapError(err error, message string, messageArgs ...interface{}) BaseError {
	return BaseError{
		InnerError: err,
		Message:    fmt.Sprintf(message, messageArgs),
		StackTrace: string(debug.Stack()),
		Misc:       make(map[string]interface{}),
	}
}

func (err *BaseError) Error() string {
// 实现 Error 接口
	return err.Message
}
	
```
## 抛出问题
> 开发中经常需要检查返回的错误值并作相应处理。下面给出一个最简单的示例。

```go
import (
   "database/sql"
   "fmt"
)

func GetSql() error {
   return sql.ErrNoRows
}

func Call() error {
   return GetSql()
}

func main() {
   err := Call()
   if err != nil {
      fmt.Printf("got err, %+v\n", err)
   }
}
//Outputs:
// got err, sql: no rows in result set
```

有时需要根据返回的error类型作不同处理，例如：

```go

import (
   "database/sql"
   "fmt"
)

func GetSql() error {
   return sql.ErrNoRows
}

func Call() error {
   return GetSql()
}

func main() {
   err := Call()
   if err == sql.ErrNoRows {
      fmt.Printf("data not found, %+v\n", err)
      return
   }
   if err != nil {
      // Unknown error
   }
}
//Outputs:
// data not found, sql: no rows in result set
```

实践中经常需要为错误增加上下文信息后再返回，以方便调用者了解错误场景。例如 Getcall 方法时常写成：
```go
func Getcall() error {
   return fmt.Errorf("GetSql err, %v", sql.ErrNoRows)
}
```

不过这个时候 `err == sql.ErrNoRows` 就不成立了。除此之外，上述写法都在返回错误时都丢掉了调用栈这个重要的信息。我们需要更灵活、更通用的方式来应对此类问题。

## 解决方案
>针对存在的不足，目前有几种解决方案。这些方式可以对错误进行上下文包装，并携带原始错误信息， 还能尽量保留完整的调用栈

### 方案 1： github.com/pkg/errors

如果只有错误的文本，我们很难定位到具体的出错地点。虽然通过在代码中搜索错误文本也是有可能找到出错地点的，但是信息有限。所以，在实践中，我们往往会将出错时的调用栈信息也附加上去。调用栈对消费方是没有意义的，从隔离和自治的角度来看，消费方唯一需要关心的就是错误文本和错误类型。调用栈对实现者自身才是是有价值的。所以，如果一个方法需要返回错误，我们一般会使用errors.WithStack(err)或者errors.Wrap(err, "custom message")的方式，把此刻的调用栈加到error里去，并且在某个统一地方记录日志，方便开发者快速定位问题。

1. Wrap 方法用来包装底层错误，增加上下文文本信息并附加调用栈。 一般用于包装对第三方代码（标准库或第三方库）的调用。
2. WithMessage 方法仅增加上下文文本信息，不附加调用栈。 如果确定错误已被 Wrap 过或不关心调用栈，可以使用此方法。 注意：不要反复 Wrap ，会导致调用栈重复
3. Cause方法用来判断底层错误 。

现在我们用这三个方法来重写上面的代码：

```go 
import (
   "database/sql"
   "fmt"

   "github.com/pkg/errors"
)

func GetSql() error {
   return errors.Wrap(sql.ErrNoRows, "GetSql failed")
}

func Call() error {
   return errors.WithMessage(GetSql(), "bar failed")
}

func main() {
   err := Call()
   if errors.Cause(err) == sql.ErrNoRows {
      fmt.Printf("data not found, %v\n", err)
      fmt.Printf("%+v\n", err)
      return
   }
   if err != nil {
      // unknown error
   }
}
/*Output:
data not found, Call failed: GetSql failed: sql: no rows in result set
sql: no rows in result set
main.GetSql
    /usr/three/main.go:11
main.Call
    /usr/three/main.go:15
main.main
    /usr/three/main.go:19
runtime.main
    ...
*/

```

从输出内容可以看到， 使用 %v 作为格式化参数，那么错误信息会保持一行， 其中依次包含调用栈的上下文文本。 使用 %+v ，则会输出完整的调用栈详情。
如果不需要增加额外上下文信息，仅附加调用栈后返回，可以使用 WithStack 方法：

```go
func GetSql() error {
   return errors.WithStack(sql.ErrNoRows)
}
```

> 注意：无论是 Wrap ， WithMessage 还是 WithStack ，当传入的 err 参数为 nil 时， 都会返回nil， 这意味着我们在调用此方法之前无需作 nil 判断，保持了代码简洁

### 方案 2：golang.org/x/xerrors

结合社区反馈，Go 团队完成了在 Go 2 中简化错误处理的提案。 Go核心团队成员 Russ Cox 在xerrors中部分实现了提案中的内容。它用与 github.com/pkg/errors相似的思路解决同一问题， 引入了一个新的 fmt 格式化动词: %w，使用 Is 进行判断。

```go

import (
   "database/sql"
   "fmt"

   "golang.org/x/xerrors"
)

func Call() error {
   if err := GetSql(); err != nil {
      return xerrors.Errorf("bar failed: %w", GetSql())
   }
   return nil
}

func GetSql() error {
   return xerrors.Errorf("GetSql failed: %w", sql.ErrNoRows)
}

func main() {
   err := Call()
   if xerrors.Is(err, sql.ErrNoRows) {
      fmt.Printf("data not found, %v\n", err)
      fmt.Printf("%+v\n", err)
      return
   }
   if err != nil {
      // unknown error
   }
}
/* Outputs:
data not found, Call failed: GetSql failed: sql: no rows in result set
bar failed:
    main.Call
        /usr/four/main.go:12
  - GetSql failed:
    main.GetSql
        /usr/four/main.go:18
  - sql: no rows in result set
*/
```

与 github.com/pkg/errors 相比，它有几点不足：
1. 使用 : %w 代替了 Wrap ， 看似简化， 但失去了编译期检查。 如果没有冒号，或 : %w 不位于于格式化字符串的结尾，或冒号与百分号之间没有空格，包装将失效且不报错；
2. 而且，调用 xerrors.Errorf 之前需要对参数进行nil判断。 这完全没有简化开发者的工作

### 方案 3：Go 1.13 内置支持

 Go 1.13 将 xerrors 的部分功能（不是全部）整合进了标准库。 它继承了上面提到的 xerrors 的全部缺点， 并额外贡献了一项。因此目前没有使用它的必要。

```go

import (
   "database/sql"
   "errors"
   "fmt"
)

func Call() error {
   if err := GetSql(); err != nil {
      return fmt.Errorf("Call failed: %w", GetSql())
   }
   return nil
}

func GetSql() error {
   return fmt.Errorf("GetSql failed: %w", sql.ErrNoRows)
}

func main() {
   err := Call()
   if errors.Is(err, sql.ErrNoRows) {
      fmt.Printf("data not found,  %+v\n", err)
      return
   }
   if err != nil {
      // unknown error
   }
}
/* Outputs:
data not found,  Call failed: GetSql failed: sql: no rows in result set
*/
```

上面的代码与 xerrors 版本非常接近。但是它不支持调用栈信息输出， 根据官方的说法， 此功能没有明确的支持时间。因此其实用性远低于 github.com/pkg/errors。

## Golang 中将来可能的错误处理方式
> 在Go 2的草案中，我们看到了有关于error相关的一些提案，那就是check/handle函数。

我们也许在下一个大版本的Golang可以像下面这样处理错误：

```go
import "fmt"
func game() error {
    handle err {
        return fmt.Errorf("dependencies error: %v", err)
    }

    resource := check findResource() // return resource, error
    defer func() {
        resource.Release()
    }()

    profile := check loadProfile() // return profile, error
    defer func() {
        profile.Close()
    }

    // ...
}

```
感兴趣的同学可以关注下这个提案：https://go.googlesource.com/proposal/+/master/design/go2draft-error-handling-overview.md


## 得出结论
> 重要的是要记住，包装错误会使该错误成为您API的一部分。如果您不想将来将错误作为API的一部分来支持，则不应包装该错误。
无论是否包装错误，错误文本都将相同。那些试图理解错误的人将得到相同的信息，无论采用哪种方式; 是否要包装错误的选择取决于是否要给程序提供更多信息，以便他们可以做出更明智的决策，还是保留该信息以保留抽象层。

通过以上对比， 相信你已经有了选择。 再明确一下我的看法，如果你正在使用 `github.com/pkg/errors` ，那就保持现状吧。目前还没有比它更好的选择。如果你已经大量使用 golang.org/x/xerrors ， 别盲目换成 go 1.13 的内置方案。


总的来说，Go 在诞生之初就在各个方面表现得相当成熟、稳健。 在演进路线上很少出现犹疑和摇摆， 而在错误处理方面却是个例外。 除了被广泛吐槽的 if err != nil 之外， 就连其改进路线也备受争议、分歧明显，以致于一个改进提案都会因为压倒性的反对意见而不得不作出调整。 好在 Go 团队比以前更加乐于倾听社区意见，团队甚至专门就此问题建了个反馈收集页面。相信最终大家会找到更好的解决方案。


<hr />
