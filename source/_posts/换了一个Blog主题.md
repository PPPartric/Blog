---
title: 换了一个 Blog 主题
date: 2016-11-11 09:17:09
tags: 
- Hexo
- Yilia
- Theme
categories: Blog
---

# yilia

前几天deploy博客的时候，发现打开blog页面是空的，只有head部分显示出来了。打开控制台排查问题，发现hexo主题里面有几个外部ajax call失败，导致整个页面都没有渲染出来，这是一件恼火的事情。<img src="/assets/img/angry.png" alt="我是一只生气的图片">

于是果断换主题，其实对之前的主题还是很满意的： 简洁，渲染速度也很快，功能虽然不多，但是基本满足我的需求。

这次选择的主题是腾讯的工程师Litten制作的 [「yilia」](https://github.com/litten/hexo-theme-yilia)

「yilia」 同样是我喜欢的简洁样式，作者甚至移除了搜索框。而且对于移动端的优化也做得很不错。
<!-- more -->
# yotuku

之前Blog里面的图片一直都选择 [「yotuku」](http://yotuku.cn/) 生成在线图片，然后在markdown里面引用，如果图片大小或者位置不合适的话，会在md里面手写一段html，这样做很省事。

今天早上看自己的Blog发现有几张图片没有加载出来，以为是新主题渲染的问题，重新deploy以后发现还是没有。看来不是主题的锅。

使用控制台发现

<img src="/assets/img/ajax_call_failure.png" alt="我是一只失败的图片">

原来这几张图片都没有拿到，已经在官网留言，希望能够解决。

不过使用免费云服务存储自己Blog的图片确实不太安全，像这样丢失图片的行为可能会导致几张图片加载不出来，但是如果以后云服务提供商挂掉了（这里不是诅咒yutuku不好，希望这样良心企业越来越好），那这些图片岂不就再也找不到了。

还是老老实实把图片放到Blog路径下，用相对地址引用吧。

