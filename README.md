# 使用swag自动生成RESTful API
## 一、swag简介
swag能够将go代码中的合规注释转换成swagger2.0风格的文档，并且主流的web框架都有符合swag的中间件（以swagger-ui的形式展示）。因此这里选用swag工具。

## 二、在Iris项目中的使用方式

### 1、安装
```sh
$ go get -u github.com/swaggo/swag/cmd/swag
```

### 2、在代码中添加注释 （<a href="https://swaggo.github.io/swaggo.io/">注释规则</a>）

* 在main函数上添加General API注释

```go
// @title Iris swagger 使用示例
// @version 1.0
// @description 这是Iris Swaggo 的使用示例.

// @contact.name 林鹏
// @contact.url http://www.swagger.io/support
// @contact.email iterlp@163.com

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html

// @host localhost:8080
// @BasePath /api/v1
func main() {
}
```

* 在handler函数（API函数）上添加API Opration注释

```go
package handles

import (
	"github.com/kataras/iris/v12"
)

// @Summary 测试SayHello
// @Description 向你说Hello
// @Tags 测试
// @Accept mpfd
// @Produce mpfd
// @Param who query string true "人名"
// @Success 200 {string} string "成功请求响应"
// @Failure 400 {string} string "失败请求响应"
// @Router /hello [get]
func SayHello(c iris.Context) {
	who := c.URLParam("who")
	c.WriteString("hello " + who)
}

type UserInfo struct {
	ID int `json:"id"`
	Name string `json:"name"`
	Age int `json:"age"`
	Company string `json:"company"`
}

// @Summary 根据用户ID获取用户信息
// @Description get userInfo
// @Tags 测试
// @Accept mpfd
// @Produce json
// @Param id query int true "用户id"
// @Success 200 {object} UserInfo "成功请求响应"
// @Failure 400 {object} ErrResponse "失败请求响应"
// @Router /getUser [get]
func PostInfo(c iris.Context) {
	id, _ := c.URLParamInt("id")
	if id == 1 {
		c.JSON(Response{
			Status: true,
			Data:   UserInfo{
				ID:      id,
				Name:    "林鹏",
				Age:     25,
				Company: "XXXX",
			},
		})
		return
	}

	c.JSON(Response{
		Status: false,
		Data:   nil,
	})
}

type Response struct {
	Status bool `json:"status"`
	Data interface{} `json:"data"`
}
```

### 3、根据注释生成swagger文档

```sh
$ swag init  // 在项目根目录下运行
```
此时项目根目录下自动生成了docs目录。

### 4、将生成docs包导入项目中，同时在代码中引入iris的swagger中间件

```go
import (
	"github.com/iris-contrib/swagger/v12"
	"github.com/iris-contrib/swagger/v12/swaggerFiles"

	_ "github.com/Niclausse/iris-swagger-demo/docs" // docs is generated by Swag CLI, you have to import it.
)
```

```go
func main() {
	app := iris.New()

	config := &swagger.Config{
		URL: "http://localhost:8080/swagger/doc.json", //The url pointing to API definition
	}
	// use swagger middleware to
	app.Get("/swagger/{any:path}", swagger.CustomWrapHandler(config, swaggerFiles.Handler))

	v1 := app.Party("/api/v1")
	{
		v1.Handle(http.MethodGet, "/hello", handles.SayHello)
		v1.Handle(http.MethodGet, "/getUser", handles.PostInfo)
	}

	app.Run(iris.Addr(":8080"))
}
```

### 5、启动项目，访问Swagger-UI
`http://localhost:8080/swagger/index.html `

* 示例地址 https://github.com/Niclausse/iris-swagger-demo