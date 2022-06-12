---
title: Go关于time包的解析与使用
tags: [golang]
copyright: true
date: 2018-12-16 23:16:34
permalink:
categories: golang
description: golang time包的解析与使用
image: https://static001.geekbang.org/resource/image/cb/b5/cbca9c8c9a3b46ea297863f4ac9d20b5.jpg
---
<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### Go关于时间与日期的处理

#### 关于time的数据类型

- time包依赖的数据类型有：**time.Time**,**time.Month**,**time.WeekDay**,**time.Duration**,**time.Location**.

- 详细介绍以上几种数据类型

  - time.Time

  - `/usr/local/go/src/time/time.go` 定义如下:

    ```go
    type Time struct {
        sec int64 // 从1年1月1日 00:00:00 UTC 至今过去的秒数
        nsec int32 // 最近一秒到下一秒过去的纳秒数
        loc *Location // 时区
    }
    ```

    time.Time会返回纳秒时间精度的时间

    ```go
    var ti time.Time
    ti = time.Now()
    fmt.Printf("时间: %v, 时区:  %v,  时间类型: %T\n", t, t.Location(), t)
    //时间: 2018-12-15 09:06:05.816187261 +0800 CST, 时区:  Local,  时间类型: time.Time
    ```

  - time.Month, go中自己重新定义了month的类型，与time.year和time.day不同。

    ```
    type Month int
    
    const (
        January Month = 1 + iota
        February
        March
        April
        May
        June
        July
        August
        September
        October
        November
        December
    )
    ```

    iota是golang语言的常量计数器,只能在常量的表达式中使用。
     iota在const关键字出现时将被重置为0(const内部的第一行之前)，const中每新增一行常量声明将使iota计数一次(iota可理解为const语句块中的行索引)。 使用iota能简化定义，在定义枚举时很有用。

  - time.WeekDay,代表一周之中的星期几（当然是按照西方的规则，他们把周日当做是一周的开始）

    ```go
    type WeekDay int
    
    const (
        Sunday Weekday = iota
        Monday
        Tuesday
        Wednesday
        Thursday
        Friday
        Saturday
    )
    ```

  - time.Duration,代表两个时间点之间的纳秒差值。

    ```go
    type Duration int64
    
    const (
        Nanosecond  Duration = 1
        Microsecond          = 1000 * Nanosecond
        Millisecond          = 1000 * Microsecond
        Second               = 1000 * Millisecond
        Minute               = 60 * Second
        Hour                 = 60 * Minute
    )
    ```

  - time.Location,时区信息

    ```go
    type Location struct {
        name string
        zone []zone
        tx   []zoneTrans
        cacheStart int64
        cacheEnd   int64
        cacheZone  *zone
    }
    //北京时间：Asia/Shanghai
    ```

  #### 以上类型receiver的实现方法

  - time.Time相关方法

  `func Now() Time {}` // 当前本地时间

   `func Unix(sec int64, nsec int64) Time {}`  // 根据时间戳返回本地时间

   `func Date(year int, month Month, day, hour, min, sec, nsec int, loc *Location) Time {}` // 返回指定时间

  ```go
  / 当前本地时间
  t = time.Now()
  fmt.Println("'time.Now': ", t)
  
  // 根据时间戳返回本地时间
  t_by_unix := time.Unix(1487780010, 0)
  fmt.Println("'time.Unix': ", t_by_unix)
  
  // 返回指定时间
  t_by_date := time.Date(2017, time.Month(2), 23, 1, 30, 30, 0, l)
  fmt.Println("'time.Date': ", t_by_date)
  
  ```

  - 按照时区信息显示时间

  - `func (t Time) UTC() Time {}` // 获取指定时间在UTC 时区的时间表示
  -  `func (t Time) Local() Time {}` // 以本地时区表示
  -  `func (t Time) In(loc *Location) Time {}` // 时间在指定时区的表示
  -  `func (t Time) Format(layout string) string {}` // 按指定格式显示时间


```go
// 获取指定时间在UTC 时区的时间表示
t_by_utc := t.UTC()
fmt.Println("'t.UTC': ", t_by_utc)

// 获取本地时间表示
t_by_local := t.Local()
fmt.Println("'t.Local': ", t_by_local)

// 时间在指定时区的表示
t_in := t.In(time.UTC)
fmt.Println("'t.In': ", t_in)

// Format
fmt.Println("t.Format", t.Format(time.RFC3339))
```

- 获取年月日等信息

