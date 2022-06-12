---
title: Go team 关于如何保持 Go Modules 兼容性的一些实践
tags: [golang]
copyright: true
date: 2020-08-13 14:05:13
permalink:
categories: golang
description: Go team 关于如何保持 Go Modules 兼容性的一些实践
image: https://static001.geekbang.org/resource/image/0f/76/0f0963bc50f97aa1171fa81ae6e96776.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

近日，Go team 在其官方 blog 上讨论了如何让你的 Go Modules 保持兼容性的话题，并给出了一些建议，这些建议都是该团队在实际开发中不断踩坑总结出来的精华，可以说是最佳实践。我们站在巨人的肩膀上，可以写出更优雅，更具有兼容性的代码，下面让我们深入逐条解读这些建议。

------

随着新功能的添加，或者重构 Go Module 的某些公共部分，Go Module 将随着时间的推移而不断发生变化。

但是，发布新的 Go Module 版本对使用者来是一个噩耗。 他们必须找到新版本，学习新的API，并更改其代码。 而且某些用户可能永远不会更新，这意味着您必须永远为代码维护两个版本。 因此，通常最好以兼容的方式更改现有的 Go Module。

在本文中，我们将探讨一些代码技巧，能够让你保持 Go Module 的兼容性。 核心的思想就是是：添加，但是不要更改或删除你的 Go Module 代码。 我们还将从宏观角度讨论如何设计具备高度兼容性的 API 。

### 新增函数

通常来说，改变函数的形参是破坏代码兼容性最常见的情况。我们讲过讨论几个解决这种问题的方式，但让我们首先看一个不好的实践。

有这么一个函数：

```go
func Run(name string)
```

当我们出于某个情况要扩展这个函数，为这个函数添加一个形参 `size` ：

```go
func Run(name string, size ...int)
```

假如你在其他代码中，或者 Go Module 的使用者更新了，那么像下面的代码就会出现问题：

```go
package mypkg
var runner func(string) = yourpkg.Run
```

原来的 `Run` 函数的类型是 `func(string)`，但是新的 `Run` 函数的类型变成了 `func(string, ...int)`，所以在编译阶段就会报错。必须要根据新的函数类型修改调用方式，这给使用 Go Module 的开发者造成很多不便，甚至出现 bug。

针对这种情况，我们可以新增一个函数来解决这个问题，而不是修改函数签名。我们都知道，`context` 包是 Golang 1.17 版本之后才引入的，通常 `ctx` 会做为函数的第一个参数传入。但是现有的已经很稳定的 API 的可导出函数不可能去修改函数签名，在其函数第一个入参添加 `context.Context`，这样会影响所有函数调用方，尤其在一些底层代码库中，这是非常危险的操作。

Go team 使用`新增函数` 的方法解决了这个问题。举个栗子，`database/sql` 这个 package 的 `Query` 方法的签名一直是:

```go
func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
```

当 `context` package 引入的时候，Go team 新增了这样一个函数：

```go
func (db *DB) QueryContext(ctx context.Context, query string, args ...interface{}) (*Rows, error)
```

并且只修改了一处代码：

```go
func (db *DB) Query(query string, args ...interface{}) (*Rows, error) {
    return db.QueryContext(context.Background(), query, args...)
}
```

通过这种方式，Go team 能够在平滑地升级一个 `package` 的同时不对代码的可读性、兼容性造成影响。类似的代码在 golang 源码中随处可见。

### 可选参数（optional arguments）

如果你在实现 package 之前就确定这个函数后面可能会需要添加参数来扩展某些功能，那么你可以提前在函数签名是使用可选参数（optional arguments）。最简单的方法是在函数签名中使用结构体参数，下面是 golang 源码中 `crypto/tls.Dial` 的一段代码：

```go
func Dial(network, addr string, config *Config) (*Conn, error)
```

`Dial` 函数实现 `TLS` 的握手操作，这个过程中需要其他很多参数，同时还支持默认值。当给 `config` 传递 `nil` 的时候就是使用默认值；当传递 `Config struct` 的时候将会覆盖默认值。假如以后出现了新的 `TLS` 配置参数，可以很轻松地通过在 `Config struct` 中添加新字段来实现，这种方式是向后兼容的。

有些情况下，新增函数和使用可选参数的方式可以结合起来，通过把可选参数的结构体变成一个方法的接收者(receiver)。比如，在 Go 1.11 之前，`net` package 中的`Listen` 方法的签名是：

```go
func Listen(network, address string) (Listener, error)
```

但是在 Go 1.11 中，Go team 新增加了两个 `feature` :

1. 传递了 context 参数；
2. 增加了 `control function`，允许调用者在网络连接还没有 `bind` 的时候调整原始连接的参数。

这看起来是相当大的调整了，如果是一般开发者，最多也就会新增一个函数，参数中添加 `context`, `control function`。但是 Go team 的开发者非等闲之辈，`net` package 的作者想到未来的某一天这个函数是不是会有调整，或者需要更多的参数？于是就预留了一个 `ListenConfig` 的结构体，为这个 strcut 实现了 `Listen` 方法，从而也不用再新增一个函数才能解决问题。

```go
type ListenConfig struct {
    Control func(network, address string, c syscall.RawConn) error
}

func (*ListenConfig) Listen(ctx context.Context, network, address string) (Listener, error)
```

还有一种叫做可选类型的设计模式，是把可选的函数作为函数形参，每一个可选函数都可以通过参数来调整其状态。在 Rob Pike 的 blog (https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html) 里对这种模式进行了详细的解读。这种设计模式在 grpc 的源码中大量使用。

`option types` 与函数参数中的 `option struct`具有相同的作用：它们是传递行为，修改配置的可扩展方式。 决定选择哪个很大程度上取决于具体场景。 来看一下 gRPC 的 `DialOption` 选项类型的用法：

