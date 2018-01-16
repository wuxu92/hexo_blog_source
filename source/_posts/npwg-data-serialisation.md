---
date: 2015/11/04 12:34:50
title: Go 网络编程-数据序列化
categories:
- programming
tags:
- network
- golang
- go
---
![](/images/golang/gopher-banner-small.jpg)

这一篇主要是关于Go序列化数据。数据序列化在网络编程中非常重要，在网络传输数据时，如果数据是对象，数组等等，需要先把对象序列化为才能传递，同时接收方需要进行反序列化才能使用这些数据。一个比较常见的是json数据。 序列化和反序列化操作又成为 marshalling 和 unmarshalling 。

## 序列化方法 ##
序列化其实简单地说就是一种把变量似乎用字节流表示出来的规范，我们可以定义自己的规范，只要接收方能使用相应的方法反序列化出来原来的对象就可以了。但是我们要考虑序列化方法的通用性，希望对任意的数据可以使用同样的序列化和反序列化方法实现。
早期很著名的系列化方法是 XDR(external data representation),它最早用于Sun的RPC中。Go没有对不透数据的marshalling 和 unmarshalling 有明确的支持。RPC包中没有使用XDR，而是使用`gob`。
要实现对数据统一的序列化方法，需要实现一套 **自描述数据**的规范。像xml就是一类自描述的数据，能通过节点属性对数据进行描述。xml还有很强的扩展性。这也是为什么很多项目中会使用xml作为数据传输的格式。

<!-- more -->
## ASN ##
ASN 是 abstract Syntax notation 的缩写，也就是抽象格式标记。这是1984年为电信行业设计的。ASN.1非常复杂，在Go的 'ans.1' 包中实现了一个它的一个子集。它可以从复杂的数据结构中建立自描述的序列化数据。在现在的网络系统中用的最多的是作为 X.509 认证的编码，我们知道 X.509 在认证（SSL）中用途非常广。
下面两个方法可以用来marshal和unmarshal数据。

```
func Marshal(val interface{}) ([]byte, os.Error)
func Unmarshal(b []byte, val interface{}) (rest []byte, err error)
```
第一个方法将数据(val)整理为一个序列化的字节数组；第二个方法就是反序列化了。注意由于Go是静态类型的语言，与PHP和JS中的反序列化不太一样，Unmarshal的第一个参数的类型是interface{}，在实际反序列化时需要做类型检查，因为要把反序列化得到一个序列化时输入的对象相同的类型的变量中。其一个示例如下：

```
import (
	"fmt"
	"encoding/asn1"
)
func main() {
	serialed, e := asn1.Marshal(12)
	CheckErr(e)
	fmt.Println("serialed data", serialed)   // [2 1 12]
	var n int
	_, err := asn1.Unmarshal(serialed, &n)
	CheckErr(err)
	fmt.Println("After unmarshal", n)  // 12
}
```
要注意对于基础类型的Unmarshal需要取地址符，否则会panic。
要知道序列化支持的数据类型跟Go对其的实现程度有关。asn定义了很多的字符集和类型，但是Go只支持 PrintableString 和 IA5String(ASCII)。注意它不能实现unicode字符的序列化。

asn1可以序列化结构体(struct)，但是**要求结构体的所有字段都是可导出的**（Exportable）即首字母都是大写的，如果有非导出的小写字母field则会unmarshal的时候panic。

```
type Person struct {
	Name string
	Age int
	gender int  // cause unmarshal panic
}

func main() {
	me := Person{"wuxu", 23, 1}
	serialMe, err := asn1.Marshal(me)
	CheckErr(err)
	
	var anotherMe = new(Person)
	_, err = asn1.Unmarshal(serialMe, anotherMe) // panic: reflect: reflect.Value.SetInt using value obtained using unexported field

	CheckErr(err)
	
	fmt.Println("unmarshal me", anotherMe)
}
```
还有要注意的是序列化的时候只考虑属性的类型，而不考虑属性的变量名。所以对于只是属性名不同的结构体的序列化字节数组可以互相反序列化操作的。

