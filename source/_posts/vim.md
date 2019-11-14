---
title: Vim
date: 2018-11-22
categories:
  - linux
tags:
  - linux
  - vim
---

<!-- more -->

## 编辑文件 /etc/vimrc

```
autocmd BufNewFile *.py,*.sh,*, exec ":call SetTitle()"
let $author_name = "jeremy"
let $filetype_name = strpart(expand("%"), stridx(expand("%"), "."))
let $file_name = strpart(expand("%"), 0, stridx(expand("%"), "."))

func SetTitle()
if &filetype == 'sh'
call setline(1, "\#!/bin/bash")
call setline(2, "\# writen by: ".$author_name)
call setline(3, "\# file name: ".$file_name.$filetype_name)
endif
endfunc
```