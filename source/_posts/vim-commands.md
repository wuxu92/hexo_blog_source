---
date: 2015/07/18 12:34:50
title: vim 常用的命令模式
categories:
- linux
tags:
- vim
- tool
---

vim中常用的命令，日常更新 


```
!ls  使用感叹号开头调用系统命令
~	 切换当前光标所在字符的大小写
```

## buffer相关 ##

```
:ls
:bn :bnext
:bp :bprev
:bd :bdelete
:b1 :buffer1
:bufdo cmd 在所有缓冲区执行命令
```

## 参数列表 ##

```
:args 列出参数列表或者设置参数列表
:args **/*.postfix
:write :w
:edit! :e!放弃修改重新载入文件
:qall :qa
:wall :wa
```

## 多窗口 ##

```
:sp [file] 水平分割
:vsp [filw] 垂直分割
:clo <C-w>c 关闭当前窗口
:no[ly] <C-v>o 关闭当前之外的所有窗口
```

**切换窗口**

```
<C-w><C-w> 在串口中切换，连续按两次Ctrl+w
<C-w>h|j|k|l 切换到左/下/上/右的窗口
```

**改变窗口大小**

```
<C-w>=
<C-w>_  最大化当前窗口的高度
<C-w>|  最大化当前窗口的宽度
[n]<C-w>_   设置
[n]<C-w>|
```

## 标签 ##

```
:tabedit :tabe {filename} 如果filename为空则打开一个新的标签页
:tabclose :tabc 
:tabonly  :tabo  关闭当前标签之外的所有标签
```

**标签切换**

```
:tabnext  :tabn {N}  Ngt
:tabn gt
:tabp gT
:tabmove [N] 移动标签页
```
## 打开及保存文件 ##

```
:edit full/path/name
:edit %:h<tab>filename 	使用缓冲区文件的完整路径打开文件, :h去掉当前文件名，tab键补全
:edit path 打开文件管理窗口
:find filename 在path下查找文件
: set path+=/path/to/workspace  将工作区添加到vim的path中用于寻找
:e 是 :edit的缩写
:Explore :E 打开当前的目录管理器
:Sexplore  explore的切分
:Vexplore
```
**文件操作**

```
:!mkdir -p %:h    前缓冲区创建目录，然后在:w，解决某些不能保存的问题
:w !sudo tee % > /dev/null 由于保存用非sudo打开的无权限文件的保存
```


