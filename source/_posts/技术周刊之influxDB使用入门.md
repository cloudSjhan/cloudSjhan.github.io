---

title: 技术周刊之influxDB使用入门
tags: [influxDB,技术周刊]
copyright: true
date: 2018-12-08 23:25:10
permalink:
categories: 技术周刊
description: 介绍influxDB的基本概念，基本操作以及如何使用go实现influxDB的操作
image: https://static001.geekbang.org/resource/image/ed/02/edb8e8c857c26b85273eb83d58b68d02.jpg
---

<p class="description"></p>

<img src="https://" alt="" style="width:100%" />

<!-- more -->

### 前言

InfluxDB是一个用于存储和分析时间序列数据的开源数据库。

主要特性有：

- 内置HTTP接口，使用方便
- 数据可以打标记，查让查询可以很灵活
- 类SQL的查询语句
- 安装管理很简单，并且读写数据很高效
- 能够实时查询，数据在写入时被索引后就能够被立即查出
- ……

在最新的[DB-ENGINES](https://db-engines.com/en/ranking/time+series+dbms)给出的时间序列数据库的排名中，InfluxDB高居第一位，可以预见，InfluxDB会越来越得到广泛的使用。

- influxDB使用go语言编写，采用了SQL like的语法，非常灵活高效，如果你的数据是与时间相关的，那么使用influxDB做数据可视化是最合适不过的，尤其是influxDB自身就提供数据库CRUD所需要的API，虽然不是RESTFul的，但是也省去了编写后端接口的力气。

- 下面从influxDB的安装、使用CLI的influxdb基本操作、使用API对influxdb操作及golang代码实现、经历的坑，几个方面分享influxDB的入门经历。

### 安装

- 本次安装的环境是：

  - CentOS 7
  - 内核版本：4.4.135-1.el7.elrepo.x86_64

- 直接使用yum安装

  - 

  ```shell 
  cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo # 输入influxDB的repoURL地址等信息
  [influxdb]
    name = InfluxDB Repository - RHEL \$releasever
    baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
    enabled = 1
    gpgcheck = 1
    gpgkey = https://repos.influxdata.com/influxdb.key
    EOF
    #EOF是文本的结束符
  ```

- ```shell
  sudo yum install influxdb
  ```

- influxd config

  安装完成后使用该命令查看influxDB的配置内容，default的config文件路径在：/etc/influxdb/influxdb.conf

- 启动influxDB

```shell
[root@k8s-m1 ~]# influx
Connected to http://localhost:8086 version 1.7.1
InfluxDB shell version: 1.7.1
Enter an InfluxQL query
>
```

到此完成了influxDB的安装，接下来我们做基本的配置。

### 配置

- 用户管理

```shell
-- 创建一个管理员用户
CREATE USER "admin" WITH PASSWORD 'xxxx' WITH ALL PRIVILEGES
-- 创建一个普通用户
CREATE USER "user" WITH PASSWORD 'xxxxx'
-- 为用户授权读权限
GRANT READ ON [database] to "user"
-- 为用户授权写权限
GRANT WRITE ON [database] to "user"
--------------------- 
# 需要修改InfluxDB的配置文件/etc/influxdb/influxdb.conf，设置http下的auth-enabled = true，重启后，使用influx命令登录数据库就需要用户名和密码了。（Influx命令实际上也是使用API来操作InfluxDB的，InfluxDB只提供了API接口）
```

- 查看用户

```shell
> show users
user  admin
----  -----
admin true
sjhan true
>
```

influxDB的配置项目有很多，剩下的可以根据自己的需求继续研究，这里就不展开了。

在进行数据库基本操作之前我们必须了解一下infuxDB的一些基本概念

#### influxDB基本概念

- influxDB里面最基本的概念就是，measurement，tags，fields，points。我们可以类比于MySQL来理解这几个字段：
- measurement类似于SQL中的table；
- tags类似SQL中的被索引的列；
- fields类似于SQL中没有被索引的列；
- points对应SQL的table中的每行数据。
- 知道了这几个概念，便可以继续往下进行，如需更加详细的文档，英文版文档[猛戳这里](https://docs.influxdata.com/influxdb/v1.7/)，当然也有中文版，[猛戳这里](https://jasper-zhang1.gitbooks.io/influxdb/)。不知为何中文版我只有番蔷才能访问。

### influxDB基本操作

- 首先，跟MySQL一样，我们需要创建一个数据库

```
-- 创建数据库，默认设置
CREATE DATABASE "first_db"
-- 创建数据库，同时创建一个Retention Policy，数据保留时间描述
-- Retention Policy各部分描述：DURATION为数据存储时长，下面的1d即只存1天的数据；REPLICATION为数据副本，一般在使用集群的时候才会设置为>1；SHARD DURATION为分区间隔，InfluxDB默认对数据分区，填写30m即对数据每隔30分钟做一个新的分区；Name是RP的名字。
CREATE DATABASE "first_db" WITH DURATION 1d REPLICATION 1 SHARD DURATION 30m NAME "myrp"
```

我们创建了一个influxDB的数据库，名字为first_db, 数据存储时间为一天，一个副本，每30分钟做一个新的分区。

- influxDB插入数据

  influx -username admin -password

  我们插入一条数据到刚刚创建的数据库中

  ```sql
  insert product,productName=disk,usageType=pay,creator=zhangsan,appId=105 cost=3421.6
  ```

  我们分析一下这条插入语句，其中product字段是influxDB中的measurement,前面讲基本概念的时候已经解释过，类似于MySQL中的table，“productName=disk,usageType=pay,creator=zhangsan,appId=105”，这一坨在influxDB中叫做tag set,可以理解为tag的一个集合，tag的类型只能是字符串的K-V，还有需要注意的是tag set与前面的measurement之间只有一个逗号，**并没有空格！**，一开始不知道这回事，怎么插入都是失败。“cost=3421.6”这个叫做filed set，filed的类型可以是float、boolean、integer。这样插入的一条数据，influxDB中叫做一个point。



  - 查询操作

  查询之前要选择你想查询的数据库

  ```shell
  use first_db
  ```



  ```shell
  select * from product
  ```

- ![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxzdysawf8j31hq0cqwgt.jpg)

- 可以看到influxDB自动为我们的这个point加了一个timestamp，这个是数据的UNIX时间格式的时间精度，我们在启动数据库时可以定义这个precision，像下面这样

  ```shell
  influx --precision rfc3339
  ```

  influxDB规定了很多时间精度，具体可以在命令行输出help查看

  ```shell
  precision <format>    specifies the format of the timestamp: rfc3339, h, m, s, ms, u or ns
  # 可指定的时间精度
  ```

  - 使用influxDB内置CLI执行查询操作

    还是查询我们刚刚插入的那条数据,在命令行中输入以下命令

    ```shell
    curl -G 'http://localhost:8086/query?pretty=true' --data-urlencode "db=first_db" --data-urlencode "q=SELECT \"cost\" FROM \"product\" WHERE \"productName\"='disk'"
    ```

    得到输出为json结构的查询结果![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxzegqffyhj31x50u0jvr.jpg)

    influxDB内置的API很大程度简化了后端的开发，使各种项目可以快速上线。

  - 插入操作的API

  在命令行中输入

  ```shell
  curl -i -XPOST 'http://localhost:8086/write?db=first_db' --data-binary 'weather,location=us-midwes temperature=125'
  # 插入一条数据，measurement=weather，tag=location，filed=temperature,时间戳为当地服务器时间
  ```

  - 我们使用postman测试这个插入接口，以确定该接口的header，body等，为接下来使用go编写请求代码做好准备。通过分析URL，我们可知请求的param是db=first_db，--dat-binary这个参数，意味着你的request body必须是raw,而且header的content-Type="text",具体的postman设置参照下图：![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxzq4jt9xkj31qa0qcgoc.jpg)
  - 点击Send之后，可以在下面看到response的statusCode是204，在http协议中，这个状态码意思是返回体中没有内容。![](https://ws4.sinaimg.cn/large/006tNbRwly1fxzqcl2pvtj31q605yq3l.jpg)
  - 我们回到influxDB的terminal中查看一下，可以看到这条数据已经插入成功了。

  ### GO操作influxDB的API实现插入数据

  - 可以利用这样方便的API，编写代码，实现数据的批量采集、管理、展示，这里我用GO对插入数据的操作简单实现。

    ```go
    func main() {
    	reqBody := "weather,location=us-midwes temperature=521 1475839730100400200"
    	rb := []byte(reqBody)
        headers := map[string]string{
    		"Content-type": "text",
    	}
    	resp, _,err := simpleHttpClient.DoRequest("POST","http://10.18.5.30:8086/write?db=first_db",headers,rb,10)
    	if err != nil {
    		panic(err)
    	}
    	fmt.Println(string(resp))
    ```

    - 使用的DoRequest方法来自[这里](https://github.com/hantmac/simple-httpClient)，这个库对golang的http操作进行简单的封装，而且加入了错误处理，timeout异常检测等。

    - 当然也可以使用Go自带的net/http包中的POST方法

      ```go
      reqBody := "weather,location=us-midwes temperature=521 1475839730100400200"
      	//rb := []byte(reqBody)
          rb := io.NewReader(reqBody)
      	resp, err := http.Post("http://10.18.5.30:8086/write?db=first_db","text",rb)
      	if err != nil {
      		panic(err)
      	}
      	fmt.Println(string(resp))
      ```

      - 需要注意的是对request body的类型处理，net/http.post方法要求该参数的类型是io.reader，所以要使用io.NewReader()进行转换。

      ### 总结

      - 以上就是对influxDB的入门介绍，包括基本概念，安装，配置，基本操作（CLI，API）以及使用GO编写操作数据库的代码。但influxDB的奥秘远不止这些，[如需更加深入的研究可参阅官方文档](https://docs.influxdata.com/influxdb/v1.7/)。

<hr />









