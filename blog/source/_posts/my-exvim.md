---
title: 基于exvim搭建Linux下查看Android源码的IDE
date: 2017-09-21 15:38:48
categories: 工具脚步
tags: 
  - gvim
  - exvim
  - SourceInsight
  - Android源码
---

作为一个Android开发，如果最终想进阶，都少不了去看Android源码，但是使用AndroidStudio导入Android源码，c/c++语言就没法像java那样自由跳转了；[exvim](https://exvim.github.io/)默认集成了cscope,ctags等一系列优秀的插件，能很好的阅读c/c++代码，但是我在使用过程中还是觉得有些地方不方便，鉴于此我便在exvim的基础上做了修改，搭建自己查看大型工程的源代码的IDE；比如我添加了lookupfile,LeaderF等插件用来查找文件(LearderF插件在大型项目使用过程中会报错，于是我又禁用了)，优化了exvim创建工程的默认配置，优化了文件索引已经id的创建过程，优化了默认主题等，下面我们进入正题,当然在这之前可以来张炫图  

{% asset_img imgserver.jpg This is an imgserver image %}  

# 安装修改后的exvim

按照如下流程安装
```
$ git clone git@github.com:borneywpf/main.git exvim //将 *exvim* clone到你的软件目录下
$ cd exvim
$ sh unix/install.sh
```
这个过程稍慢一些，因为要安装插件

# 修改vimrc中exvim的指向

在自己的.vimrc中添加如下配置，这样不会影响自己之前vim的配置
```
let g:exvim_custom_path='/exvim'
source /exvim/.vimrc
```

至此，这个超级IDE已经算是搞好了，是不是很惊讶！！！

> **注意:** 如过想使用LeaderF插件，自己可以在 *.vimrc.mini* *.vimrc.plugins* 中打开LeaderF
> LeaderF 需要vim7.4高版本，并且需要支持python特性，安装vim流程如下：
```
$ git clone git@github.com:vim/vim.git
$ cd vim
$ ./configure
$ sudo make
$ sudo make install
```

> 使用 *vim --version* 查看python或python3的特性是否开启，如果是+表示开启如果没有开启则运行如下命令
```
$ ./configure --enable-pythoninterp --enable-python3interp
$ sudo make
$ sudo make install
```

如上就已经搭建好自己的环境了，下面介绍使用方法

# 使用方法

进入到自己的工程目录下，运行 **gvim <工程名>.exvim** 然后修改项目配置，可参考[exvim工程配置](https://exvim.github.io/docs-zh/config-exvim/)，不过这个一般不需要自己配置，默认选项已经配置的差不多了  

在gvim命令模式下输入　**Update** 就会生产工程配置文件如lookupfile的文件,cscope的文件等，也可以在一个工程中建立子工程，比如android目录下建立一个大工程，但是可以在packages/app/Mms下建立一个小工程  

# 一些快捷键

Ctrl + 鼠标左键　方法跳转
Ctrl + 鼠标右键　跳转返回
F4 打开方法变量查看窗口  
Ctrl+F5 用lookupfile的方式查找文件,这个建议小工程用  
Ctrl+F8 打开工程目录树，使用的时nerdtree插件  
Ctrl+P 使用ctrlp查找文件，这个可以大工程用  
Ctrl+Shift+- 快速按 c 可以找到函数的调用处，cscope的用法可以自己去搜索下  
Shift + 方向键　可以切换窗口  
命令行 **b <file name>** 可以打开buffer中的文件  

exvim 集成的插件比较多也比较全，具体每个插件怎么用，我还没有深入研究，只有使用的时候通过help查看，如果有兴趣可以自己去研究下
