---
title: "使用官方库在golang中表示json的三种方法"
date: 2020-05-23T07:53:54+08:00
---

以下方法均只使用了`encoding/json`这个库，但事实上业界还有很多很优秀的`JSON`解析库，也对应着有不同表示`JSON`的方法。本文只描述使用官方库时可用的3种方式。

## 解析为结构体

第一步当然是设计`JSON`对应的结构体。使用类似[json-to-go](https://mholt.github.io/json-to-go/)这样的工具可以直接用`JSON`文件生成对应的结构体。

接着用`json.Unmarshal`解析到对应的结构体即可。

这种方式当然只适合于`JSON`格式比较固定的情形，如果`JSON`中的字段名会变化，结构会变化，这种方法就不能正确地表达`JSON`的内容了。但是，这个特性在某些场景也挺有用，比如处理`POST`请求时，程序可以合法接受的`JSON`格式一般是固定的。把用户提交的`JSON`内容解析到一个`struct`上，这样可以自动过滤掉用户提交的`JSON`文件中不需要的字段。

## 解析为`map[string]interface{}`

这种方式非常**通用**，任何`JSON`结构都可以解析为`map[string]interface{}`。给定`JSON`的文本后，`encoding/json`这个库实际上还是会将`JSON`的内容按类型正确解析，该是`int`的就是`int`，但是，都存储在`interface{}`类型的变量中。这样，一个变量可以容纳`JSON`的所有内容，但每次取用其中的一部分内容时，都要首先将`interface{}`转成你需要的变量类型。事实上，虽然可以表示任何`JSON`，但要取用`JSON`的部分内容，还是要首先知道`JSON`的这部分的类型。举个例子：

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

// 预定义的JSON文件
var jsontext = []byte(`
{
	"code": 200,
	"info": [{
		"host": "wrong.wang",
		"spend": 1.23,
		"author": {
			"name": "Wrong Wang"
		}
	}]
}
`)

func main() {
	var m map[string]interface{}
	// 解析
	if err := json.Unmarshal(jsontext, &m); err != nil {
		log.Fatal(err)
	}
	// 内部已经知道了对应的类型，只是把数据存在interface{}容器中
	fmt.Println(m)

	code, ok := m["code"]
	if !ok {
		log.Fatal("'code' field not found")
	}
	// 数字一定被解析为float64,尽管这里200可以被解析为int类型
	switch code.(type) {
	case int64:
		fmt.Println("unmarshal  `code` into int64", code)
	case float64:
		fmt.Println("unmarshal  `code` into float64", code)
	case int:
		fmt.Println("unmarshal  `code` into int", code)
	case string:
		fmt.Println("unmarshal  `code` into string", code)
	default:
		fmt.Println("unmarshal what type??")
	}

	// 为了代码简洁，假设这些类型都已知，
	// 实际使用时，每一步类型转换，每一步map取值，都需要检查是否报错。
	infoi := m["info"]
	info := infoi.([]interface{})
	infom := info[0].(map[string]interface{})
	fmt.Println(infom["host"])
}
```

上面的代码可以在[The Go Playground](https://play.golang.org/p/N_gwctYV17m)中查看。

上面这份代码运行结果如下：

```plaintext
map[code:200 info:[map[author:map[name:Wrong Wang] host:wrong.wang spend:1.23]]]
unmarshal  `code` into float64 200
wrong.wang
```

总的来说，解析为`map[string]interface{}`类型后，读取任何一个字段都需要检查，都需要类型转换。`JSON`中类型和Go类型对应关系如下：

| `JSON`类型 |      `Golang`类型      |
| :--------: | :--------------------: |
|  boolean   |          bool          |
|  numbers   |        float64         |
|   string   |         string         |
|    nil     |          null          |
|   array    |     []interface{}      |
|    map     | map[string]interface{} |

## 混合以上两种表达

上面两种表示法实际上已经根据`JSON`类型转换到了Go中的类型，即已经解析了。还有一种暂不解析的`JSON`的表达方式：解析为`map[string]json.RawMessage`。[`json.RawMessage`](https://golang.google.cn/pkg/encoding/json/#RawMessage) 是原始未解析的`JSON`值，它实现了`json.Marshaler`和`json.Unmarshaler`的方法，可以用来延迟解析。即真正要使用对应字段的内容时再继续解析`JSON`值。官方库文档中的这个例子就显示的很清楚了！

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

func main() {
	type Color struct {
		Space string
		Point json.RawMessage // delay parsing until we know the color space
	}
	type RGB struct {
		R uint8
		G uint8
		B uint8
	}
	type YCbCr struct {
		Y  uint8
		Cb int8
		Cr int8
	}

	var j = []byte(`[
	{"Space": "YCbCr", "Point": {"Y": 255, "Cb": 0, "Cr": -10}},
	{"Space": "RGB",   "Point": {"R": 98, "G": 218, "B": 255}}
]`)
	var colors []Color
	err := json.Unmarshal(j, &colors)
	if err != nil {
		log.Fatalln("error:", err)
	}

	for _, c := range colors {
		var dst interface{}
		switch c.Space {
		case "RGB":
			dst = new(RGB)
		case "YCbCr":
			dst = new(YCbCr)
		}
		err := json.Unmarshal(c.Point, dst)
		if err != nil {
			log.Fatalln("error:", err)
		}
		fmt.Println(c.Space, dst)
	}
}
```

这种方式适合从一个大`JSON`文件中提取部分结构，比较灵活方便。同时因为是用时才解析，相比于前两种方法也有一定的性能优势。
