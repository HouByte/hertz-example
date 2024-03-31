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

## hz 构建项目

### 安装

> 确保 GOPATH 环境变量已经被正确的定义（例如 export GOPATH=~/go）并且将 $GOPATH/bin 添加到 PATH 环境变量之中（例如 export PATH=$GOPATH/bin:$PATH）；请勿将 GOPATH 设置为当前用户没有读写权限的目录

```shell
go install github.com/cloudwego/hertz/cmd/hz@latest
```

> 验证是否安装成功 hz -v, 如果显示如下版本的信息，则说明安装成功
> 
> hz version v0.x.x

> 注意，由于 hz 会为自身的二进制文件创建软链接，因此请确保 hz 的安装路径具有可写权限。

### 创建项目

简单创建

```shell
# 整理
go mod init # 上一步在 GOPATH 下执行不会生成 go.mod

# GOPATH 下执行，go mod 名字默认为当前路径相对 GOPATH 的路径，也可自己指定
hz new

# 或者

# 在 GOPATH 外执行，需要指定 go mod 名，
hz new -module hertz-example

# 拉取依赖
go mod tidy
```

编译程序

```shell
go build
```

执行目录下的可执行文件

```shell
./{{your binary}}
```

测试生成的接口
```shell
curl 127.0.0.1:8888/ping
```

## 基于protobuf IDL

### 创建项目

> 当前项目创建idl目录

1. 创建 api.proto
```protobuf
// idl/api.proto; 注解拓展
syntax = "proto2";

package api;

import "google/protobuf/descriptor.proto";

option go_package = "/api";

extend google.protobuf.FieldOptions {
    optional string raw_body = 50101;
    optional string query = 50102;
    optional string header = 50103;
    optional string cookie = 50104;
    optional string body = 50105;
    optional string path = 50106;
    optional string vd = 50107;
    optional string form = 50108;
    optional string js_conv = 50109;
    optional string file_name = 50110;
    optional string none = 50111;

    // 50131~50160 used to extend field option by hz
    optional string form_compatible = 50131;
    optional string js_conv_compatible = 50132;
    optional string file_name_compatible = 50133;
    optional string none_compatible = 50134;
    // 50135 is reserved to vt_compatible
    // optional FieldRules vt_compatible = 50135;

    optional string go_tag = 51001;
}

extend google.protobuf.MethodOptions {
    optional string get = 50201;
    optional string post = 50202;
    optional string put = 50203;
    optional string delete = 50204;
    optional string patch = 50205;
    optional string options = 50206;
    optional string head = 50207;
    optional string any = 50208;
    optional string gen_path = 50301; // The path specified by the user when the client code is generated, with a higher priority than api_version
    optional string api_version = 50302; // Specify the value of the :version variable in path when the client code is generated
    optional string tag = 50303; // rpc tag, can be multiple, separated by commas
    optional string name = 50304; // Name of rpc
    optional string api_level = 50305; // Interface Level
    optional string serializer = 50306; // Serialization method
    optional string param = 50307; // Whether client requests take public parameters
    optional string baseurl = 50308; // Baseurl used in ttnet routing
    optional string handler_path = 50309; // handler_path specifies the path to generate the method

    // 50331~50360 used to extend method option by hz
    optional string handler_path_compatible = 50331; // handler_path specifies the path to generate the method
}

extend google.protobuf.EnumValueOptions {
    optional int32 http_code = 50401;

// 50431~50460 used to extend enum option by hz
}

extend google.protobuf.ServiceOptions {
    optional string base_domain = 50402;

    // 50731~50760 used to extend service option by hz
    optional string base_domain_compatible = 50731;
}

extend google.protobuf.MessageOptions {
    // optional FieldRules msg_vt = 50111;

    optional string reserve = 50830;
    // 550831 is reserved to msg_vt_compatible
    // optional FieldRules msg_vt_compatible = 50831;
}
```

> api.proto 是 hz 提供的注解文件，内容如下，请在使用了注解的 proto 文件中，import 该文件。
> 
> 如果想自行拓展注解的使用，请不要以 “5” 作为序号的开头，避免出现冲突。例如 “optional string xxx = 77777;"。


2. 创建主 IDL
```protobuf
// idl/hello/hello.proto
syntax = "proto3";

package hello;

option go_package = "hertz/hello";

import "api.proto";

message HelloReq {
   string Name = 1[(api.query)="name"];
}

message HelloResp {
   string RespBody = 1;
}

service HelloService {
   rpc Method1(HelloReq) returns(HelloResp) {
      option (api.get) = "/hello";
   }
}
```
3. 创建新项目
```shell
# 在 GOPATH 外执行，需要指定 go mod 名，如果主 IDL 的依赖和主 IDL 不在同一路径下，需要加入 "-I" 选项，其含义为 IDL 搜索路径，等同于 protoc 的 "-I" 命令

hz new -module hertz-example -I idl -idl idl/hello/hello.proto

# 整理 & 拉取依赖
go mod tidy
```
4. 修改handler,修改自己的逻辑

