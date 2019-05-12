title: Vim 文件快速定位
date: 2015-10-14 13:49:44
categories:
tags:
- vim
---
## :find 命令
在 vim 的 path 路径中搜索并打开文件
使用方式 :find {file}，可以用 tab 进行补全
Linux 中 vim 默认 path

```vim
path=.,/usr/include,,
```

<!--more-->
即会在当前文件所在目录，/usr/include目录以及**当前目录**（vim中cd指定，pwd显示的目录）中搜索。可以用 :set path+={dir} 添加至搜索路径。
因为不能递归搜索，所以即使当前目录为项目路径，也不能对所有文件进行搜索。

**解决办法：**
在 .vimrc 中修改为：

```vim
path=.,/usr/include,**
```

\*\* 为 vim 中的提供的功能，可以对路径递归搜索，详细见 :h \*
app/\*\* 表示为对 app/ 下所有路径进行搜索。因为空缺代表当前目录，所以 \*\*就是对当前目录进行递归搜索。
