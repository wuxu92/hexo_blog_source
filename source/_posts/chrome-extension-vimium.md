---
date: 2015/11/23 12:34:50
title: Chrome插件Vimium
categories:
- tools
tags:
- chrome
- extension
- efficient
- vim
---

Vimuim是一款很酷的chrome插件，它能让你使用Vim的方式操作你的chrome浏览器，滚动，打开链接，tab跳转，复制，搜索，功能非常强大。

在有鼠标的时候可以使用鼠标手势快速操作浏览器，但是在使用笔记本（我现在的T430）的时候，基本不使用鼠标，使用小红点浏览网页的时候还是有点麻烦。今天使用了一下大名鼎鼎的Vimium插件，简直解放了我的食指。

首先安装插件，地址是 [https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?utm_source=chrome-ntp-icon ](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?utm_source=chrome-ntp-icon) ，另外还有一些类似的插件，像 cVim 等，不过用的最多是这个 Vimium。
<!-- more -->
安装完后就可以使用了，不过相比 Wgestures 的鼠标手势的缺点是在只能在网页页面使用，也就是在应用中心，设置等页面是不能使用的（这是chrome安全策略限制的）。不过已经完全够用了。

其用法和Vim的操作很像，使用h,j,k,l操作页面的滚动，j向下滚屏，k向上滚屏，简直太方便了啊，再也不用费劲的使用小红点加触控板滚屏了。其他的操作可以查看插件安装也的说明，其中比较常用的f, F, d, u, gg, G, r, x, X, yy, o, T, /, H, L。忘记了快捷键可以使用 `?` 唤出快捷键帮助。

具体可以看下面的总结：

Modifier keys are specified as <c-x>, <m-x>, <a-x> for ctrl+x, meta+x, and alt+x respectively.
 
Navigating the current page:
 
    ?       show the help dialog for a list of all available keys
    h       scroll left
    j       scroll down
    k       scroll up
    l       scroll right
    gg      scroll to top of the page
    G       scroll to bottom of the page
    d       scroll down half a page
    u       scroll up half a page
    f       open a link in the current tab
    F       open a link in a new tab
    r       reload
    gs      view source
    i       enter insert mode -- all commands will be ignored until you hit esc to exit
    yy      copy the current url to the clipboard
    yf      copy a link url to the clipboard
    gf      cycle forward to the next frame
 
Navigating to new pages:
 
    o       Open URL, bookmark, or history entry
    O       Open URL, bookmark, history entry in a new tab
    b       Open bookmark
    B       Open bookmark in a new tab
 
Using find:
 
    /       enter find mode -- type your search query and hit enter to search or esc to cancel
    n       cycle forward to the next find match
    N       cycle backward to the previous find match
 
Navigating your history:
 
    H       go back in history
    L       go forward in history
 
Manipulating tabs:
 
    J, gT      go one tab left
    K, gt      go one tab right
    g0         go to the first tab
    g$         go to the last tab
    t          create tab
    x          close current tab
    X          restore closed tab (i.e. unwind the 'x' command)
    T          search through your open tabs
 
Additional advanced browsing commands:
 
    ]]      Follow the link labeled 'next' or '>'. Helpful for browsing paginated sites.
    [[      Follow the link labeled 'previous' or '<'. Helpful for browsing paginated sites.
    <a-f>   open multiple links in a new tab
    gi      focus the first (or n-th) text input box on the page
    gu      go up one level in the URL hierarchy

