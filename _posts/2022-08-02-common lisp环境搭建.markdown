---
layout: post
title:  "common lisp 环境搭建"
date:   2022-08-02 2:43:00 +0800
categories: language lisp 
---
# 建立 common lisp 环境 (windows 上)
> common lisp 官网
> https://common-lisp.net/downloads

和其他语言一样,common lisp (以下简称 lisp) 有自己的一套编辑器,构建系统和包管理器(其中编辑器对于vscode的支持不够好)

# 基础设施
Lisp 基础设施有三部分:
- 一个lisp解释器
- 一个连接到解释器的文本编辑器, 
- 包管理器兼构建器

最常见的方法是
- IDE/文本编辑器 : Emacs 或 Slime
- 包管理器/构建器 : [ASDF](https://asdf.common-lisp.dev/) + [Quicklisp](https://www.quicklisp.org/)


SLIME 是 Emacs 编辑器的一个扩展,把当前的编辑器连接到一个正在运行的lisp映像(称作 *inferior-lisp*),并且和它互动.你可以用它对lisp代码求值,编译,展开宏,生成文档,做代码导航,调试,等等等.(自注: 我觉得现在它可以写成一个lsp的形式,这样我就可以把它移植到我喜欢的编辑器上,比如vscode,目前vscode上好用的插件有Alive) 见[commonlisp cookbook](https://lispcookbook.github.io/cl-cookbook/#download-in-epub)

[ASDF](https://asdf.common-lisp.dev/) 是 Lisp 版的 Make. 他可以用来定义项目(projects,原文也称系统-systems),项目依赖,还有加载/编译项目.

[Quicklisp](https://www.quicklisp.org/) 是 Common Lisp的库管理器.你可以用它下载,安装,加载1500多个库,只需要几个简单的命令.

# 生成可执行文件

参考[cookbook](https://lispcookbook.github.io/cl-cookbook/scripting.html)

鉴于历史惯性,我总觉得生成一个可以独立运行的程序才算及程序的最终完成.lisp并不是一个主要用来编译的语言,所以这方面功能相对难找一些.

在sbcl 事情是这样子的
```lisp
(sb-ext:save-lisp-and-die 
  #P"path/name-of-executable" 
  :toplevel #'my-app:main-function 
  :executable t)
```
`sb-ext` is an SBCL extension to run external processes. See other SBCL extensions (many of them are made implementation-portable in other libraries).

`:executable t `表示要构建的是个可执行程序,不是一个镜像. 保存程序的目的自不用说,保存镜像的目的是留着下次直接打开.这对于计算量大的程序尤其顶用,我们直接读取镜像,就免得它再计算一遍.

用 `sbcl --core name-of-image` 读取镜像.

`:toplevel` 表示程序的入口点,相当于`main`函数.此处是`my-app:main-function`. 记得把符号导出去,或者用`my-app::main-function`强行找到.(两个冒号)

比如这个`hello.lisp`
```lisp
(defun main ()
  (format T "Hello World,I'm lisp~%")); '~%' 意思是换行



(sb-ext:save-lisp-and-die
  "hello.exe"
  :executable T
  :toplevel 'main)
```
运行程序,它编译完自己就退出去了.
```
$ sbcl --load .\hello.lisp
This is SBCL 2.2.4, an implementation of ANSI Common Lisp.
More information about SBCL is available at <http://www.sbcl.org/>.

SBCL is free software, provided as is, with absolutely no warranty.
It is mostly in the public domain; some portions are provided under
BSD-style licenses.  See the CREDITS and COPYING files in the
distribution for more information.
[undoing binding stack and other enclosing state... done]
[performing final GC... done]
[saving current Lisp image into hello.exe:
writing 6544 bytes from the read-only space at 0000000020000000
writing 2016 bytes from the static space at 0000000020220000
writing 37453824 bytes from the dynamic space at 0000001000000000
done]
```
现在你的目录里面应该有这些东西
```
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----          2022/8/2      4:02            138 hello.lisp
-a----          2022/8/2      4:03       39529656 hello.exe
```
他真大啊.其实lisp在保存程序的时候把整个lisp镜像都打进去了,一个大概30多M.

## 代码风格
见[CommonLisp代码风格](https://lisp-lang.org/style-guide/)
