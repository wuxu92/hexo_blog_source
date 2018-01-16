---
title: Conver all GBK files to UTF8
date: 2017-10-21 22:50:36
tags: 
- Linux
- Bash
categories:
- Linux
---
*水一篇水一篇，要不要发霉了*
平时的整轨音乐文件作者在发布的时候使用的播放列表文件（cue/m3u/m3u8）大部分都是 BGK编码的，在qmmp播放时不能正确识别GBK编码的文件，之前都是用vscode一个个复制转码保存，今天处理歌神的一个合集时发现文件实在太多了，就想着用bash处理以下，发现非常简单的一个命令就能搞定了：

```
while read -r f; do echo "Processing $f..."; iconv -f GBK -t utf8 -o "${f%%cue}utf8.cue"  $f; done <<< $(find ./ -name '*.cue');
````
就是用iconv命令进行转码，上面的命令可以将当前目录下的所有 .cue 文件从 BGK 转码为 UTF8 并保存为一个带 .utf8.cue 的同目录下的文件，这样使用起来就很方便了。

虽然很简单也很水，但很好用啊。。。。

再附一个处理所有的的详细版(要排除了一些之前手动处理过的文件)：

```
#! /bin/bash

# mydir=$(readlink -e `dirname $0`)
skip_count=0
work_count=0

while read -r f; do
    if [[ -n `echo "$f" | grep utf8` || -f "${f%%cue}utf8.cue" || -f "${f%%.cue} utf8.cue" ]]; then
        echo "skiping $f"
        skip_count=$((skip_count+1))
        continue
    fi

    echo "Processing $f..."
    iconv -f GBK -t utf8 -o "${f%%cue}utf8.cue" "$f"
    work_count=$((work_count+1))
done < <(find ./ -name '*.cue')

echo "done: skipped: $skip_count, Processed: $work_count."
```
