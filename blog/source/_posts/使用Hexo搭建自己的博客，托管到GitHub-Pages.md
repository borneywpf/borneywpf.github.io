---
title: 使用Hexo搭建自己的博客，托管到GitHub Pages
date: 2017-09-21 10:07:54
categories: 工具脚步
tags: 
  - Hexo
  - 博客
  - NexT
  - GitHub Pages
---
我使用过好几种方式搭建自己的博客，比如[Octopress](http://octopress.org/),它的安装方式见[这里](http://octopress.org/docs/setup/)，这是一个不错的博客框架，但是我在使用的过程中觉得它还不是够简洁，环境搭建还是有点略复杂(其实已经很简单了)；我还用过开源项目[BGAIssueBlog](https://github.com/bingoogolapple/BGAIssueBlog),这个是通过github issues来维护自己的博客，我个人觉得这个作者还是很牛的(关键我也太菜了^_^)，但是这个开源项目还有很多功能需要添加比如统计等等；其实我很早就关注了Hexo，我觉得Hexo主题比较丰富，而且界面比较简洁，速度也比较快，只是太过懒惰就懒的倒腾了，这几天良心发现于是就决定倒腾一下，虽然网上有Hexo的搭建过程，但是还是觉得自己再搞一个文档出来，针对自己的实际情况来搞出适合自己的写作环境，废话太多了，我们开始吧！！  

# 什么是Hexo

  这个可以通过它的[官网](https://hexo.io/)首页的一句话概述"A fast, simple & powerful blog framework",真是霸气侧漏啊！！！
  
# 搭建环境

  官网的doc其实已经很详细了，我们在这里就再赘述下吧(我算是忠实的Linux用户，Windows的方式我就忽略了，这里只列出Linux的安装方式了，如果想看Windowｓ的，就自己看[文档](https://hexo.io/zh-cn/docs/)吧)  
  ## 安装git
  
  ```
  $ sudo apt-get install git-core
  ```
  ## 安装node.js
  
  安装 Node.js 的最佳方式是使用 nvm
  cURL:
  ```
  $ curl https://raw.github.com/creationix/nvm/master/install.sh | sh
  ```
  
  Wget:
  ```
  $ wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
  
  ```
  安装完成后，重启终端并执行下列命令即可安装 Node.js
  ```
  $ nvm install stable
  ```
  
  ## 安装 Hexo
  
  ```
  $ npm install -g hexo-cli
  ```
  
  Ok,到这里你的博客环境已经搭建好了，是不是很惊讶，就是这么简单粗暴

# Hexo的建站
  安装完Hexo，我们就可以开始建立blog了，不过我到这里就不同了，因为我们不可能只在一台电脑上管理维护自己的博客，所以我决定将文档托管到github上
  
  ## 创建自己的GitHub Pages
  什么是GitHub Pages，你自己看官网的介绍吧，地址在[这里](https://pages.github.com/)
  
  clone工程
  ```
  $ git clone https://github.com/username/username.github.io
  $ cd username.github.io
  ```
  
  ## 开始创建博客，并托管到github
  采用**Octopress**的托管思想，将博客托管到username.github.io的source分之中
  ```
  $ git checkout -b source
  ```
  
  开始创建博客
  ```
  $ hexo init <folder> #这里的<folder>，我后边都用blog代替了
  $ cd blog
  $ npm install
  ```
  
  查看效果，运行如下命令，然后通过浏览器 http://localhost:4000/ 预览本地效果；至于hexo命令的详细介绍可以查看官方文档或通过 **hexo help** 查看
  ```
  $ hexo g
  $ hexo s
  ```
  
  推送搭建好的环境到github上，可以查看blog目录下的 **.gitignore** 文件，观察有那些文件是托管时是被忽略的
  ```
  $ git add .
  $ git commit -am "create blog"
  $ git push origin source #托管到source分之上
  ```
  ## 配置博客
  
  基本配置，我就说下基本的吧，具体大家可以参考官网  
  打开blog下的 **_config.yml **文件(之后称root配置文件)，做如下修改  
  ```
  title: XXXXX的博客 # 博客标题
  subtitle:   # 副标题
  description:  # 描述
  author: XXXXXX  # 作者
  ```
  
  配置主题，Hexo的主题有很多，见[这里](https://hexo.io/themes/),我自己配置的是[NexT](http://theme-next.iissnan.com/)
  ```
  $ git clone https://github.com/iissnan/hexo-theme-next themes/next
  ```
  
  修改主题，打开root配置文件，将修改theme为next就可以了
  
  NexT配置Scheme，打开themes/next/_config.yml文件(之后称next配置文件)，文件中scheme有４种，选择自己喜欢的就可以了  
  博客主页的文章默认是全部显示的，我们可以配置只显示多少个字，然后缩进，将auto_excerpt的enable设置为true，length就是显示的字数；还有很多其他设置如语言等，这些大家都可以看文档，这里就不赘述了
  
  ## 托管主题
  
  如上我们已经配置好博客的主题，现在我们也将主题托管到github,在blog目录中，进行如下操作
  ```
  $ rm -rf themes/next/.git #删除主题下的.git文件
  $ git add themes/next
  $ git commit -am "add next themes"
  $ git push origin source
  ```
  
  ## 更换电脑后
  
  如果我们更换了电脑那么要怎么做呢
  - 首先要搭建环境，具体步骤参见上文
  - 其次clone托管的博客,如下
  ```
  $ git clone https://github.com/username/username.github.io
  $ git checkout source #切换分之
  $ npm install #这一步很重要
  ```
  
  至此，写博客的托管环境也配置好了，我们再也不用担心学习的环境变了还要重新配置了。
  
# 部署hexo到GitHub Pages
  
  修改GitHub Pages关联配置,在root的配置文件中，修改deploy配置
  ```
  deploy:
  type: git
  repo: git@github.com:username/username.github.io.git #如果在github上配置了ssh key用git协议，否则只能用https了
  branch: master
  ```
  
  安装 hexo-deployer-git,在blog目录下
  ```
  $ npm install hexo-deployer-git --save
  ```
  然后，确保通过 **hexo g** 已经生成了静态网页,用如下命令进行部署
  ```
  $ hexo d
  ```
  
  如果有自己的域名想关联到GitHub Pages,可以在blog目录下，通过下面的方法关联
  ```
  $ touch source/CNAME
  $ vi source/CNAME # 然后将自己的域名写入CNAME中
  $ hexo g # 重新生产静态网页
  $ hexo d # 重新部署
  ```
至此一个简单的写作环境就搭建好了，简单粗暴，使用GitHub Pages的master进行博客部署，用source分之托管文档，便于维护；如果还想更细致的配置，比如统计，留言等功能，可以参考Hexo的文档；编辑markdown文档推荐[Atom](https://atom.io/)这款编辑器，绝对赞;文章内容比较比较简单，但基本上已经将清楚了，如果还有哪里不明白，可以联系我，我们共同交流学习。

-------
参考文档  
[添加评论、浏览次数、打赏、RSS等功能参考](http://www.jianshu.com/p/5973c05d7100)
[统计分析的地址参考](http://col.dog/2015/11/12/hello-world/)