```go
grpc.Dial("some-target",
  grpc.WithAuthority("some-authority"),
  grpc.WithMaxDelay(time.Second),
  grpc.WithBlock())
```

当然你也可以作为 struct 选项实现：

```go
notgrpc.Dial("some-target", &notgrpc.Options{
  Authority: "some-authority",
  MaxDelay:  time.Minute,
  Block:     true,
})
```

以上，任一种方式都是能够维持 Go Module 兼容性的方法，可以根据不同的场景选择合理的实现。

### 保证 interfaces 的兼容性

有时候，新特性的支持需要更改对外暴露的（public）的接口:  需要使用新的方法来扩展接口。直接向接口添加方法是不合适的，这会导致接口的实现方都需要修改代码。那么，我们如何才能在公开的接口上支持新方法呢？

Go team 给出的建议是：使用新方法定义一个新接口，然后在使用旧接口的任何地方动态检查提供的类型是旧类型还是新类型。

让我们以 golang  源码中 `archive/tar` package 来详细说明一下。 `tar.NewReader` 以 `io.Reader` 作为参数，但是后来 Go team 觉得应该提供一种更加高效的方式，就是当调用 `Seek` 方法的时候可以跳过一个文件的 header。但是又不能直接在 `io.Reader` 中新增 `Seek` 方法，这会影响所有实现了 `io.Reader` 的方法（如果你有看过 golang 源码，就会知道 io.Reader 接口的应用有多广泛了）。

另外一种方法是将 `tar.NeaReader` 的入参改成 `io.ReaderSeeker` interface，因为该 interface 同时支持 `io.Reader` 和 `Seek` 。但是正如前面所讲，改变一个函数的签名，不是一种好的方式。

所以 Go team 决定维持 `tar.NewReader` 的签名不变，在 `Read` 方法中进行类型检查：

```go
package tar

type Reader struct {
  r io.Reader
}

func NewReader(r io.Reader) *Reader {
  return &Reader{r: r}
}

func (r *Reader) Read(b []byte) (int, error) {
  if rs, ok := r.r.(io.Seeker); ok {
    // Use more efficient rs.Seek.
  }
  // Use less efficient r.r.Read.
}
```

如果遇到要向现有接口添加方法的情况，则可以遵循此策略。 首先使用新方法创建新接口，或者使用新方法标识现有接口。 接下来，确定需要添加的相关代码啊，对第二个接口进行类型检查，并添加使用它的代码。

在可能的情况下，最好避免这种问题。 例如，在设计构造函数时，最好返回具体类型。 与接口不同，使用具体类型可以让你将来在不中断用户使用的情况下添加新方法，同时将来可以更轻松地扩展你的 Go Module。

Tip: 如果你用到了一个 interface，但是你不想用户去实现它，你可以为 interface 添加 `unexported` 的方法。

```go
type TB interface {
    Error(args ...interface{})
    Errorf(format string, args ...interface{})
    // ...

    // A private method to prevent users implementing the
    // interface and so future additions to it will not
    // violate Go 1 compatibility.
// private 避免用户去实现它
    private()
}
```

### 新增配置方法

到目前为止，我们讨论了修改函数签名或者为 interface 添加方法，会影响到用户的代码导致编译失败。实际上，函数行为的改变会造成同样的问题。例如，很多开发者希望 `json.Decoder` 可以忽略 `struct` 中没有的 `json` 字段。但是当 Go team 想要在这种情况下返回一些错误的时候，就必须要小心，因为这样做会导致很多使用该方法的用户突然收到以前从未遇到的错误。

因此，他们没有更改所有用户的行为，而是向Decoder结构添加了一种配置方法：`Decoder.DisallowUnknownFields` 。 调用此方法会使用户选择新行为，同时会为现有用户保留旧的方法。

### 保持 struct 的兼容性

通过上面的内容我们了解到，对函数签名的任何更改都是一种破坏性的改动。 但是如果使用 `struct` 就会让你的代码灵活很多， 如果具有可导出的结构体类型，则几乎可以随时添加一个字段或删除一个未导出的字段而不会破坏兼容性。 添加字段时，请确保其零值有意义并保留旧的行为，以便未设置该字段的现有代码继续起作用。

还记得上面讲到的 `net` package 的作者在 Go 1.11 的时候添加的 `ListenConfig` struct 吗？事实证明他的设计是对的。在 Go 1.13 中，新增了 `KeepAlive` 字段，允许取消或使用 keep-alive 的功能。有了之前的设计，这个字段的加入就容易多了。

关于 struct 的使用，有一个细节如果你没有注意到的话，也会对用户造成很大的影响。如果 struct 中所有的字段都是可判等的（意思是可用通过 `==` or `!=`来比较，或者可以作为 map 的 key），那么这个 struct 就是可判等的。这种情况下，如果你为 struct 添加了一个不可判等的类型，将会导致这个 struct 也变成不可判等的。如果用户在代码中使用了你的 struct 进行判等操作，那么就会遇到代码错误。

如果你要保持结构体可判等，就不要向其添加不可比较的字段。可以为此编写测试用例来避免遗忘。

### Conclusion

从头开始规划 API 的时候，请仔细考虑 API 在将来的可扩展性。 而且当你确实需要添加新功能时，请记住以下规则：添加，不要更改或删除。请牢记，添加接口方法，函数参数和返回值都会导致 Go Module 不能向后兼容。

如果你确实需要大规模更改 API，或者要添加更多新特性，那么使用新的 API 版本会是更好的方式。 但是大多数时候，进行向后兼容的更改应该是你的首选，能够避免给用户带来麻烦。

<hr />
