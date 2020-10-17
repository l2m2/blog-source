---
title: Go创建Windows服务
toc: false
date: 2020-10-17 18:53:00
description: 学习GO过程中的一些笔记
tags:
- Go
---

安装需要使用的库

```bash
$ go get github.com/kardianos/service
```

按照例子写一个简单的服务

定义一个结构体，实现Start, Stop接口。

```go
type program struct{}

func (p *program) Start(s service.Service) error {
	logger.Info("start running")
	go p.run()
	return nil
}
func (p *program) run() {
	// Do work here
}
func (p *program) Stop(s service.Service) error {
	logger.Info("stop running")
	return nil
}

func main() {
	svcFlag := flag.String("service", "", "Control the system service.")
	flag.Parse()

	svcConfig := &service.Config{
		Name:        "GoServiceExampleSimple",
		DisplayName: "Go Service Example",
		Description: "This is an example Go service.",
	}

	prg := &program{}
	s, err := service.New(prg, svcConfig)
	if err != nil {
		log.Fatal(err)
	}
	logger, err = s.Logger(nil)
	if err != nil {
		log.Fatal(err)
	}
	if len(*svcFlag) != 0 {
		err := service.Control(s, *svcFlag)
		if err != nil {
			log.Printf("Valid actions: %q\n", service.ControlAction)
			log.Fatal(err)
		}
		return
	}
	err = s.Run()
	if err != nil {
		logger.Error(err)
	}
}
```

构建

```bash
$ go build -o service-demo.exe main.go
```

安装服务

```bash
$ ./service-demo.exe -service install
```

安装后运行`services.msc`，在服务列表中可以找到我们的服务。

![](/images/golang-windows-service-1.png)

其他命令参数有start, stop, uninstall

```
Valid actions: ["start" "stop" "restart" "install" "uninstall"]
```

完整的代码在[这里](https://github.com/l2m2/l2-learn-go/tree/master/windows-service)。

