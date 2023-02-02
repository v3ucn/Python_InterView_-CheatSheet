###为什么要学习Go语言？Go语言是现在非常火爆🔥的一门编程语言，极大地提高了后端编程的效率，同时有着极高的性能。借助Go语言我们 可以用同步的方式写出高并发的服务端软件，同时，Go语言也是云原生第一语言，Docker，Kubernetes等等著名的项目都是使用Go语言实现的。

###Go语言之所以如此火爆，是因为Go语言既有动态语言的开发效率，又能有静态语言的编译期检验，编译通过之后基本上不会遇到崩溃，再加上 Go语言简单而又刚好足够的语法，以及统一的内置工具和编码风格，使得用户能够轻松的和团队里的其他人协作。

###静态页生成器 hugo 
攻略:https://v3u.cn/a_id_81

###为什么要用go来改造爬虫，主要还是因为python的性能瓶颈，而go除了本身的高性能以外，其内置的bufio模块可以提高http请求速率以及文件操作的性能，与此同时还可以很轻松的应用并发下载及其断点续传功能，所以有的时候需要为了高性能而牺牲一部分的开发效率

代码如下：
```
package main

import (
    "net/http"
    "log"
    "time"
    "net/url"
    "path"
    "os"
    "io"
    "bufio"
    "math"
    "strconv"
)
//定义下载地址
var durl = "https://video.pearvideo.com/mp4/adshort/20190515/cont-1554493-13908887_adpkg-ad_hd.mp4";

func main() {
    uri, err := url.ParseRequestURI(durl)
    if err != nil {
        panic("网址错误")
    }

    filename := path.Base(uri.Path)
    log.Println("[*] Filename " + filename)

    client := http.DefaultClient;
    client.Timeout = time.Second * 60 //设置超时时间
    resp, err := client.Get(durl)
    if err != nil {
        panic(err)
    }
    if resp.ContentLength <= 0 {
        log.Println("[*] Destination server does not support breakpoint download.")
    }
    raw := resp.Body
    defer raw.Close()
    reader := bufio.NewReaderSize(raw, 1024*32);


    file, err := os.Create(filename)
    if err != nil {
        panic(err)
    }
    writer := bufio.NewWriter(file)

    buff := make([]byte, 32*1024)
    written := 0
    go func() {
        for {
            nr, er := reader.Read(buff)
            if nr > 0 {
                nw, ew := writer.Write(buff[0:nr])
                if nw > 0 {
                    written += nw
                }
                if ew != nil {
                    err = ew
                    break
                }
                if nr != nw {
                    err = io.ErrShortWrite
                    break
                }
            }
            if er != nil {
                if er != io.EOF {
                    err = er
                }
                break
            }
        }
        if err != nil {
            panic(err)
        }
    }()

    spaceTime := time.Second * 1
    ticker := time.NewTicker(spaceTime)
    lastWtn := 0
    stop := false

    for {
        select {
        case <-ticker.C:
            speed := written - lastWtn
            log.Printf("[*] Speed %s / %s \n", bytesToSize(speed), spaceTime.String())
            if written-lastWtn == 0 {
                ticker.Stop()
                stop = true
                break
            }
            lastWtn = written
        }
        if stop {
            break
        }
    }
}

func bytesToSize(length int) string {
    var k = 1024 // or 1024
    var sizes = []string{"Bytes", "KB", "MB", "GB", "TB"}
    if length == 0 {
        return "0 Bytes"
    }
    i := math.Floor(math.Log(float64(length)) / math.Log(float64(k)))
    r := float64(length) / math.Pow(float64(k), i)
    return strconv.FormatFloat(r, 'f', 3, 64) + " " + sizes[int(i)]
}

```