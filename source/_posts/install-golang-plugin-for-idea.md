---
title: IntelliJ IDEA 支持Golang项目
date: 2016-01-24 13:11:25
categories:
- tools
tags:
- tools
- intellij
- golang
---

之前一直使用 LiteIDE 编写Go的程序，在单个文件或文件比较少的时候liteide还是挺好用的，但是文件多了的时候自动提示总是不能即使更新，新添加的方法总是没有提示，感觉是自动提示的索引更新太慢的原因。

前几天把IDEA的golang插件装了一下，发现用IDEA写go的项目舒服多了，intellij的IDE不愧是宇宙最好的IDE之一。免费的社区版（IDEA 15 CE）也支持Golang插件，不过CE没有Nodejs支持，后来装 Vue.js 的插件时遇到了问题，所以如果对盗版没有愧疚的话装 IDEA的ultimate版本也可以。

## 安装语言支持插件
先安装Golang语言支持的插件，插件地址：[https://github.com/go-lang-plugin-org/go-lang-idea-plugin](https://github.com/go-lang-plugin-org/go-lang-idea-plugin)，在默认的插件库中没有，需要手动添加 `File->Settings->Plugins->Browser repositories->Manage repositories` 点右侧的加号，添加源：

Paste the URL for the version you need:

- alpha: [https://plugins.jetbrains.com/plugins/alpha/5047](https://plugins.jetbrains.com/plugins/alpha/5047)
- nightly: [https://plugins.jetbrains.com/plugins/nightly/5047](https://plugins.jetbrains.com/plugins/nightly/5047)

然后到安装插件那搜索 go 就可以了。

## 配置SDK
在IDEA中添加GO SDK： `File->Project Structure->SDKs` 点击添加，选择Go SDK，如果系统配置好了Go的环境变量的话会自动定位到Go的目录比如我的 `C:\Go`,添加好SDK后修改项目的SDK。

在 `File->Project Structure->Project` 现在Project SDK下拉框应该可以选择Go的SDK了。

这样就可以了，之后写Go的项目就非常方便了