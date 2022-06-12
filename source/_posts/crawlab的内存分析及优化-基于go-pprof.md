---

title: crawlab的golang后端内存分析及优化-基于go pprof
tags: [golang]
copyright: true
date: 2019-08-20 19:09:59
permalink:
categories: golang
description: crawlab的golang后端内存分析及优化-基于go pprof
image:
---
<p class="description"></p>
<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 背景

Crawlab发布几个月以来，其中经历过多次迭代，在使用者们的积极反馈下，crawlab爬虫平台逐渐稳定，但是最近有用户报出crawlab启动一段时间后，主节点机器会出现内存占用过高的问题，一台4G内存的机器在运行crawlab后竟然能占用3.5G以上，几乎可以肯定后端服务的某个接口由于代码不规范导致内存占用，于是决定对crawlab进行一次内存分析。

### 2. 分析

分析内存光靠手撕代码是比较困难的，总要借助一些工具。Golang pprof是Go官方的profiling工具，非常强大，使用起来也很方便。

首先，我们在crawlab项目中嵌入如下几行代码：

```go
_ "net/http/pprof"
go func() {
		http.ListenAndServe("0.0.0.0:8888", nil)
	}()
```

将crawlab后端服务启动后，浏览器中输入`http://ip:8899/debug/pprof/`就可以看到一个汇总分析页面，显示如下信息：

```go
/debug/pprof/

profiles:
0    block
32    goroutine
552    heap
0    mutex
51    threadcreate

full goroutine stack dump
```

点击heap，在汇总分析页面的最上方可以看到如下图所示，红色箭头所指的就是当前已经使用的堆内存是25M,amazing！在我只上传一个爬虫文件的情况下，一个后端服务所用的内存竟然能达到25M

![mYnJuF.jpg](https://s2.ax1x.com/2019/08/20/mYnJuF.jpg)



接下来我们需要借助`go tool pprof`来分析：

```go
go tool pprof -inuse_space http://本机Ip:8888/debug/pprof/heap
```

这个命令进入后，是一个类似`gdb`的交互式界面，输入`top`命令可以前10大的内存分配，`flat`是堆栈中当前层的inuse内存值，cum是堆栈中本层级的累计inuse内存值（包括调用的函数的inuse内存值，上面的层级）

[![mYMF91.png](https://s2.ax1x.com/2019/08/20/mYMF91.png)](https://imgchr.com/i/mYMF91)

可以看到，bytes.makeSlice这个内置方法竟然使用了24M内存，继续往下看，可以看到ReadFrom这个方法，搜了一下，发现 `ioutil.ReadAll()` 里会调用 `bytes.Buffer.ReadFrom`, 而 `bytes.Buffer.ReadFrom` 会进行 `makeSlice`。再回头看一下io/ioutil.readAll的代码实现，

```go
func readAll(r io.Reader, capacity int64) (b []byte, err error) {
    buf := bytes.NewBuffer(make([]byte, 0, capacity))
    defer func() {
        e := recover()
        if e == nil {
            return
        }
        if panicErr, ok := e.(error); ok && panicErr == bytes.ErrTooLarge {
            err = panicErr
        } else {
            panic(e)
        }
    }()
    _, err = buf.ReadFrom(r)
    return buf.Bytes(), err
}

// bytes.MinRead = 512
func ReadAll(r io.Reader) ([]byte, error) {
    return readAll(r, bytes.MinRead)
}

```

可以看到，`ioutil.ReadAll` 每次都会分配初始化一个大小为 `bytes.MinRead` 的 buffer ，`bytes.MinRead` 在 Golang 里是一个常量，值为 `512` 。就是说每次调用 `ioutil.ReadAll` 都会分配一块大小为 512 字节的内存，看起来是正常的，但我们再看一下`ReadFrom`的实现，

```go
// ReadFrom reads data from r until EOF and appends it to the buffer, growing
// the buffer as needed. The return value n is the number of bytes read. Any
// error except io.EOF encountered during the read is also returned. If the
// buffer becomes too large, ReadFrom will panic with ErrTooLarge.
func (b *Buffer) ReadFrom(r io.Reader) (n int64, err error) {
    b.lastRead = opInvalid
    // If buffer is empty, reset to recover space.
    if b.off >= len(b.buf) {
        b.Truncate(0)
    }
    for {
        if free := cap(b.buf) - len(b.buf); free < MinRead {
            // not enough space at end
            newBuf := b.buf
            if b.off+free < MinRead {
                // not enough space using beginning of buffer;
                // double buffer capacity
                newBuf = makeSlice(2*cap(b.buf) + MinRead)
            }
            copy(newBuf, b.buf[b.off:])
            b.buf = newBuf[:len(b.buf)-b.off]
            b.off = 0
        }
        m, e := r.Read(b.buf[len(b.buf):cap(b.buf)])
        b.buf = b.buf[0 : len(b.buf)+m]
        n += int64(m)
        if e == io.EOF {
            break
        }
        if e != nil {
            return n, e
        }
    }
    return n, nil // err is EOF, so return nil explicitly
}
```

这个函数主要作用就是从 `io.Reader` 里读取的数据放入 buffer 中，如果 buffer 空间不够，就按照每次 `2x + MinRead` 的算法递增，这里 `MinRead` 的大小也是 512 Bytes ，也就是说如果我们一次性读取的文件过大，就会导致所使用的内存倍增，假设我们的爬虫文件总共有500M,那么所用的内存就有500M * 2 + 512B，况且爬虫文件中还带了那么多log文件，那看看crawlab源码中是哪一段使用`ioutil.ReadAll`读了爬虫文件，定位到了这里：

[![mYQTWn.jpg](https://s2.ax1x.com/2019/08/20/mYQTWn.jpg)](https://imgchr.com/i/mYQTWn)

这里直接将全部的文件内容，以二进制的形式读了进来，导致内存倍增，令人窒息的操作。

其实在读大文件的时候，把文件内容全部读到内存，直接就翻车了，正确是处理方法有两种，一种是流式处理：

```go
func ReadFile(filePath string, handle func(string)) error {
    f, err := os.Open(filePath)
    defer f.Close()
    if err != nil {
        return err
    }
    buf := bufio.NewReader(f)

    for {
        line, err := buf.ReadLine("\n")
        line = strings.TrimSpace(line)
        handle(line)
        if err != nil {
            if err == io.EOF{
                return nil
            }
            return err
        }
        return nil
    }
}
```



第二种方案就是分片处理，当读取的是二进制文件，没有换行符的时候，使用这种方案比较合适：

```go
func ReadBigFile(fileName string, handle func([]byte)) error {
    f, err := os.Open(fileName)
    if err != nil {
        fmt.Println("can't opened this file")
        return err
    }
    defer f.Close()
    s := make([]byte, 4096)
    for {
        switch nr, err := f.Read(s[:]); true {
        case nr < 0:
            fmt.Fprintf(os.Stderr, "cat: error reading: %s\n", err.Error())
            os.Exit(1)
        case nr == 0: // EOF
            return nil
        case nr > 0:
            handle(s[0:nr])
        }
    }
    return nil
}
```



我们这类采用的第二种方式来优化，优化后再来看下内存分析：

[![mYltYj.png](https://s2.ax1x.com/2019/08/20/mYltYj.png)](https://imgchr.com/i/mYltYj)

占用1M内存，这才是一个正常后端服务该有的内存大小，源码已push到crawlab，可以在GitHub项目源码中阅读。

最后附上项目链接，https://github.com/tikazyq/crawlab，为crawlab打电话，欢迎大家一起贡献，让crawlab变得更好用！

<hr />