## JSON ##
JSON 应该都比较熟悉了，它是一种很方便的JavaScript对象标记，本是用来在js系统之间传递数据的轻量级方法，它使用基于文本的格式且具有足够的一般性，被许多的编程语言用作数据序列化方法。
关于json标记的具体语法可以看[官方的文档](http://www.json.org/)，有很多非常好理解的图示说明。
下面是一个序列化json的示例

```
import (
	"fmt"
	"encoding/json"
)
type Person struct {
	Name Name
	Email []Email
}
type Name struct {
	Xing string
	Ming string
}
type Email struct {
	Type string
	Addr string
}
func main() {
	me := Person{
		Name: Name{"wu", "xu"},
		Email: []Email{
			Email{"work", "wuxu92@outlook.com"},
			Email{"home", "wuxu92@163.com"},
		},
	}
	serialData, err := json.Marshal(me)
	fmt.Println("marshaled", string(serialData))
	var newMe Person
	err = json.Unmarshal(serialData, &newMe)
	CheckErr(err)
	fmt.Println("umarshal data", newMe)
	saveJson("me.json", me)
	loadMe := new(Person)
	loadJson("me.json", loadMe)
	fmt.Printf("load from file %v %T\n", loadMe, loadMe)
}
func saveJson(file string, v interface{}) {
	oFile, err := os.Create(file)
	CheckErr(err)
	defer oFile.Close()
	encoder := json.NewEncoder(oFile)
	err = encoder.Encode(v)
	CheckErr(err)
}
func loadJson(file string, v interface{} ) {
	iFile, err := os.Open(file)
	CheckErr(err)
	defer iFile.Close()
	decoder := json.NewDecoder(iFile)
	err = decoder.Decode(v)
	CheckErr(err)
}
```
这段代码有使用json包的marshal/unmarshal方法以及Encoder/Decoder的使用。其中Decoder/Encoder的使用比较有意思的。NewEncoder使用一个 `io.Writer` 作为参数，其Encode方法直接将序列化的结果保存到这个writer，我们使用一个文件作为参数就能直接将序列化结果保存到文件了，非常方便，而Decoder的使用也同样方便。

## gob ##
gob应该是Go的特有的技术，因为在Go之前没有听说过。

> Gob is a serialisation technique specific to Go. It is designed to encode Go data types specifically and does not at present have support for or by any other languages. It supports all Go data types except for channels, functions and interfaces. 

果然gob是go特有的技术，可以对go中很多特有的类型提供序列化支持，比如 channel， interface等等，目前只此一家，其他语言都不支持它。
gob 会把类型信息保存在序列化之后的数据中，这使得它比的可扩展性非常好。同时它相比XML的类型信息要高效很多。由于序列化保存了类型信息，所以下面两个类型的序列化可以互相转换：

```
type T struct {
	a int
	b int
]

type T2 struct {
	b int
	a int
}
```
也就是属性的顺序可以随意，甚至多一些少一些字段的类型都是可以的，鲁棒性非常强。对于指针类型也是可以的，上面的T的序列化数据甚至能反序列化到下面的类型：

```
type T3 struct {
	*a int
	**b int
}
```
gob的使用和json的encoder/decoder模式很像，首先创建一个Encode对象，它使用一个 `io.Writer` 作为参数，encoder的 `Encode` 方法将序列化的值写到数据流中。Decoder使用也很相似，下面是一个示例：

```
import (
	"fmt"
	"os"
	"encoding/gob"
)
type Person struct {
	Name Name
	Email []Email
}
type Name struct {
	Xing string
	Ming string
}
type Email struct {
	Type string
	Addr string
}

func main() {
	me := Person{
		Name: Name{"wu", "xu"},
		Email: []Email{
			Email{"work", "wuxu92@outlook.com"},
			Email{"home", "wuxu92@163.com"},
		},
	}
	saveGob("me.gob", me)
	var loadMe = new(Person)
	loadGob("me.gob", loadMe)
	fmt.Printf("load %v \ntype:%T\n", loadMe, loadMe)
}
func saveGob(file string, v interface{}) {
	iFile, err := os.Create(file)
	CheckErr(err)
	defer iFile.Close()
	encoder := gob.NewEncoder(iFile)
	err = encoder.Encode(v)
	CheckErr(err)
}
func loadGob(file string, v interface{}) {
	oFile, err := os.Open(file)
	CheckErr(err)
	defer oFile.Close()
	decoder := gob.NewDecoder(oFile)
	err = decoder.Decode(v)
	CheckErr(err)
}
```
通过和上面的json的比较，gob序列化出来的并不是一个基于可打印字符串的，而是一个二进制流，使用文本编辑器打开可以看到email等信息，但是有很多是乱码，序列化结果gob比json的文件要小很多。

因为可以使用 `io.Write/io.Read` 创建Encoder和Decoder，而之前讲的网络服务部分的 `Conn` 就同时实现了Write和Read，所以可以直接使用Conn变量传输序列化数据，非常方便。

## 二进制数据的序列化 ##
把二进制数据序列化的方法最常用的像base64这样的方法，把二进制文件转换为ascii码，go的 `encoding/base64` 提供了相应的支持。其中最重要的两个方法如下：

```
func NewEncoder(enc *Encoding, w io.writer) io.WriteCloser
func NewDecoder(enc *Encoding, r io.Reader) io.Reader
```
其中的第一个参数常用 `base64.StdEncoding` 既可。base64包定义了一些 Encoding 类型的变量，如下：

```
var RawStdEncoding = StdEncoding.WithPadding(NoPadding)
var RawURLEncoding = URLEncoding.WithPadding(NoPadding)
var StdEncoding = NewEncoding(encodeStd)  // StdEncoding is the standard base64 encoding, as defined in RFC 4648.
var URLEncoding = NewEncoding(encodeURL)
```
base64 encoding的基本用法示例如下：

```
bitData := []byte{1,2,3,4,5,6,7,8}
bb := &bytes.Buffer{}  // buffer是一个Writer和Reader
encoder := base64.NewEncoder(base64.StdEncoding, bb)
encoder.Write(bitData) // 把二进制数据写入 Writer bb
rncoder.Close()

dbuf := make([byte, 12)
decoder := base64.NewDecoder(baes64.StdEncoding, bb)
decoder.Read(dbuf)
for _, ch := range dbuf {
	fmt.Print(ch)
}
```

本节结束，对于数据序列方法应该有了基本的印象，在实际项目中应该知道怎么序列化数据了。