`func (t Time) Date() (year int, month Month, day int) {}` // 返回时间的日期信息

 `func (t Time) Year() int {}` // 返回年

 `func (t Time) Month() Month {}` // 月

 `func (t Time) Day() int {}` // 日

 `func (t Time) Weekday() Weekday {}` // 星期

 `func (t Time) ISOWeek() (year, week int) {}` // 返回年，星期范围编号

 `func (t Time) Clock() (hour, min, sec int) {}` // 返回时间的时分秒

 `func (t Time) Hour() int {}` // 返回小时

 `func (t Time) Minute() int {}` // 分钟

 `func (t Time) Second() int {}` // 秒

 `func (t Time) Nanosecond() int {}` // 纳秒

 `func (t Time) YearDay() int {}` // 一年中对应的天

 `func (t Time) Location() *Location {}` // 时间的时区

 `func (t Time) Zone() (name string, offset int) {}` // 时间所在时区的规范名和想对UTC 时间偏移量

 `func (t Time) Unix() int64 {}` // 时间转为时间戳

 `func (t Time) UnixNano() int64 {}` // 时间转为时间戳（纳秒）

```go
// 返回时间的日期信息
year, month, day := t.Date()
fmt.Println("'t.Date': ", year, month, day)

// 星期
week := t.Weekday()
fmt.Println("'t.Weekday': ", week)

// 返回年，星期范围编号
year, week_int := t.ISOWeek()
fmt.Println("'t.ISOWeek': ", year, week_int)

// 返回时间的时分秒
hour, min, sec := t.Clock()
fmt.Println("'t.Clock': ", hour, min, sec)

```

- 时间运算

`func (t Time) IsZero() bool {}` // 是否是零时时间

 `func (t Time) After(u Time) bool {}` // 时间在u 之前

 `func (t Time) Before(u Time) bool {}` // 时间在u 之后

 `func (t Time) Equal(u Time) bool {}` // 时间与u 相同

 `func (t Time) Add(d Duration) Time {}` // 返回t +d 的时间点

 `func (t Time) Sub(u Time) Duration {}` // 返回 t-u

 `func (t Time) AddDate(years int, months int, days int) Time {}` 返回增加了给出的年份、月份和天数的时间点Time

```
// 返回增加了给出的年份、月份和天数的时间点Time
t_new := t.AddDate(0, 1, 1)
fmt.Println("'t.AddDate': ", t_new)

// 时间在u 之前
is_after := t.After(t_new)
fmt.Println("'t.After': ", is_after)

```

- time.Duration的类型receiver实现的方法

`func (d Duration) String() string` // 格式化输出 Duration

 `func (d Duration) Nanoseconds() int64` // 将时间段表示为纳秒

 `func (d Duration) Seconds() float64` // 将时间段表示为秒

 `func (d Duration) Minutes() float64` // 将时间段表示为分钟

 `func (d Duration) Hours() float64` // 将时间段表示为小时

```go
// time.Duration 时间段
fmt.Println("time.Duration 时间段")
d = time.Duration(10000000000000)//输入参数为int64类型

fmt.Printf("'String: %v', 'Nanoseconds: %v', 'Seconds: %v', 'Minutes: %v', 'Hours: %v'\n", 
d.String(), d.Nanoseconds(), d.Seconds(), d.Minutes(), d.Hours())
// 'String: 2h46m40s', 'Nanoseconds: 10000000000000', 'Seconds: 10000', 'Minutes: 166.66666666666666', 'Hours: 2.7777777777777777'

```

- time.Location的receiver实现的方法

`func (l *Location) String() string` // 输出时区名

 `func FixedZone(name string, offset int) *Location` // FixedZone 使用给定的地点名name和时间偏移量offset（单位秒）创建并返回一个Location

 `func LoadLocation(name string) (*Location, error)` // LoadLocation 使用给定的名字创建Location

`func Sleep(d Duration)` // Sleep阻塞当前go程至少d代表的时间段。d<=0时，Sleep会立刻返回

```go
d_second := time.Second
time.Sleep(d_second)
```



### 一些常用的技巧与代码示例

- string与time.Time互转

  ```go
  const (
  	date        = "2006-01-02"
  	datetime    = "2006-01-02 15:04:02"
  )
  
  timeStamp := time.Now().Format(date) //将当前时间，即time.Time类型转为string
  
  billTimeStamp, err := time.Parse(date,timeStamp)//将String类型的时间转为time.Time类型
  ```

- unix time与String互转

```go
endTime := time.Unix(time.Now().Unix(), 0)//此时endTime是time.Time类型
endStr := endTime.Format(datetime)//将unix time 转为了String，再根据上面的例子，可转为time.Time

package main


import (

"fmt"

"time"
)

func main() {

//获取时间戳

timestamp := time.Now().Unix()

fmt.Println(timestamp)

//格式化为字符串,tm为Time类型

tm := time.Unix(timestamp, 0)

fmt.Println(tm.Format("2006-01-02 03:04:05 PM"))

fmt.Println(tm.Format("02/01/2006 15:04:05 PM"))

 

//从字符串转为时间戳，第一个参数是格式，第二个是要转换的时间字符串

tm2, _ := time.Parse("01/02/2006", "02/08/2018")

fmt.Println(tm2.Unix())

}
```

#### 相关参考

[pkg/time中文翻译](https://link.jianshu.com/?t=http://studygolang.com/static/pkgdoc/pkg/time.htm)

[pkg/time英文](https://link.jianshu.com/?t=https://golang.org/pkg/time/)

