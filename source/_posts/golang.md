---
title: Golang
tags: golang
categories: golang
---

# Go

## 环境搭建

- 安装 Go
    - 标准安装
        1. 下载二进制包  
            `curl -OkL https://golang.google.cn/dl/go1.18.linux-amd64.tar.gz`
            > - [window 版本下载](https://studygolang.com/dl/golang/go1.16.3.windows-amd64.zip)
        2. 将下载的二进制包解压至 /usr/local目录  
            `tar -C /usr/local -xzf go*.tar.gz`
        3. 将 /usr/local/go/bin 目录添加至 PATH 环境变量  
            `export PATH=$PATH:/usr/local/go/bin`

- [LiteIDE](https://github.com/visualfc/liteide)
    ```
    http://sourceforge.mirrorservice.org/l/li/liteide/x36/liteidex36.linux64-qt5.5.1.tar.gz
    http://sourceforge.mirrorservice.org/l/li/liteide/x36/liteidex36.windows-qt5.9.5.zip
    ```

- 国内加速
    ```bash
    export GO111MODULE=auto
    # export GOPROXY="https://goproxy.io"
    export GOPROXY="https://mirrors.aliyun.com/goproxy/"
    ```

- 获取第三方库
    ```bash
    # go get github.com/go-sql-driver/mysql
    go install github.com/go-sql-driver/mysql@latest
    ```

## 编译运行

1. 新建文件 greeting.go 并编辑内容如下
    ```golang
    package main

    import "fmt"

    func main() {
        fmt.Println("hello, world")
    }
    ```

2. 直接运行  
    `go run greeting.go`

3. 编译为可执行文件  
    `go build greeting.go`

### 交叉编译

- Linux 下编译 Mac、Windows 平台的 64 位可执行程序
    ```bash
    CGO_ENABLED=0 # 编译成单个可执行文件
    GOOS=darwin # Mac
    GOARCH=amd64 # x64
    go build main.go # 编译
    ```
    ```bash
    CGO_ENABLED=0
    GOOS=windows # Windows
    GOARCH=amd64 # x64
    go build main.go # 编译
    ```

- Windows 下编译 Mac、Linux 平台的 64 位可执行程序
    ```batch
    SET CGO_ENABLED=0
    SET GOOS=darwin3
    SET GOARCH=amd64
    go build main.go
    ```
    ```batch
    SET CGO_ENABLED=0
    SET GOOS=linux
    SET GOARCH=amd64
    go build main.go
    ```

- Mac 下编译 Linux、Windows 平台的 64 位可执行程序
    ```bash
    CGO_ENABLED=0
    GOOS=linux
    GOARCH=amd64
    go build main.go
    ```
    ```bash
    CGO_ENABLED=0
    GOOS=windows
    GOARCH=amd64
    go build main.go
    ```


> 参数描述：
> - CGO_ENABLED  
>   当 `CGO_ENABLED=1`， 进⾏编译时， 会将⽂件中引⽤libc的库（⽐如常⽤的 net 包），以动态链接的⽅式⽣成⽬标⽂件。  
>   当 `CGO_ENABLED=0`， 进⾏编译时， 则会把在⽬标⽂件中未定义的符号（外部函数）⼀起链接到可执⾏⽂件中。
> - GOOS  
>   目标可执行程序运行操作系统，支持 `darwin`、`freebsd`、`linux`、`windows`
> - GOARCH  
>   目标可执行程序操作系统构架，包括 `386`、`amd64`、`arm`

## 语法

- 基本数据类型
    - 布尔型  
        布尔型的值只可以是常量 `true` 或者 `false`（默认为 false）
        ```go
        var a bool = true
        ```
    - 数字类型  
        整型 int 和浮点型 float32、float64，支持复数，其中位的运算采用补码
        ```go
        var a int; // 32 位有符号整数
        ```
        - 整型
            | 位长度（bit） | 有符号 | 无符号 |
            | --- | --- | --- |
            | 8 | `int8` (-128 - 127) | `uint8` (0 - 255) |
            | 16 | `int16` (-32768 - 32767) | `uint16` (0 - 65535) |
            | 32 | `int32` (-2147483648 - 2147483647) | `uint32` (0 - 4294967295) |
            | 64 | `int64` (-9223372036854775808 - 9223372036854775807) | `uint64` (0 - 18446744073709551615) |
        - 浮点型
            | 位长度（bit） | |
            | --- | --- |
            | 32 | `float32` |
            | 64 | `float64` |
        - 复数
            | 位长度（bit） | |
            | --- | --- |
            | 32 | `complex64` |
            | 64 | `complex128` |
        - 其他
            | | |
            | --- | --- |
            | `byte` | 同 uint8 |
            | `rune` | 同 int32 |
            | `uint` | 即 uint32 |
            | `int` | 即 int32 |
            | `uintptr` | 无符号整型，用于存放一个指针 |
    - 字符串类型
        字符串的字节使用 UTF-8 编码标识 Unicode 文本（默认值为空字符串）
        ```go
        const b string = "abc" // 显式指定 string 类型
        const b = "abc" // 隐式指定 string 类型
        ```
    - 派生类型
        - 指针类型（Pointer）  
            - chan，即 Channel
            - map
            - slice，即切片
            - 函数，如 `fun := func(s string) string { return "modify" + s }`
            - interface，一个类型为 interface{} 或者 interface 接口的变量可能是指针变量，也可能是普通变量
            - function type，如 `type Func func(string)`
            其它非指针类型有：struct、基本数据类型、数组（区别切片）
        - 数组类型（[len]byte）
        - 结构化类型（struct）
            ```go
            type User struct {
                Name    string
                Gender  bool
                Age     int
                Address string
                Parents []User
            }

            // 方法一：基本的实例化：结构体本身是一种类型，可以像整型、字符串等类型一样，以 var 的方式声明结构体即可完成实例化
            var user User
            user.Name = "zhangsan"
            user.Age = 20

            // 方法二：创建指针类型的结构体：使用 new 关键字对类型（包括结构体、整型、浮点数、字符串等）进行实例化，结构体在实例化后会形成指针类型的结构体
            user2 := new(User)
            user2.Name = "lisi"
            user2.Age = 21

            // 方法三：取结构体的地址实例化：对结构体进行 & 取地址操作时，视为对该类型进行一次 new 的实例化操作
            user3 := &User{}
            user3.Name = "wangwu"
            user3.Age = 22
            ```
            ```go
            // 匿名结构体
            user := struct {
                name string
                age int
            } {
                name: "zhangsan,
                age: 18
            }
            ```
        - Channel 类型
        - 函数类型
        - 切片类型
        - 接口类型（interface）
        - Map 类型（map）
            ```go
            var m map[string]int // 定义 map
            m = make(map[string]int) // 初始化 map
            m["a"] = 1 // 赋值

            v, ok = m["a"]
            fmt.Println(v, ok) // 1 true

            v2, ok2 = m["c"] // 无法取出 c 的值
            fmt.Println(v, ok) // 0 false
            ```
            ```go
            var m = map[string]int {
               "a": 1,
               "b": 2
            }
            fmt.Println(m) // map[a:1 b:2]
            ```

- [数组和切片](https://blog.csdn.net/gaobinzhan/article/details/106109160)
    - 数组：容量不可伸缩，两个数组可以比较
        ```go
        // 数组初始化
        // var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
        // balance := [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
        balance := [...]float32{1000.0, 2.0, 3.4, 7.0, 50.0} // 使用 ... 代替数组的长度，编译器会根据元素个数自行推断数组的长度

        len(balance) // 获取数组长度
        ```
        声明
        ```go
        var a [3]int // 声明并初始化为默认零值
	    a[0] = 1
	    b := [3]int{1, 2, 3}           // 声明同时初始化
	    c := [2][2]int{{1, 2}, {3, 4}} // 多维数组初始化
	    t.Log(a[0], a[2])
	    t.Log(b[2])
	    t.Log(c[1][1])
        ```
        遍历
        ```go
        arr := [...]int{1, 2, 3, 4, 5, 6} // 自动判断长度

	    for i := 0; i < len(arr); i++ { // 典型写法遍历数组
		    t.Log(arr[i])
	    }

	    for idx, e := range arr { // 相当于其它语言的foreach
		    t.Log(idx, e)
	    }

	    for _, e := range arr { // 我们可能用不到 idx 但go语言定义一个值不去使用编译会不通过 使用_代表不关心这个结果，来占位
		    t.Log(e)
	    }
        ```
        截取，格式：a[开始索引(包含):结束索引(不包含)]
        ```go
        arr := [...]int{1, 2, 3, 4, 5, 6}
	    // a[开始索引(包含):结束索引(不包含)]
	    t.Log(arr[0:1]) // 1
	    t.Log(arr[2:]) // 3 4 5 6
	    t.Log(arr[1:len(arr)]) // 2 3 4 5 6
	    t.Log(arr[1:3]) // 2 3
        ```
    - 切片：容量可以伸缩，两个切片不能比较
        声明
        ```go
        var s0 []int            // 定义看起来特别像数组，但没有指定长度
	    t.Log(len(s0), cap(s0)) // 0 0
	    s0 = append(s0, 1)
	    t.Log(len(s0), cap(s0)) // 1 1

	    s1 := []int{1, 2, 3, 4} // 初始化一个切片
	    t.Log(len(s1), cap(s1)) // 4 4

	    // []type,len,cap 其中len个元素会被初始化为默认零值，未初始化元素不可以访问
	    s2 := make([]int, 3, 5)    // len为3 cap为5
	    t.Log(len(s2), cap(s2))    // 3 5
	    t.Log(s2[0], s2[1], s2[2]) // 成功被初始化 结果：0 0 0
	    //t.Log(s2[0], s2[1], s2[2], s2[3]) // 出现了一个错误 index out of range [3]
	    s2 = append(s2, 1)
	    t.Log(s2[0], s2[1], s2[2], s2[3]) // 0 0 0 1
	    t.Log(len(s2), cap(s2))
        ```
        遍历
        ```go
        s := []int{}
	    for i := 0; i < 10; i++ {
		    s = append(s, i) // 为什么重新赋值给s,是因为结构体指向的连续存储空间进行了变化,并把原有的连续存储空间拷贝到新的连续存储空间
		    t.Log(len(s), cap(s))
	    }
        ```

- 循环
    ```go
    sum := 0;
    for index := 0; index < 10; index++ {
        sum += index
        // break
        // continue
    }
    fmt.Println("sum is equal to", sum)
    ```
    ```go
    var user = map[string]string {
        "name": "zhangsan",
        "age": "20"
    }
    for k, v := range user { // 遍历 map 类型
        fmt.Println("map's key:", k)
        fmt.Println("map's value:", v)
    }
    ```

- 并发
    - 协程（即轻量级线程）/goroutine
        ```go
        import (
            "fmt"
            "time"
        )
        func say(s string) {
            for i := 0; i < 5; i++ {
                time.Sleep(100 * time.Millisecond)
                fmt.Println(s)
            }
        }
        func main() {
            go say("world") // 使用 go 语句开启一个新的运行期线程（即 goroutine），以一个不同的、新创建的 goroutine 来执行一个函数。 同一个程序中的所有 goroutine 共享同一个地址空间
            say("hello")
        }
        ```
    - chan（类似于 Java 的 BlockingQueue）
        ```go
        import (
            "fmt"
        )
        func main() {
            c := make(chan int, 10) // 创建一个（双向）通道，指定缓冲区大小位 10
            go fibonacci(cap(c), c)
            for i := range c { // range 函数遍历每个从通道接收到的数据，因为 c 在发送完 10 个数据之后就关闭了通道，所以这里我们 range 函数在接收到 10 个数据之后就结束了。如果上面的 c 通道不关闭，那么 range 函数就不会结束，从而在接收第 11 个数据的时候就阻塞了。
                fmt.Println(i)
            }
        }
        func fibonacci(n int, c chan int) {
            x, y := 0, 1
            for i := 0; i < n; i++ {
                c <- x // 把变量 x 发送到通道 c
                x, y = y, x + y
            }
            close(c) // 关闭通道
        }
        ```

- 延迟调用 defer  
    defer 修饰的语句会被延迟执行，在 defer 所在的函数返回或执行结束前，会将 defer 语句的按照逆序进行执行（类似栈，即后进先出）。Go 语言中的 defer 类似于 Java 或 C# 的 finally 语句块，一般用于释放某些已分配的资源，典型的例子就是对一个互斥解锁，或者关闭一个文件。
    ```go
    func main() {
        fmt.Println("defer begin")
        // 将defer放入延迟调用栈
        defer fmt.Println(1)
        defer fmt.Println(2)
        // 最后一个放入, 位于栈顶, 最先调用
        defer fmt.Println(3)
        fmt.Println("defer end")
    }

    // 代码输出如下
    // defer begin
    // defer end
    // 3
    // 2
    // 1
    ```

## 标准库

- 定时任务
    ```go
    import "time"
    ticker := time.NewTicker(time.Minute * 1)
    go func() {
        // 每隔一分钟打印当前时间
        for _ = range ticker.C {
            fmt.Printf("\rnow time is %v", time.Now())
        }
    }()
    ```

- 正则
    ```go
    import "regexp"

    re, _ := regexp.Compile("[a-z]{2,4}")

    a := "I am learning Go language"

    // 查找符合正则的第一个
    one := re.Find([]byte(a))
    fmt.Println(string(one)) // am

    // 查找符合正则的所有 slice
    all := re.FindAll([]byte(a), -1) // n (这里是 -1) 小于 0,表示返回全部符合的字符串，不然就是返回指定的长度
    fmt.Println(all)
    ```

- json
    ```go
    import "encoding/json"

    type User struct {
        Name    string `json:"name"`
        Gender  bool   `json:"gender,omitempty"` // "omitempty" 表示该字段如果没有提供，在序列化成 json 的时候就不要包含其默认值
        Age     int    `json:"age"`
        Address string `json:"-"` // "-" 表示不进行序列化
    }

    user := &User{}
    user.Name = "zhangsan"
    user.Gender = true
    user.Age = 22
    user.Address = "earth"

    // 序列化
    data, _ := json.Marshal(user)
    fmt.Println(string(data)) // {"name":"zhangsan","gender":true,"age":22}

    // 反序列化
    u := &User{}
    err := json.Unmarshal([]byte(data), u)
    if err != nil {
        panic(err)
    }
    fmt.Println(*u) // {zhangsan true 22 }
    ```

- embed（需 goland 版本 ≥ 1.16）
    ```go
    package main

    import (
        "embed"
        "net/http"
    )

    //go:embed js/* css/* img/* *.html robot.txt
    var fileList embed.FS

    func main() {
        http.Handle("/", http.FileServer(http.FS(fileList)))
        http.ListenAndServe(":8080", nil)
    }
    ```
    > 原理是使用 `//go:embed` 标签来完成  
    > 1. 文件不是 utf8 编码时，输出内容为中文会乱码
    > 2. 测试过嵌入文件只能为源码文件同级目录和子目录下的文件，其他目录的绝对路径或相对路径会报错
    > 3. 如果路径包含空格可以使用双引号或反引号括起来
    > 4. 变量的类型只能是 `string`、`[]byte`、`embed.FS`，即使是这三个类型的别名也不行

- unsafe
    ```go
    import (
        "fmt"
        "unsafe"
    )

    var flag bool
    var n1 int64 = 10
    var name string = "小白"

    fmt.Printf("int的字节大小", unsafe.Sizeof(n1)) // 获取 int 的字节大小
    fmt.Println()
    fmt.Printf("string的字节大小", unsafe.Sizeof(name))
    fmt.Println()
    fmt.Printf("bool的字节大小", unsafe.Sizeof(flag))
    ```

- runtime
    ```go
    import (
        "fmt"
        "runtime"
    )

    var m runtime.MemStats
    runtime.ReadMemStats(&m)

    fmt.Printf("%+v\n", m) // 获取内存信息
    fmt.Printf("os %d\n", m.Sys)
    ```

### 网络

- http 静态服务器
    ```golang
    package main
    
    import ("net/http")
    
    func main() {
        http.Handle("/", http.FileServer(http.Dir(".")))
        http.ListenAndServe(":8080", nil)
    }
    ```

- http 服务器
    ```golang
    package main

    import (
        "fmt"
        "net/http"
        "strings"
        "log"
    )

    func main() {
        http.HandleFunc("/greeting", func (w http.ResponseWriter, r *http.Request) {
            r.ParseForm() // 解析参数，默认是不会解析的
            fmt.Println(r.Form) // 这些信息是输出到服务器端的打印信息
            fmt.Println("path", r.URL.Path)
            fmt.Println("scheme", r.URL.Scheme)
            fmt.Println(r.Form["url_long"])
            for k, v := range r.Form {
                fmt.Println("key:", k)
                fmt.Println("val:", strings.Join(v, ""))
            }
            fmt.Fprintf(w, "<html><body>hello, world</body></html>") // 这个写入到 w 的是输出到客户端的
        })

        err := http.ListenAndServe(":8080", nil) // 设置监听的端口
        if err != nil {
            log.Fatal("Fail to start server", err)
        }
    }
    ```
    ```go
    package main

    import (
        "io"
        "encoding/json"
        "fmt"
        // "html"
        "io/ioutil"
        "log"
        "net/http"
    )

    type Order struct {
        AppId      string `json:"appid"`
        MchId      string `json:"mch_id"`
        OutTradeNo string `json:"out_trade_no"`
        TotalFee   int    `json:"total_fee"`
        NonceStr   string `json:"nonce_str"`
        Sign       string `json:"sign"`
        SignType   string `json:"sign_type"`
    }

    type Result struct {
        Code string `json:"code"`
        Msg  string `json:"msg"`
    }

    func main() {
        // curl http://127.0.0.1:8002/pay/unifiedorder -XPOST -d '{"appid":"1","mch_id":"1","out_trade_no":"1","total_fee":1}'
        http.HandleFunc("/pay/unifiedorder", func (w http.ResponseWriter, r *http.Request) {
            // fmt.Fprintf(w, "the url is %q", html.EscapeString(r.URL.Path))
            if r.Method == "POST" {
                b, err := ioutil.ReadAll(r.Body)
                if err != nil {
                    log.Println("Read failed", err)
                }
                defer r.Body.Close()

                order := &Order{}
                err = json.Unmarshal(b, order)
                if err != nil {
                    log.Println("Json unmarshal error", err)
                }
                log.Println("order", order)

                result := &Result{}
                result.Code = "SUCCESS"
                result_json, _ := json.Marshal(result)
                io.WriteString(w, string(result_json))
            } else {
                fmt.Fprintf(w, "The method " + r.Method + " was not support")
            }
        })
        log.Fatal(http.ListenAndServe(":8002", nil))
    }
    ```

- http 请求
    ```golang
    import (
        "fmt"
        "io/ioutil"
        "net/http"
        "time"
    )

    client := http.Client{
        Timeout: 5 * time.Second,
    }
    resp, err := client.Get("http://www.baidu.com")
    // resp, err := http.Get("http://www.baidu.com")
    if err != nil {
        fmt.Println(err)
        return
    }
    html, _ := ioutil.ReadAll(resp.Body)
    fmt.Println(string(html))
    ```

## 第三方库

- [sqlite3](https://github.com/mattn/go-sqlite3)
    1. 下载
        ```bash
        go get github.com/mattn/go-sqlite3
        ```
    2. 示例
        ```go
        package main

        import (
            "fmt"
            "time"
            "database/sql"
            _ "github.com/mattn/go-sqlite3"
        )

        func main() {
            // 打开/创建
            db, err := sql.Open("sqlite3", "./my.db")

            // 创建表
            _, err = db.Exec(`
                create table if not exists user (
                    uid integer primary key autoincrement,
                    name varchar(128) null,
                    birthday date null
                );
            `)

            // 新增
            stmt, err := db.Prepare("insert into user(name, birthday) values(?, ?)")
            if err != nil {
                panic(err)
            }
            res, err := stmt.Exec("luffy", "2012-12-09") // res 为返回结果
            if err != nil {
                panic(err)
            }
            id, err := res.LastInsertId() // 可以通过 res 取自动生成的 id
            if err != nil {
                panic(err)
            }
            fmt.Println("insert success with id is", id)

            // 更新
            stmt, err = db.Prepare("update user set name = ? where uid = ?")
            if stmt == nil || err != nil {
                panic(err)
            }
            _, err = stmt.Exec("zhangsan", 1)
            if err != nil {
                panic(err)
            }

            // 查询
            rows, err := db.Query("select * from user")
            if err != nil {
                panic(err)
            }
            defer rows.Close()
            for rows.Next() {
                var uid int
                var name string
                var birth time.Time
                err = rows.Scan(&uid, &name, &birth)
                if err != nil {
                    panic(err)
                }
                fmt.Printf("{ \"uid\": %d, \"name\": \"%s\", \"birthday\": \"%s\" }\n", uid, name, birth)
            }

            // 删除
            stmt, err = db.Prepare("delete from user where uid = ?")
            if err != nil {
                panic(err)
            }
            res, err = stmt.Exec(id)
            if err != nil {
                panic(err)
            }

            // 关闭
            db.Close()
        }
        ```

- [Gin](https://github.com/gin-gonic/gin)（需要 Go 版本 ≥ 1.6）
    1. 下载并安装
        ```bash
        # 加速
        go env -w GO111MODULE=on
        go env -w GOPROXY=https://goproxy.cn,direct
        
        go mod init myapp

        # 下载、安装 Gin
        go get -u github.com/gin-gonic/gin
        ```
    2. 示例
        ```go
        package main

        import "github.com/gin-gonic/gin"

        func main() {
            r := gin.Default()
            r.GET("/ping", func(c *gin.Context) {
                c.JSON(200, gin.H{
                    "message": "pong",
                })
            })
            r.Run() // listen and serve on 0.0.0.0:8080
        }
        ```
        ```go
        package main

        import (
            "fmt"
            "net/http"
            "github.com/gin-gonic/gin"
        )

        type Login struct {
            Name string `form:"name" json:"name" xml:"name"  binding:"required"`
            Pass string `form:"pass" json:"pass" xml:"pass" binding:"required"`
        }

        func main() {
            router := gin.Default()

            // curl http://127.0.0.1:8002/greeting/luffy
            router.GET("/greeting/:name", func(c *gin.Context) {
                name := c.Param("name") // 获取 URL 参数
                c.String(http.StatusOK, "hello, %s", name)
            })

            // curl -XPOST http://127.0.0.1:8002/parameter/Andy?q=Bob -d 'f=Charlie'
            // curl -XPOST "http://127.0.0.1:8002/parameter/Andy?qm[a]=Bob&qm[b]=Emma" -d 'fm[a]=Charlie&fm[b]=Frank'
            router.POST("/parameter/:p", func(c *gin.Context) {
                p := c.Param("p") // 获取 URL 参数
                q := c.Query("q") // 获取 URL 后面拼接的参数
                d := c.DefaultQuery("d", "Dick") // 获取 URL 后面拼接的参数，如果没有入参，则使用默认值 Dick
                f := c.PostForm("f") // 获取 form 表单参数
                qm := c.QueryMap("qm") // 获取 URL 后面拼接的参数，返回 Map
                fm := c.PostFormMap("fm") // 获取 form 表单参数，返回 Map
                c.JSON(200, gin.H{
                    "p": p,
                    "q": q,
                    "d": d,
                    "f": f,
                    "qm": qm,
                    "fm": fm,
                })
            })

            // curl -XPOST http://127.0.0.1:8002/login -H "Content-Type: application/json" -d '{ "name": "admin", "pass": "123" }'
            router.POST("/login", func(c *gin.Context) {
                var user Login
                if err := c.ShouldBindJSON(&user); err != nil {
                    c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
                    return
                }
                if user.Name != "admin" || user.Pass != "123" {
                    c.JSON(http.StatusUnauthorized, gin.H{ "status": "unauthorized" })
                    return
                }
                c.JSON(http.StatusOK, gin.H{ "status": "you are logged in" })
            })

            router.MaxMultipartMemory = 8 << 20  // Set a lower memory limit for multipart forms (default is 32 MiB)
            // curl http://127.0.0.1:8002/upload -F "file=@data.zip"
            // curl -X POST http://127.0.0.1:8002/upload -F "file=@data.zip" -H "Content-Type: multipart/form-data"
            router.POST("/upload", func(c *gin.Context) { // 单文件上传
                file, _ := c.FormFile("file")
                fmt.Println("upload with filename", file.Filename)
                c.SaveUploadedFile(file, "./" + file.Filename) // 保存文件至指定位置
                c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded", file.Filename))
            })
            // curl -X POST http://127.0.0.1:8002/uploads -F "upload[]=@data1.zip" -F "upload[]=@data2.zip" -H "Content-Type: multipart/form-data"
            router.POST("/uploads", func(c *gin.Context) { // 多文件上传
                form, _ := c.MultipartForm()
                files := form.File["upload[]"]
                for _, file := range files {
                    fmt.Println("upload with filename %s", file.Filename)
                    c.SaveUploadedFile(file, "./")
                }
                c.String(http.StatusOK, fmt.Sprintf("%d files uploaded", len(files)))
            })

            v1 := router.Group("/v1") // 分组
            {
                // curl http://127.0.0.1:8002/v1/greeting
                v1.GET("/greeting", greeting)
                v1.POST("/greeting2", greeting)
            }

            router.Run(":8002")
        }

        func greeting(c *gin.Context) {
            c.String(http.StatusOK, "hello, world")
        }
        ```

- [otto](https://github.com/robertkrimen/otto)
    ```go
    import "github.com/robertkrimen/otto"

    vm := otto.New()
    // 执行 js 代码
    vm.Run(`
        abc = 2 + 2;
        console.log("The value of abc is " + abc); // 4
    `)

    // 获取变量值
    if value, err := vm.Get("abc"); err == nil {
        if value_int, err := value.ToInteger(); err == nil {
	        fmt.Printf("", value_int, err)
        }
    }

    // 变量赋值
    vm.Set("def", 11)
    vm.Run(`
        console.log("The value of def is " + def);
        // The value of def is 11
    `)
    vm.Set("xyzzy", "Nothing happens.")
    vm.Run(`
        console.log(xyzzy.length); // 16
    `)

    // 获取表达式值
    value, _ = vm.Run("xyzzy.length")
    {
        // value is an int64 with a value of 16
        value, _ := value.ToInteger()
    }
    ```

- [goja](https://github.com/dop251/goja)
    ```go
    import (
        "github.com/dop251/goja"
        "github.com/dop251/goja_nodejs/require"
    )

    registry := require.NewRegistryWithLoader(func(path string) ([]byte, error) { // 创建自定义 require loader（registry 每次重新生成，防止 module 被缓存，从而导致 module 修改后不生效）
        rows, err := Db.Query("select jscontent from script where name = ?", path)
        if err != nil {
            panic(err.Error())
            return nil, err
        }
        defer rows.Close()
        if rows.Next() == false {
            return nil, errors.New("The module was not found.")
        }
        script := Script{}
        err = rows.Scan(&script.JsContent)
        return []byte(script.JsContent), err
    })

    vm := goja.New()

    _ = registry.Enable(vm) // 启用自定义 require loader 

    vm.Set("exports", vm.NewObject())

    time.AfterFunc(60000 * time.Millisecond, func() { // 允许脚本最大执行的时间为 60 秒
        vm.Interrupt("The script executed timeout.")
    })

    // res, err := vm.RunString("require('./" + name + "').main();")

    _, err = vm.RunString(script.JsContent)
    if err != nil {
        panic(err)
        return
    }
    main, isFunction := goja.AssertFunction(vm.Get("main")) // 执行 js 中的 main 函数
    if !isFunction {
        panic(errors.New("The function main can not be found."))
        return
    }
    res, err := main(goja.Undefined(), vm.ToValue("this is parameter"))
    ```

## 第三方应用

- 使用 [httpdump](https://github.com/rogercoll/httpdump) 抓包 HTTP 报文
    ```bash
    apt install libpcap-dev
    # yum install libpcap-devel

    go install github.com/rogercoll/httpdump@latest
    ```
    ```bash
    httpdump
    
    httpdump -level all -port 80

    # parse pcap file
    sudo tcpdump -wa.pcap tcp
    httpdump -file a.pcap

    # capture specified device:
    httpdump -device eth0

    # filter by ip and/or port
    httpdump -port 80  # filter by port
    httpdump -ip 101.201.170.152 # filter by ip
    httpdump -ip 101.201.170.152 -port 80 # filter by ip and port
    ```
