---
layout: post
title: 搭建jekyll运行环境
date: 2015-12-25
categories:
- jekyll
tags: [jekyll]
status: publish
type: post
published: true
---
jekyll是一个不可思议的博客系统，优雅，简单。

要用jekyll写博客，我们需要让jekyll在我们自己的电脑上跑起来。

### 搭建步骤
这里使用的是jekyll 2.0/3.0，需要ruby2.0或以上的支持

#### 安装ruby2.0
{% highlight bash %}
sudo apt-get install ruby2.0 ruby2.0-dev
{% endhighlight %}

#### 更新gems(使用root或者sudo运行)
{% highlight bash %}
gem install rubygems-update
update_rubygems
gem update --system
{% endhighlight %}

#### 安装需要的gems
{% highlight bash %}
sudo gem2.0 install pygments.rb
sudo gem2.0 install redcarpet.rb
{% endhighlight %}

pygments是代码高亮,redcarpet是markdown解析器。

github pages 现在只支持kramdown 和rouge了，使用相同方式安装即可。


#### 安装jekyll
{% highlight bash %}
sudo gem2.0  install jekyll
{% endhighlight %}

如果遇到下面的错误，说明你需要翻Q了：

{% highlight bash %}
ERROR: While executing gem ... (Gem::RemoteFetcher::FetchError)
Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://api.rubygems.org/quick/Marshal.4.8/jekyll-3.0.1.gemspec.rz)
{% endhighlight %}

则需要更换成taobao的gem source
{% highlight bash %}
sudo gem sources −−remove  https://rubygems.org/ 
sudo gem sources -a https://ruby.taobao.org/
{% endhighlight %}

再确认一下：
{% highlight bash %}
$ gem sources -l
*** CURRENT SOURCES ***
https://ruby.taobao.org
{% endhighlight %}
出现上面的信息说明已经更换成功。

#### 创建一个叫my-blog的项目
{% highlight bash %}
jekyll new my-blog
{% endhighlight %}

这行命令将会产生一个my-blog的文件夹，然后我们就可以在那个文件夹里面倒腾了。
文件结构可以看[文件目录官方文档](http://jekyllrb.com/docs/structure/)

#### 运行项目
{% highlight bash %}
cd my-blog
jekyll serve
{% endhighlight %}
一般情况下，我们访问127.0.0.1:4000就可以访问我们的博客了。
但是有时候会出现缺少组件的问题,我遇到过下面的错误：

{% highlight bash %}
Configuration file: /home/slayer/vhost/blog/_config.yml
/usr/lib/ruby/2.0.0/rubygems/core_ext/kernel_require.rb:53:in `require': cannot load such file -- jekyll-sitemap (LoadError)`
{% endhighlight %}

这是因为我的博客使用了一个叫sitemap的组件没有安装，安装上即可：
{% highlight bash %}
sudo gem2.0 install jekyll-sitemap
{% endhighlight %}

然后就可以愉快写博客了，感谢jekyll的作者为我们提供了这么好的作品。

附上[官方文档链接](https://jekyllrb.com/docs/home/)，基本上花几十分钟看完就能熟练的使用了




