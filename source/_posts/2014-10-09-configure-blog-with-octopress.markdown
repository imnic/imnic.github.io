---
layout: post
title: "使用Octopress搭建自己的博客系统"
date: 2014-10-09 16:55:54 +0800
comments: true
description: 如何在Mac上使用Octopress搭建自己的博客系统.
keywords: octopress, blog, 博客, iOS
categories: 工具
---
一直都有写技术博客的想法，今天花了一些时间研究了一下如何使用Octopress来搭建自己的博客，从早上来就开始折腾，一直到现在，才算是全部搞定，那第一篇blog就先记录一下整个过程吧，也算是个良好的开端。  

具体的步骤其实完全可以按照 [Octopress](http://octopress.org)官网给的步骤一步一步来就可以了，在此做一些简单的归纳，具体的步骤可以分为三部分：  
<!-- more -->
###第一部分：Octopress的安装及配置
1. **安装** [Git](http://http://git-scm.com),如果你的电脑上已经安装过了，可以忽略此步骤。  

2. **安装 Ruby：**可以使用[Rbenv](http://octopress.org/docs/setup/rbenv)或者[RVM](http://octopress.org/docs/setup/rvm)两种方式安装，注意，我们要保证ruby的版本不能低于1.9.3，具体的查看方式为：在Terminal中输入 ruby --version 命令查。   
3. **设置 Octopess：** 
{%codeblock%}git clone git://github.com/imathis/octopress.git octpress       
cd octopress {%endcodeblock%}   
4. **安装依赖：**  
{%codeblock%}gem install bundler  	
rbenv rehash  
bundle install{%endcodeblock%}	
5. **安装octopres的默认主题：**   
{%codeblock%}rake install{%endcodeblock%}	
  
###第二部分：Github Pages相关的部署及配置： 
1. 当然你首先要有个Github账号，然后建立一个新的repo，repo的名字为youraccountname.github.io。  
**注意:Github Pages已经将链接改为xxxx.github.io不再是xxxx.github.com**  

2. 在Terminal中输入 rake setup_github_pages 命令，输入之后，Terminal将会提示你输入你刚才新建的repo的url，比如：git@github.com:username/username.github.io.git，直接复制粘贴即可。  

###第三部分，写blog，常用的几个命令如下：  
1. **rake new_post ["the title of your blog"]**,使用此命令可以会在$OCTOPRESS/source/_posts中新建一个markdown文件,Mac下可以使用Mou对其进行编辑，这个文件就是我们blog的原文件。  
2. **rake generate**  
3. **git add .**
4. **git commit -m "xxxxx"**
5. **git push origin source**
6. **rake preview**,输入完此命令，可以打开浏览器输入<http://localhost:4000>预览效果。  
7. **rake deploy**

PS:在使用rake deploy部署的过程中，我遇到了以下问题，git 提示：  
  
{%codeblock%}! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'https://github.com/yeesterbunny/yeesterbunny.github.com.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Merge the remote changes (e.g. 'git pull')
hint: before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.{%endcodeblock%}

这里参照了[这篇博客](http://allenyee.me/blog/2013/08/21/what-i-learned-from-hosting-octopress-on-github/)的解决办法。
 
以上就是使用Octopress来搭建个人博客的一些流程，另外我们可能还会需要更进一步的去定制你的个人博客，比如给你的博客添加评论和分享功能，定制你喜爱的主题等等，在此我就不再一一赘述，网上也有一些很不错的博客来介绍，比如[这篇博客](http://biaobiaoqi.me/blog/2013/07/10/decorate-octopress/)就介绍的很详细。

最后，也算是给自己提个醒，以后无论是开发或者研究一项新技术或者遇到其他的技术问题的时候，第一时间不要着急去google或者stackoverflow去查答案，要先看看要没有官方文档，有的话要第一时间去翻阅文档，看看自己的方法有没有问题，因为网上的各种资料可能会有偏差，而且随着时间的推移，官方文档可能会有更新，但网上的资料可能还是当时的解决方案，因此，遇到问题一定要先从官方文档着手。  

当然，在这里也要感谢前辈们的付出，我在学习的时候参照了他们的博客：  
<http://blog.devtang.com/blog/2012/02/10/setup-blog-based-on-github/>   

<http://biaobiaoqi.me/blog/2013/07/10/decorate-octopress/>   

<http://alvinzhu.me/blog/2013/10/02/an-zhuang-he-pei-zhi-octopress/>  

