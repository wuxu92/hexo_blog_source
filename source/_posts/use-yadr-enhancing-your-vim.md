---
date: 2015/07/25 12:34:50
title: yadr中提供的vim快捷操作
categories:
- tools
tags:
- vim
- yadr
- linux
---

github上面的活跃的dotfiles项目[yadr](https

这里主要就是官方readme文档的 [Vim部分](https://github.com/skwp/dotfiles/#vim---whats-included "https://github.com/skwp/dotfiles/#vim---whats-included") 的记录。基本的vim操作就不详细介绍了。

参考： [http://skwp.github.io/dotfiles/](http://skwp.github.io/dotfiles/ "http://skwp.github.io/dotfiles/")
## 有什么？ ##
yadr提供了

## 导航 ##

- `,z` 切换到前一个缓冲文件（buffer），相当于 :bp[revious]
- `,x` 切换到后一个缓冲，相当于命令行的 :bn[ext]
- `Alt+j` 和 `Alt+k` 下跳或者上跳一个函数；好像不起作用
- `Ctrl+o` 跳转到旧的指针位置，这是vim的标准快捷键，**非常有用**
- `Ctrl+i` Ctrl+o的逆操作，也是标准按键

## 搜索与代码导航 ##

- `,f` 跳转到类定义，需要系统安装了ctags，并创建tags file。

> ctags是一个独立的软件，不包含在vim之中，使用yum安装即可，在项目目录下运行 `ctags -R .` 后会创建tags file，这是就可以使用跳转到定义的功能了。

- `,F` 与`,f`相同不过在新建垂直分屏（vsp）显示定义
- `,gf`或者 `ctrl+f` 跳转到光标坐在变量名对应的文件，但是在一个新的分栏中显示，这在java中比较好用
- `gF` 标准快捷键，打开文件
- `K` 搜索光标所在单词，并在quickfix窗口显示结果，这个功能需要安装silver serach, `sudo yum install -y the_silver_searcher `
- `,K` 没太多用，Grep the current word up to next exclamation point (useful for ruby foo! methods)
- `,hl` 开关搜索结果的高亮，相当于 :set noh与其逆操作
- `,gg`或者`,ag`: 搜索，键入这个命令后，会出现一个输入符，在双引号中输入字符后回车会搜索包含这段字符的行
- `,gd` 搜索包含字符的函数定义， grep def,不太用
- `,gcf` grep current file的缩写，查找对本文件的引用
- `//` 清空搜索
- `,,w` `,<Esc>`的alias，EasyMotion，highlights jump-points on the screen and lets you type to get there
- `,mc` 多标签，这个功能和sublime中的比较像，在一个单词上面使用`,mc`会记录这个单词，然后使用`Ctrl-n`或者`Ctrl+p`选择前一个或者后一个相同的单词的，或者使用Ctrl+x跳过一个。 这个比较有用
- `gK` 打开光标所在单词同名的文档？不常用

## 文件导航 ##

- `,t`  文件选择器，不常- 用，键入指令后可以输入字符筛选文件
- `,b` 打开缓存区的一个文件，比较常用
- `Cmd-Shift-M` 跳转到方法，linux下好像没用。
- `,jm` jump to models,可能在rails里面比较有用
- `Alt+Shift+N` nerd tree toogle,好像没什么用
- `Ctrl+\` 在侧边栏文件树显示当前文件，和sublime的sidebar有点像
- `Cmd-Shift-P` 清空ctrlp 缓存，没用过

## 常用编辑快捷键 ##

- `Ctrl+space` 自动补全- ， Tab键
- `,#`; `,"`; `,'`; `,]`; `,)`; `,}`;  这些优点麻烦的样子，待补充
- `,.` 跳转到上一个编辑地点。和`'.`的作用相同。
- `,ci` change inside any set of quotes/brackets/etc。其作用是删除", {}, ()之间的字符并进入插入模式

## 多标签，多窗口，分割模式 ##

- `alt+n` - 快速跳转到指定的tab
- `Ctrl-h,j,k,l` 在分栏窗口中上下左右移动。
- `Q` 关闭当前的window，**非常方便**
- `vv` 与 `Ctrl-w,v`一样，vertical split, 上下分栏
- `ss` 与 `Ctrl-w,s`一样，horizontal split，左右分栏
- `,qo` 打开quickfix， qo => quickfix open, **有用**
- `,qc` 关闭quickfix,  qc => quickfix close， **有用**，yadr配置的vim在保存文件时会做语法检查，如果检查不通过会在quickfix窗口显示错误，并且不会自动消失，这时候也许需要 `,qc`来关闭它。

## 其它常用 ##

- `Ctrl-p` 循环历史剪切板，这个很有用，p是粘贴，Ctrl+p会粘贴之前的剪切板内容
- `,yr` view yanking, 查看历史复制记录， `q`退出查看
- `crs`, `crc`, `cru` cr 应该是 coerce（强制）， s是snake, c是camelcase， u是UPPER，转换变量大小写下划线形式，比较有意思,可以使用`:help abolish `查看更多
- `:NR` NarrowRgn，看名字不知道是干什么的，其实是选中一段代码，然后`:NR`vim会新开一个split把选中的代码放在心的split中操作，操作完后，`wq`会把结果覆盖掉原来选中的代码。也挺有用的。
- `,ig` toggle visual indentation guids
- `,cf` 拷贝当前文件的完整路径到系统剪切板
- `,cn` 拷贝文件名
- `,yw` yw是从复制从光标位置开始到单词结束，`,yw`是在任意位置复制整个单词
- `,ow` 使用yank buffer中内容覆盖当前所在的单词，ow => overwrite
- `,ocf` 打开所有git标记为修改过的文件，在splits中打开。使用git版本控制下的文件才有用, ocf => open changed files
- `,w` 去掉尾部多余的空格，应该很有用， StripTrailingWhitespaces
- `sj` 把单行的hash表格式化为多行的
- `sk` unsplit a link 应该不怎么用
- `,he` he => html escape
- `,hu` hu => html unescape
- `Alt+Shift-A` align things, 好像没什么用
- `:ColorToggle` 顾名思义
- `:Gitv` git log browser
- `,hi`  显示当前高亮的组
- `,gt` Go Tidy， 格式化html代码
- `:Wrap` 折叠长行，这个比较有用,特别是quickfix中的提示经常跑出去了 // 好像对quickfix部分不起作用 -_-
- `Cmd-/` toggle comments,在linux下使用alt
- `gcp` comment a paragraph

## vim相关 ##
`,vr` 重新加载vim， vim reload
