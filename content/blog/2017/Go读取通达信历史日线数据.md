---
date: 2017-07-26T10:32:00Z
description: "使用Go从通达信读取A股历史行情信息" 
slug: "Go-du-qu-tong-da-xin-li-shi-ri-xian-shu-ju"
tags:
- Golang
- 通达信
- 行情
title: Go读取通达信历史日线数据
topics:
- 编程语言与开发
- 金融
---

突然间想使用Go从通达信读取A股历史行情信息，其实也蛮简单的。从通达信获取数据难点在于分析数据结构，而读取则各类语言分分钟搞定。

### 准备工作

1. 下载安装通达信,[通达信官网][1]
2. 下载历史行情数据

下载操作路径：系统->盘后数据下载

![通达信下载盘后数据](https://static.yushuangqi.com/blog/2017/201772601.png)

下载后数据按股票市场分别存放：

+ 上海交易所：`{通达信安装目录}\vipdoc\sh\lday\*.day`
+ 深圳交易所：`{通达信安装目录}\vipdoc\sz\lday\*.day`

### 通达信历史日线数据文件格式

每只股票一个day文件，如：sh000001.day。文件中每一天数据总共32字节。其中每32字节数据格式如下：

|数据含义|数据类型|数据长度|举例|单位|
|----|----|----|----|----|
|日期|Integer|4|20170703|
|开盘价|Integer|4|2476|当前值/100,元|
|最高价|Integer|4|2520|当前值 /100,元|
|最低价|Integer|4|2436|当前值 / 100,元|
|收盘价|Integer|4|2457|当前值 / 100,元|
|成交金额|single|4|1317335898|元|
|成交量|Integer|4|45293799|股|
|保留|Integer |4|

注意，因为价格均是两位小数，故文件中的价格放大100倍，以便按数字存储。


### Go读取日线数据文件

文件每32字节存储一天数据，在Go中只需读取指定长度的字节，再转换为int即可。

第一步：读取文件
以万科股票为例，打开该day文件，获得 reader。
```go
f, err := os.Open(`D:\new_tdx\vipdoc\sz\lday\sz000002.day`)
if err != nil {
	log.Fatal(err)
}
defer f.Close()
```
第二步：获取32字节数据

从文件流中填充32个字节的数据到oneDay中，如果无错误则可以按照数据格式读取单独一天的日行情数据。

注意取价格时需再除以100，已显示正确的金额。
```go
oneDay := make([]byte, 32)
_, err = f.Read(oneDay)
if err != nil {
	log.Fatal(err)
}
fmt.Println("日期:\t", binary.LittleEndian.Uint32(oneDay[0:4]))
fmt.Println("开盘价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[4:8]))/100)
fmt.Println("最高价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[8:12]))/100)
fmt.Println("最低价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[12:16]))/100)
fmt.Println("收盘价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[16:20]))/100)
fmt.Println("成交金额(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[20:24]))/100)
fmt.Println("成交量:\t", binary.LittleEndian.Uint32(oneDay[24:28]))
fmt.Println("Other:\t", binary.LittleEndian.Uint32(oneDay[28:32]))

//output:
/*
日期:    20170703
开盘价(元):      24.76
最高价(元):      25.2
最低价(元):      24.36
收盘价(元):      24.57
成交金额(元):    1.317335898e+07
成交量:  45293799
Other:   65536
*/
``` 

第三步：遍历获取所有日线数据
```go
oneDay := make([]byte, 32)
for {
	l, err := f.Read(oneDay)
	if err == io.EOF {
		break
	} else if err != nil {
		log.Fatal(err)
	} else if l != 32 {
		log.Fatal("数据不完整，终止")
	}
    //Date
	fmt.Printf("%d,", binary.LittleEndian.Uint32(oneDay[0:4]))
	//other
}
```
历史数据可在其他网站上查看，下图来自搜狐数据

 ![通达信下载盘后数据](https://static.yushuangqi.com/blog/2017/20170726115452.png)

完整代码：
```go
package main

import (
	"bytes"
	"encoding/binary"
	"fmt"
	"io"
	"log"
	"os" 
)

func main() {
	f, err := os.Open(`D:\new_tdx\vipdoc\sz\lday\sz000002.day`)
	if err != nil {
		log.Fatal(err)
	}
	defer f.Close()

	oneDay := make([]byte, 32)
	fmt.Println("日期\t\t开盘价(元)\t最高价(元)\t最低价(元)\t收盘价(元)\t成交金额(元)\t成交量\tOther")
	for {
		l, err := f.Read(oneDay)
		if err == io.EOF {
			break
		} else if err != nil {
			log.Fatal(err)
		} else if l != 32 {
			log.Fatal("数据不完整，终止")
		}
		fmt.Printf("%d\t%f\t%f\t%f\t%f\t%f\t%d\t%d\t\n",
			binary.LittleEndian.Uint32(oneDay[0:4]),
			(float64)(binary.LittleEndian.Uint32(oneDay[4:8]))/100,
			(float64)(binary.LittleEndian.Uint32(oneDay[8:12]))/100,
			(float64)(binary.LittleEndian.Uint32(oneDay[12:16]))/100,
			(float64)(binary.LittleEndian.Uint32(oneDay[16:20]))/100,
			(float64)(binary.LittleEndian.Uint32(oneDay[20:24]))/100,
			binary.LittleEndian.Uint32(oneDay[24:28]),
			binary.LittleEndian.Uint32(oneDay[28:32]),
		)
	}
}
```

### 进价

有没有更好的办法来进行数据转换？如下格式处理，过于麻烦，幸好该格式数据不多。但容易弄错，或者调整麻烦。
```go
fmt.Println("日期:\t", binary.LittleEndian.Uint32(oneDay[0:4]))
fmt.Println("开盘价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[4:8]))/100)
fmt.Println("最高价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[8:12]))/100)
fmt.Println("最低价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[12:16]))/100)
fmt.Println("收盘价(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[16:20]))/100)
fmt.Println("成交金额(元):\t", (float64)(binary.LittleEndian.Uint32(oneDay[20:24]))/100)
fmt.Println("成交量:\t", binary.LittleEndian.Uint32(oneDay[24:28]))
fmt.Println("Other:\t", binary.LittleEndian.Uint32(oneDay[28:32])) `
```
可以利用Go的`encoding/binary`包从reader中读取二元数据到指针对象中:
```go
// Read reads structured binary data from r into data.
// Data must be a pointer to a fixed-size value or a slice
// of fixed-size values.
// Bytes read from r are decoded using the specified byte order
// and written to successive fields of the data.
// When decoding boolean values, a zero byte is decoded as false, and
// any other non-zero byte is decoded as true.
// When reading into structs, the field data for fields with
// blank (_) field names is skipped; i.e., blank field names
// may be used for padding.
// When reading into a struct, all non-blank fields must be exported.
//
// The error is EOF only if no bytes were read.
// If an EOF happens after reading some but not all the bytes,
// Read returns ErrUnexpectedEOF.
func binary.Read(r io.Reader, order ByteOrder, data interface{}) error{}
```

这样便可以方便的将byte读取到对象中，下面我们定义data结构：
```go
type DayData struct {
	Date                   int32
	Open, High, Low, Close int32
	Amount, Qty            int32
	Other                  int32
}

var d DataData
buf := bytes.NewBuffer(oneDay)
binary.Read(buf, binary.LittleEndian, &d)
fmt.Printf("%v\n", d)
```
注意，在binary读取buf到对象时，是依次遍历对象的内存结构赋值的，因为文件中各数据均是4位长度。故在定义字段类型时，选择int32,占4个字节。字段定义顺序便是内存结构顺序，同文件数据定义顺序保持一致。

但因为数据中均存放的是int32，而收盘价等需要进行转换，故对外时提供另一个结构体，已方便正常访问各数据。
在解析时，进行一次解析即可。
```go
//DayQuotaion 日线行情
type DayQuotaion struct {
	Marker    string  //股票市场
	StockCode string  //股票代码
	Date      int     //日期
	Open      float32 //开盘价
	High      float32 //最高价
	Low       float32 //最低价
	Close     float32 //收盘价
	Amount    float32 //总成交金额
	Qty       float32 //  总成交量
} 
```

### 完整代码

```go
package main

import (
	"bytes"
	"encoding/binary"
	"errors"
	"fmt"
	"io"
	"log"
	"os"
	"path/filepath"
	"strings"
)

const historyDailyQuotationPath = `D:\new_tdx\vipdoc`

type dayData struct {
	Date                   int32
	Open, High, Low, Close int32
	Amount, Qty            int32
	Other                  int32
}

// To 转换为日线行情数据
func (d *dayData) To() *DayQuotation {
	return &DayQuotation{
		Date:   d.Date,
		Open:   float32(d.Open) / 100,
		High:   float32(d.High) / 100,
		Low:    float32(d.Low) / 100,
		Close:  float32(d.Close) / 100,
		Amount: float32(d.Amount),
		Qty:    d.Qty,
	}
}

// DayQuotation 日线行情
type DayQuotation struct {
	Date   int32   //日期
	Open   float32 //开盘价
	High   float32 //最高价
	Low    float32 //最低价
	Close  float32 //收盘价
	Amount float32 //总成交金额
	Qty    int32   //  总成交量
}

//GetStockQuoation 获取股票历史行情
func GetStockQuoation(marker, stockCode string) ([]*DayQuotation, error) {
	marker = strings.ToLower(strings.TrimSpace(marker))
	stockCode = strings.ToLower(strings.TrimSpace(stockCode))
	if marker == "" || stockCode == "" {
		return nil, errors.New("marker和stockCode不能为空")
	}
	//文件路径，e.g. D:\new_tdx\vipdoc\sz\lday\sz000002.day
	name := filepath.Join(historyDailyQuotationPath, marker, "lday", fmt.Sprintf("%s%s.day", marker, stockCode))
	f, err := os.Open(name)
	if err != nil {
		return nil, err
	}
	defer f.Close()

	quos := []*DayQuotation{}
	oneDay := make([]byte, 32)
	data := &dayData{}
	for {
		l, err := f.Read(oneDay)
		if err == io.EOF {
			break
		} else if err != nil {
			return quos, err
		} else if l != 32 {
			return quos, errors.New("数据不完整")
		}
		buf := bytes.NewBuffer(oneDay)
		err = binary.Read(buf, binary.LittleEndian, data)
		if err != nil {
			return quos, err
		}
		quos = append(quos, data.To())
	}
	return quos, nil
}

func main() {
	quos, err := GetStockQuoation("sz", "000002")
	if err != nil {
		log.Fatal(err)
	}
	for _, v := range quos {
		fmt.Printf("%v\n", v)
	}

}
```

[1]: http://www.tdx.com.cn/index.html "通达信官网下载地址"
[2]: http://q.stock.sohu.com/cn/000002/lshq.shtml "搜狐证券"