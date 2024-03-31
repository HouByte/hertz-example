# Hertz Example

> 官网：[cloudwego/hertz](https://www.cloudwego.io/zh/docs/hertz/getting-started/)

## 官方Demo

1. 安装依赖

```shell
go mod tidy
```

2. 编写第一个demo

```go
package main

import (
	"context"

	"github.com/cloudwego/hertz/pkg/app"
	"github.com/cloudwego/hertz/pkg/app/server"
	"github.com/cloudwego/hertz/pkg/common/utils"
	"github.com/cloudwego/hertz/pkg/protocol/consts"
)

func main() {
	h := server.Default()

	h.GET("/ping", func(c context.Context, ctx *app.RequestContext) {
		ctx.JSON(consts.StatusOK, utils.H{"message": "pong"})
	})

	h.Spin()
}

```

3. 启动程序
```shell
go run .\hertz_demo\
```

日志打印出这两行，说明程序启动成功
> 2022/05/17 21:47:09.626332 engine.go:567: [Debug] HERTZ: Method=GET    absolutePath=/ping   --> handlerName=main.main.func1 (num=2 handlers)
> 
> 2022/05/17 21:47:09.629874 transport.go:84: [Info] HERTZ: HTTP server listening on address=[::]:8888

4. 测试接口

```shell
curl http://127.0.0.1:8888/ping
```