---
title: 用Go将jekyll的文章转换成hexo文章风格
date: 2016-01-07 14:02:07
categories:
- programming
tags:
- hexo
- blog
- fe
---
博客之前使用jekyll引擎部署在github pages, 由于几个原因一直想迁移到hexo平台：

- 原来的模板被我修改的比较严重，文章页面在大屏幕上的显示效果无法居中了
- Jekyll在Windows下编译比较麻烦，每次都要 push 到github才能预览效果
- 当时使用的Jekyll模板对代码的渲染效果很糟糕，尤其是Vue.js的大括号需要各种转义才好显示

但是迁移也挺麻烦的，主要是Jekyll使用的模板和hexo的markdown解析并不完全兼容，尤其是 Front-matter，也就是文章元数据的格式不太一样，还有就是文档的命名风格也不一样，Jekyll使用日期加标题的方式，hexo一般直接使用标题。原来已经有大概110篇文章，一篇一篇修改太麻烦了，所以用Go写了一个简单的工具去转换。

代码如下：
<!-- more -->

``` golang
package main

import (
  "fmt"
  "io/ioutil"
  "os"
  "bufio"
  "strings"
)

func main() {
  dirString := "."
  if len(os.Args) == 1 {
    fmt.Errorf("you need give the path to run, default to current directory") 
  } else {  
    dirString = os.Args[1]
  }
  fmt.Println("handle directory ", dirString)
  files, err := ioutil.ReadDir(dirString)
  checkErr(err)
  
  // craete new dir
  err = os.Mkdir("newPosts", 0644)
  if os.IsExist(err) == false {
    checkErr(err)
  } else {
    fmt.Printf("skip mkdir\n")
  }
  // line := make([]byte, 100)
  var line string
  var header []string
  var newPost string
  var linenum int
  for _, f := range files {
    desc := ""
    filename  := f.Name()
    newFileName := filename
    fmt.Println("file:", filename)
    if strings.HasSuffix(filename, ".md") {
      fmt.Printf("handle markdown file: %s\n", filename)
      // nedd get date from filename
      dateArr := strings.Split(filename, "-")
      dateStr := ""
      if len(dateArr) >= 4 {
        dateStr = "date: " + dateArr[0] + "/" + dateArr[1] + "/" + 
          dateArr[2] + " 12:34:50\n"
        newFileName = string(filename[11:])
      }
      f, err := os.Open(filename)
      checkErr(err)
      defer f.Close()
      
      scanner := bufio.NewScanner(f)
      scanner.Scan()
      linenum = 1
      newPost = "---\n"
      if strings.Compare(dateStr, "") != 0 {
        newPost += dateStr
      }
      // if first line is ---
      if strings.HasPrefix(scanner.Text(), "---") {
        for {
          scanner.Scan()
          linenum += 1
          line = strings.Trim(scanner.Text(), " ")
          if strings.HasPrefix(line, "--") {
            break
          }
          header = strings.Split(line, ":")
          if len(header) < 2 {
            continue
          }
          header[1] = strings.TrimLeft(header[1], " ")
          if strings.Compare(header[0], "title") == 0 || 
            strings.Compare(header[1], "layout") == 0 {
              newPost += line + "\n"
          }
          stmp := handleHeader(header[0], header[1])
          if strings.Compare(stmp, "") != 0 {
            newPost += stmp
          } else if strings.Compare(header[0], "description") == 0 {
            desc = header[1]
          }
        } // end of for header
      }
      newPost += "---\n"
      // scan to the end of the file
      descReplced := false
      for scanner.Scan() {
        line = scanner.Text()
        linenum += 1
        if descReplced == false {    
          oldDesc := "{{ page.description }}"
          oldDesc2 := "{{page.description}}"
          if strings.Index(line, oldDesc) != -1 {
            fmt.Println("replace desc 1")
            line = strings.Replace(line, oldDesc, desc, 1)
            descReplced = true
          } else if strings.Index(line, oldDesc2) != -1 {
            fmt.Println("replace desc 2")
            line = strings.Replace(line, oldDesc2, desc, 1)
            descReplced = true
          }
        }
        
        // raw {{ }} lines
        braceIdx := strings.Index(line, "{{")
        if braceIdx != -1 {
          fmt.Println("handle braces at ", linenum)
          line = strings.Replace(line, "{{", "{% raw %} {{", -1)
          line = strings.Replace(line, "}}", "}} {% endraw %}"， -1)
        }
        newPost += line + "\n"
      } // end of post scan
      // write to new file
      newfile, err := os.Create( "newPosts/" + newFileName)
      checkErr(err)
      defer newfile.Close()
      newfile.Write([]byte(newPost))
      
    } // end of if .md
  }
}

func checkErr(err error) {
  if err != nil {
    panic(err)
  }
}

func handleHeader(k, v string) string {
  var cats []string
  var res string
  v = strings.Trim(v, " ")
  if strings.Compare(k, "category") == 0 {
    res = "categories:\n"
    cats = strings.Split(v, " ")
    for _, cat := range cats {
      res += ("- " + cat + "\n")
    }
    return res
  }
  if strings.Compare(k, "tags") == 0 {
    res = "tags:\n"
    cats =strings.Split(v, " ")
    for _, cat := range cats {
      if strings.Compare(cat, " ") != 0 &&
        strings.Compare(cat, "") != 0 {
        res += ("- " + cat + "\n")
      }
    }
    return res
  }
  return ""
}
```
主要作用是将原来的front-matter里面的category改为hexo的categories格式，将tags的格式也修改了，将原来的description替换到文章去，删掉front-matter中的description，从Jekyll文档名中抽取日期输出新文章的date属性。

这里只替换了文档中的 page.description 模板，其他的双大括号行都是用 raw 包围，这可能导致一些错误，因为hexo渲染的代码的双大括号不需要 raw 包围就能自动转义，所以这部分可能需要自己在手动修改一下。

编译成可执行文件后，命令行执行，可以指定一个目录参数，默认在当前目录搜索 .md 后缀的文档，逐个处理后，新建 ./newPosts目录，并在该目录下生成新的文档。处理完后将生成的文档拷贝到 `hexo_path/source/_post/` 下即可。

由于原来的文章格式比较复杂，上面的处理不能保证完全能在hexo下编译渲染成功，还需要子自己排查一下，还是有点麻烦的，这里花了大概2个小时。

如果hexo渲染一直不成功一般式某些文章有双大括号内容，或者是代码的markdown格式不统一或者有错误，需要慢慢排查，我直接删掉了几篇很老的文章终于渲染成功了。

**这个转换还有一个问题是之前的行内代码有部分使用三个星号的，在新的渲染下回出错，应该将这个功能加上，不过这个代码没有机会用了，应该不会再加了**

完