```go
// handler path: biz/handler/hello/hello_service.go
// 其中 "/hello" 是 protobuf idl 中 go_package 的最后一级
// "hello_service.go" 是 protobuf idl 中 service 的名字，所有 service 定义的方法都会生成在这个文件中


// Code generated by hertz generator.

package hello

import (
	"context"

	"github.com/cloudwego/hertz/pkg/app"
	"github.com/cloudwego/hertz/pkg/protocol/consts"
	hello "hertz-example/biz/model/hertz/hello"
)

// Method1 .
// @router /hello [GET]
func Method1(ctx context.Context, c *app.RequestContext) {
	var err error
	var req hello.HelloReq
	err = c.BindAndValidate(&req)
	if err != nil {
		c.String(consts.StatusBadRequest, err.Error())
		return
	}

	resp := new(hello.HelloResp)

	//处理逻辑
	resp.RespBody = "hello," + req.Name

	c.JSON(consts.StatusOK, resp)
}
```

编译项目
```shell
go build
```

运行项目

```shell
./{{your binary}}
```

测试
```shell
curl --location --request GET 'http://127.0.0.1:8888/hello?name=hertz'
```

> 结果: `{"RespBody":"hello,hertz"}`

### 更新项目

修改proto文件

```protobuf
// idl/hello/hello.proto
syntax = "proto3";

package hello;

option go_package = "hertz/hello";

import "api.proto";

message HelloReq {
   string Name = 1[(api.query)="name"];
}

message HelloResp {
   string RespBody = 1;
}

message OtherReq {
   string Other = 1[(api.body)="other"];
}

message OtherResp {
   string Resp = 1;
}

service HelloService {
   rpc Method1(HelloReq) returns(HelloResp) {
      option (api.get) = "/hello";
   }
   rpc Method2(OtherReq) returns(OtherResp) {
      option (api.post) = "/other";
   }
}

service NewService {
   rpc Method3(OtherReq) returns(OtherResp) {
      option (api.get) = "/new";
   }
}
```

更新项目

```shell
hz update -I idl -idl idl/hello/hello.proto
```

* 如果主 IDL 的依赖和主 IDL 不在同一路径下，需要加入 -I 选项，其含义为 IDL 搜索路径，等同于 protoc 的 -I 命令。
* 在编写 update 命令时，不仅需要指定定义 service 的 IDL 文件，还需要指定所有的依赖文件，因为 protobuf 的依赖文件不会自动更新。

> 可以看到 在 biz/handler/hello/hello_service.go 下新增了新的方法； 在 biz/handler/hello 下新增了文件 new_service.go 以及对应的 “Method3” 方法。

## 目录结构

```text
.
├── biz                                // business 层，存放业务逻辑相关流程
│   ├── handler                        // 存放 handler 文件
│   │   ├── hello                      // hello/example 对应 thrift idl 中定义的 namespace；而对于 protobuf idl，则是对应 go_package 的最后一级
│   │   │   └── example
│   │   │       └── hello_service.go   // handler 文件，用户在该文件里实现 IDL service 定义的方法，update 时会查找当前文件已有的 handler 并在尾部追加新的 handler
│   │   └── ping.go                    // 默认携带的 ping handler，用于生成代码快速调试，无其他特殊含义
│   ├── model                          // idl 内容相关的生成代码
│   │   └── hello                      // hello/example 对应 thrift idl 中定义的 namespace；而对于 protobuf idl，则是对应 go_package 
│   │       └── example
│   │           └── hello.go           // thriftgo 的产物，包含 hello.thrift 定义的内容的 go 代码，update 时会重新生成
│   └── router                         // idl 中定义的路由相关生成代码
│       ├── hello                      // hello/example 对应 thrift idl 中定义的 namespace；而对于 protobuf idl，则是对应 go_package 的最后一级
│       │    ├── hello.go           // hz 为 hello.thrift 中定义的路由生成的路由注册代码；每次 update 相关 idl 会重新生成该文件
│       │    └── middleware.go      // 默认中间件函数，hz 为每一个生成的路由组都默认加了一个中间件；update 时会查找当前文件已有的 middleware 在尾部追加新的 middleware
│       └── register.go                // 调用注册每一个 idl 文件中的路由定义；当有新的 idl 加入，在更新的时候会自动插入其路由注册的调用；勿动
├── go.mod                             // go.mod 文件，如不在命令行指定，则默认使用相对于 GOPATH 的相对路径作为 module 名
├── idl                                // 用户定义的 idl，位置可任意
│   └── hello.thrift
├── main.go                            // 程序入口
├── router.go                          // 用户自定义除 idl 外的路由方法
├── router_gen.go                      // hz 生成的路由注册代码，用于调用用户自定义的路由以及 hz 生成的路由
├── .hz                                // hz 创建代码标志，无需改动
├── build.sh                           // 程序编译脚本，Windows 下默认不生成，可直接使用 go build 命令编译程序
├── script                                
│   └── bootstrap.sh                   // 程序运行脚本，Windows 下默认不生成，可直接运行 main.go
└── .gitignore
```