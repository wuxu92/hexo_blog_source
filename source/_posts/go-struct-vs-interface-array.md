---
title: Go使用interface切片类型做参数时的坑
date: 2016-01-16 22:23:00
categories:
- programming
- golang
tags:
- golang
- go
- programming
---

今天在使用mgo（go的mongodb驱动）时遇到了一个奇怪的坑。我想一次性插入一组数据到一个collection，看insert方法的定义如下：

```go
func (c *Collection) Insert(docs ...interface{}) error {
	_, err := c.writeOp(&insertOp{c.FullName, docs, 0}, true)
	return err
}
```
使用一个可变参数，是 interface{} 类型的，应该传一个对象数组就可以了，之前按照mgo的example使用是这样的：
<!-- more -->
```go
err = c.Insert(&Site{"Baidu", "www.baidu.com", 1},
    &Site{"v2ex", "v2ex.com", 22})
```
那就构建一个数：

```go
func GetInitSites() []Site {
  sites := []Site{
    {...},
    {...},
    {...},
  }
  return sites
}
```
用这个数组做插入：

```go
sites := models.GetInitSites()
c := session.DB("snp").C("site")
err = c.Insert(sites...)
if err != nil {
  panic(err)
}
```
（上面的两段代码在不同的包）结果报下面的错误：

```
 cannot use sites (type []models.Site) as type []interface {} in argument to c.Insert
```
这个错误好生奇怪，interface按理说作为所有对象都实现了的接口，那么应该可以使用任何类型数据做参数才对？

像最上面那种方式，分别传入两个Site对象可以，为什么使用Site的切片就不行呢，这和go中对 struct 和 interface 的处理实现有关， 这有点违背直觉，尤其是习惯了Java那种面向对象方式之后。

对于这个错误就是需要显示地将 Site 对象切片转换为 []interface{}，像下面的代码：

```go
sites := models.GetInitSites()
// 将 Site slice 转换为 []interface{}
siteArr := make([]interface{}, len(sites))
for idx, _ := range sites {
  siteArr[idx] = sites[idx]
}
fmt.Println("to insert site count: ", len(sites))
err = c.Insert(siteArr...)
if err != nil {
  panic(err)
}
```
这样做有些奇怪，可能有更好的实现，也可能完全是我想歪了，但是至少现在能运行了，也是一个 work around 吧。

具体细节可以查看参考的第二篇,golang-nuts 邮件组的回复

参考：

- [http://grokbase.com/t/gg/mgo-users/13cvbam2ap/question-on-batch-inserts-basic](http://grokbase.com/t/gg/mgo-users/13cvbam2ap/question-on-batch-inserts-basic)
- [https://groups.google.com/forum/#!topic/golang-nuts/Il-tO1xtAyE/discussion](https://groups.google.com/forum/#!topic/golang-nuts/Il-tO1xtAyE/discussion)