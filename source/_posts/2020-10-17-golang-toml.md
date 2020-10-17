---
title: Go操作TOML
toc: false
date: 2020-10-17 16:47:00
description: 学习GO过程中的一些笔记，没啥可看的
tags:
- Go
- TOML
---

## TOML

[TOML](https://toml.io/) 的全称是 Tom's Obvious, Minimal Language，意为 Tom 的（语义）明显、（配置）最小化的语言。

我们用JSON配置格式与TOML进行类比。

JSON:

```json
{
  "name": "Orange",
  "physical": {
    "color": "orange",
    "shape": "round"
  },
  "site": {
    "google.com": true
  }
}
```

TOML:

```toml
name = "Orange"
physical.color = "orange"
physical.shape = "round"
site."google.com" = true
```

详细的介绍参阅官方文档，此处不再赘述。

## GO操作TOML

我们使用[BurntSushi的toml库](https://github.com/BurntSushi/toml)进行TOML配置文件的操作。

使用go Module创建一个测试module

```bash
$ go mod init github.com/l2m2/go-toml-demo
```

安装TOML操作库

```bash
$ go get github.com/BurntSushi/toml
```

假设我们的配置文件长这样

```toml
Age = 25
Cats = [ "Cauchy", "Plato" ]
Pi = 3.14
Perfection = [ 6, 28, 496, 8128 ]
DOB = 1987-07-05T05:45:00Z
```

我们需要在Go中定义数据结构如下

```go
type Config1 struct {
	Age        int
	Cats       []string
	Pi         float64
	Perfection []int
	DOB        time.Time
}
```

然后这样解析，

```go
func decode1() {
	ex, err := os.Executable()
	if err != nil {
		panic(err)
	}
	exPath := filepath.Dir(ex)
	fmt.Println(exPath)
	var conf Config1
	if _, err := toml.DecodeFile(filepath.Join(exPath, "conf1.toml"), &conf); err != nil {
		panic(err)
	}
	fmt.Printf("%#v", conf)
    // main.Config1{Age:25, Cats:[]string{"Cauchy", "Plato"}, Pi:3.14, Perfection:[]int{6, 28, 496, 8128}, DOB:time.Time{wall:0x0, ext:62688059100, loc:(*time.Location)(nil)}}
}
```

**我们也可以对配置文件中的标签和数据结构的名称进行映射**

例如，

```go
type Config2 struct {
	Age  int
	Cats []string
	Pi   float64
	Perf []int `toml:Perfection` // 打标签
	DOB  time.Time
}
```

我们把数据结构中的Perfection替换成了Perf，那么打印出的值也变成了下面这样

```go
// main.Config2{Age:25, Cats:[]string{"Cauchy", "Plato"}, Pi:3.14, Perf:[]int(nil), DOB:time.Time{wall:0x0, ext:62688059100, loc:(*time.Location)(nil)}}
```

**改变配置，写入文件**

```go
	conf.Age = 34
	fmt.Printf("%#v", conf)
	f, _ := os.Create(confPath)
	defer f.Close()
	if err := toml.NewEncoder(f).Encode(conf); err != nil {
		panic(err)
	}
```

完整代码在[这里](https://github.com/l2m2/l2-learn-go/tree/master/toml)。

## Reference

- https://toml.io/
- https://github.com/BurntSushi/toml
- https://stackoverflow.com/questions/10858787/what-are-the-uses-for-tags-in-go