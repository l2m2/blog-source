---
title: 使用gin-swagger自动生成gin项目的API文档
toc: false
date: 2021-06-24 13:35:00
description: ...
tags:
- Go
---

1. 下载[Swag](https://github.com/swaggo/swag)

   ```shell
   $ go get -u github.com/swaggo/swag/cmd/swag
   ```

2. 在`main.go`的同级目录下执行初始化命令

   ```shell
   $ swag init
   ```

   执行后会自动生成一个docs目录，下面包含以下几个文件

   ```shell
   $ ls
   docs.go  swagger.json  swagger.yaml
   ```

3. 下载[gin-swagger](https://github.com/swaggo/gin-swagger)

   ```shell
   $ go get -u github.com/swaggo/gin-swagger
   $ go get -u github.com/swaggo/files
   ```

4. 在添加路由的地方加上文档的路由

   ```go
   package api
   
   import (
   	"github.com/gin-gonic/gin"
   	_ "github.com/l2m2/xpm/docs"
   	ginSwagger "github.com/swaggo/gin-swagger"
   	"github.com/swaggo/gin-swagger/swaggerFiles"
   )
   
   func IncludeRouter(r *gin.Engine) {
   	r.GET("/docs/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
   }
   ```

   **注意**：一定要import刚才生成的docs包，例如在我的项目中是`github.com/l2m2/xpm/docs`

5. 在`main.go`中添加全局描述

   ```go
   // @title XPM API
   // @version 1.0
   // @description XPM API, automatically generated by gin-swagger
   // @BasePath /v1
   func main() {
   	app := gin.Default()
   	api.IncludeRouter(app)
   
   	app.Run(":8000")
   }
   ```

6. 添加接口注释

7. 重新生成swagger描述文件

   ```shell
   $ swag init
   ```

8. 在浏览器查看：http://localhost:8000/docs/index.html

完整的代码在[这里](https://github.com/l2m2/xpm).

## Reference

- https://github.com/swaggo/gin-swagger