---
date: 2015/07/20 12:34:50
title: Go 快速拼接字符串
categories:
- programming
tags:
- golang
- lang
---

对于频繁的字符串操作，可以使用bytes.Buffer接口提供的方法快速拼接 

和一般语言类似，Go也为string重载了"+"操作符，可以使用它进行字符串拼接，但是在C#和Java中对于大量的字符串拼接操作推荐使用StringBuilder这样的类，同样对于Go中的大量字符串拼接也不推荐直接使用"+"操作。

可以使用bytes.Buffer提供的方法进行快速拼接：

```golang
import (
	"bytes"
	"fmt"
)
func concat() string {
	var buffer bytes.Buffer
	
	for i:=0; i<1000; i++ {
		buffer.WriteString("abcd")
	}
	str := buffer.String()
	// do something to str
	...
	return str
}
```

这样可以获得O(n)的时间复杂度。

参考： [stackoverflow.com/](http://stackoverflow.com/questions/1760757/ "http://stackoverflow.com/questions/1760757/")
