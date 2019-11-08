# 代码部分（使用negroni框架)
## main.go


```
package main

import (
    "os"
    "service"
    flag "github.com/spf13/pflag"
)//引入包

const (
    PORT string = "8080"
)//定义默认端口为8080

func main() {
    port := os.Getenv("PORT")//获得环境下端口
    if len(port) == 0 {
        port = PORT//若无环境下端口，则使用8080端口
    }

    pPort := flag.StringP("port", "p", PORT, "PORT for httpd listening")//定义命令行端口参数
    flag.Parse()//解析参数
    if len(*pPort) != 0 {
        port = *pPort
    }//获得命令行端口参数

    server := service.NewServer()//为服务器初始化
    server.Run(":" + port)//运行服务器
}
```
## service/server.go
```
package service

import (
    "net/http"

    "github.com/codegangsta/negroni"
    "github.com/gorilla/mux"
    "github.com/unrolled/render"
)//引入包

// 配置并返回服务器
func NewServer() *negroni.Negroni {

    formatter := render.New(render.Options{
        IndentJSON: true,
    })

    n := negroni.Classic()
    mx := mux.NewRouter()

    initRoutes(mx, formatter)//定义路由器

    n.UseHandler(mx)
    return n
}

func initRoutes(mx *mux.Router, formatter *render.Render) {
    mx.HandleFunc("/hello/{id}", testHandler(formatter)).Methods("GET")
}

func testHandler(formatter *render.Render) http.HandlerFunc {

    return func(w http.ResponseWriter, req *http.Request) {
        vars := mux.Vars(req)
        id := vars["id"]
        formatter.JSON(w, http.StatusOK, struct{ Test string }{"Hello " + id})
    }
}
```

# 测试部分
## curl 测试

```
[root@localhost ~]# curl -v http://localhost:9090/hello/testuser
* About to connect() to localhost port 9090 (#0)
*   Trying ::1...
* Connected to localhost (::1) port 9090 (#0)
> GET /hello/testuser HTTP/1.1
> User-Agent: curl/7.29.0
> Host: localhost:9090
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: application/json; charset=UTF-8
< Date: Fri, 08 Nov 2019 03:27:09 GMT
< Content-Length: 31
<
{
  "Test": "Hello testuser"
}
* Connection #0 to host localhost left intact
```

## ab 测试

```
[root@localhost ~]# ab -n 1000 -c 100 http://localhost:9090/hello/your
This is ApacheBench, Version 2.3 <$Revision: 1430300 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:
Server Hostname:        localhost
Server Port:            9090

Document Path:          /hello/your
Document Length:        27 bytes

Concurrency Level:      100
Time taken for tests:   0.103 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      150000 bytes
HTML transferred:       27000 bytes
Requests per second:    9736.25 [#/sec] (mean)
Time per request:       10.271 [ms] (mean)
Time per request:       0.103 [ms] (mean, across all concurrent requests)
Transfer rate:          1426.21 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    4   1.0      4       6
Processing:     1    6   3.4      5      22
Waiting:        0    5   3.5      4      20
Total:          5   10   3.2      9      26

Percentage of the requests served within a certain time (ms)
  50%      9
  66%      9
  75%      9
  80%     10
  90%     16
  95%     19
  98%     19
  99%     20
 100%     26 (longest request)
```


