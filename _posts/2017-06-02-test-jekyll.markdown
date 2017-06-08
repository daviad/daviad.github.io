---
layout: post
title: 从零开始使用jekyll + GitHub pages 制作 blog。
---
以前，在网上也看到过，很多教程，都学的一知半解。 这次静下心来，通过官网 http://jekyll.com.cn  一步一步从零开始制作blog。

按照官网 开始jekyll new myblog，就遇到问题 
>Dependency Error: Yikes! It looks like you don't have bundler or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. The full error message from Ruby is: 'cannot load such file -- bundler' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/! 
jekyll 3.4.0 | Error:  bundler

最终在 [这里](https://github.com/jekyll/jekyll/issues/4688) 找到答案 
>@prasadariyawansa looks like you need to run gem install bundler

查看文件index.md 说明. 看到 [这个链接](https://jekyllrb.com/docs/themes/#overriding-theme-defaults) 及jekyll官网，明白了:默认的模板是minima，及模板的大致原理。在明白这些后，就可以制做自己风格的blog.

### github-pages 是个插件，可以不使用
查看 Gemfile 文件 按照如果使用github-pages 的操作，（这部分内容不做也是可以使用GitHub的。我一开始没有修改是可以使用的，但为了学习，还是按照文档的说明做做比较好，你没有猜错，又遇到问题）
由于jekyll 由gem管理，最好先了解下gem,bundle,ruby.

rvm是用来管理ruby的，ruby的其中一个“程序”叫rubygems，简称 gem，而用来管理项目 的gem的，叫bundle.完全是不同的东西，他们相同的只是都可以管理gem


我的版本有些低，所以用命令 sudo gem update --system 升级
安装升级bundle 权限问题
>From https://github.com/bundler/bundler/issues/4065
You'll need to either change your GEM_HOME or do
sudo gem install bundler -n /usr/local/bin 
because of El Cap's introduction of SIP (System Integrity Protection).

接下bundle update 总是失败，无法成功。于是修改 mac SIP。
用rvm 删除ruby 重新安装ruby 
bundler
使用 rvm  升级ruby
rvm list known
sudo rvm install 2.4
>Running Homebrew as root is extremely dangerous and no longer supported.
As Homebrew does not drop privileges on installation you would be giving all
build scripts full access to your system.
去掉sudo

gem install bundler 


这篇文章会持续更新，使用jekyll 的心得。
心得：
1.纸上得来终觉浅， 绝知此事要躬行。  
2.学习东西，最好能够看原版，其他，资料为辅助。

update  
1.jekyll 草稿的使用。中文文档的： 运行 jekyll serve 或者带有 --drafts 配置选项的 jekyll build。翻译有些问题，请看英文文档。 server命令后也要加 \-\-drafts 不然看不到草稿的。

#### insert picture
![test picture]({{site.url}}/images/test.png)
