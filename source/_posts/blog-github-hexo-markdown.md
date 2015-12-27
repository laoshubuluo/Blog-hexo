title: Blog-搭建自己技术博客之github+hexo+markdown
description: 搭建自己技术博客之github+hexo+markdown
date: 2015/10/7 2:45:20 
categories: Blog
tags: [blog,github,hexo,markdown,分享]
---
使用github+hexo+markdown,搭建个人技术博客。
<!--more-->
# 介绍 #
一直想搭建自己的技术博客，听过很多平台，目测github各种火，各种高逼格，所以决定在github pages上搭建。但是，百度半天，发现各种没玩过 git、github、Ruby、Devkit、Jekll、markdown，好吧，我是个多白的小白~外加windows安装上述环境各种哭，各种报错，险些放弃。最终遇到你，还好我没有放弃：github+hexo+markdown——**如果你想简单，它可以足够简单；如果你想折腾，它能任你折腾。**

引用[@不如](http://ibruce.info/)大神对hexo的介绍：
> hexo出自台湾大学生tommy351之手，是一个基于Node.js的静态博客程序，其编译上百篇文字只需要几秒。hexo生成的静态网页可以直接放到GitHub Pages，BAE，SAE等平台上。

引用[@tommy351](https://github.com/hexojs/hexo)的介绍：
>hexo：快速、简单且功能强大的 Node.js 博客框架。

# 特别感谢 #
因为是小白，操作过程免不了参考大神的经验，所以特别感谢下述大神提供宝贵资料，这篇博客也“抄袭”了不少大神内容，非常感谢。如涉及侵权，请与我联系~

- [http://ibruce.info/2013/11/22/hexo-your-blog/](http://ibruce.info/2013/11/22/hexo-your-blog/)
- [http://zipperary.com/2013/05/28/hexo-guide-1](http://zipperary.com/2013/05/28/hexo-guide-1)
- [http://zipperary.com/2013/05/28/hexo-guide-2](http://zipperary.com/2013/05/28/hexo-guide-2)
- [http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)

# 环境准备 #
**安装Node**

到Node.js官网下载相应平台的最新版本，正常安装即可。

**安装Git**

Git的客户端很多，我安装的是GitExtensions，安装过程中，会绑定安装msysgit和Git，正常安装即可。

**Markdown编辑工具（可选）**

用来编写blog使用，可参考这篇介绍：[http://code.csdn.net/news/2819623](http://code.csdn.net/news/2819623)，我用的[MarkdownPad](http://markdownpad.com/) ，赶脚不错，其他还没尝试，有更不错的工具，还请留言介绍。

# 正式开始 #
GitHub注册账号，创建仓库repository，名称必须为[user_name.github.io]，我的为laoshubuluo.github.io，此过程有许多介绍，不再赘述。

## 安装hexo ##
cmd命令进入到你即将编写博客的目录（类似于博客的workspace）,我的目录是**D:\Blog\hexo**（后续所有命令均在此目录下完成），即

    C:\windows\System32>D: 
    D:\>cd D:\Blog\hexo

利用 npm 命令安装hexo

    npm install -g hexo

初始化hexo文件夹

    hexo init

安装依赖包

    npm install

本地查看

现在hexo默认博客内容已经在本地搭建成功，在hexo路径，执行下述命令，然后在浏览器输入localhost:4000即可查看。

    hexo generate

    hexo server

生成静态页面并部署

编辑D:\Blog\hexo目录下的_config.yml文件。将所有laoshubuluo替换成你的用户名（laoshubuluo是我的用户名）。

    deploy:
      type: git
      repository: https://github.com/laoshubuluo/laoshubuluo.github.io.git
      branch: master
	
执行下列指令完成部署

    hexo generate
    hexo deploy

*注意：

有些新用户需要设置 ssh，否则上述命令会失败。ssh 的介绍和设置方法请看官方教程。

当你修改文章Tag或内容，不能正确重新生成内容，可以删除hexo\db.json后重试，还不行就到public目录删除对应的文件，重新生成。*

**记住：每次修改本地文件后，需要hexo generate才能生成静态网页并保存，hexo deploy才能部署到github。**

OK,至此几个基本框架的博客已经搭建并部署完成，在浏览器访问laoshubuluo.github.io即可访问。哦也~

## 创建新文章 ##

执行new命令，生成指定名称的文章至hexo\source\_posts\postName.md。

    hexo new [layout] "postName"

其中layout是可选参数，默认值为post。有哪些layout呢，请到scaffolds目录下查看，这些文件名称就是layout名称。当然你可以添加自己的layout，方法就是添加一个文件即可，同时你也可以编辑现有的layout，比如post的layout默认是hexo\scaffolds\post.md

    title: #title
    date: #date
    categories: #文章分类目录，可以为空，注意:后面有个空格
    tags: #文章标签，可空，多标签请用格式[tag1,tag2,tag3]，注意:后面有个空格
    description: #你对本页的描述
    ---
    以上是摘要
    <!--more-->
    以下是余下全文

postName是md文件的名字，同时也出现在你文章的URL中，postName如果包含空格，必须用”将其包围，postName可以为中文。

*注意，所有“冒号”后面都必须有个空格，不然会报错。*

接下来，就可以打开生成的.md文件，利用markdown语法，进行挥洒笔墨了。markdown语法，此处不做过多介绍。 

## 创建页面 ##
执行new page命令

    hexo new page "about"

在hexo\source\下会生成about目录，里面有个index.md，可以直接编辑，然后在主题的_config.yml中将其入口配置显示出来。自动生成的路径及文件，手动创建也可。


## 网站图标 ##
修改**hexo\themes\modernist\layout\_partial\head.ejs**文件，如下语句：

    <% if (theme.favicon){ %>
    <link rel="icon" type="image/x-icon" href="<%- config.root %>favicon.ico">
    <% } %>
将ico文件放到工程根目录：hexo\source。

# 命令大全 #
## 常用命令 ##

    hexo new "postName" #新建文章
    hexo new page "pageName" #新建页面
    hexo generate #生成静态页面至public目录
    hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
    hexo deploy #将.deploy目录部署到GitHub

## 常用复合命令 ##

    hexo deploy -g
    hexo server -g

## 简写 ##

    hexo n == hexo new
    hexo g == hexo generate
    hexo s == hexo server
    hexo d == hexo deploy

# 评论插件 #
hexo默认集成的是Disqus，国内的话建议用[多说](http://dev.duoshuo.com/)。

获取多说账号及集成代码
用QQ、微博等通过开放平台成功登录多说。

选择多说首页的“我要安装”入口（创建站点），按要求输入相关内容，创建成功。

站点创建成功后，首页会出现“后台管理”入口，可通过此进入之前创建的站点的管理后台。

获取short_name：进入**设置-域名**，duoshuo.com前的字段即是，后面会用到。

获取集成通用源码：进入**设置-工具-获取代码-通用代码**，文本框内即是，后面会用到。

管理后台也可进行评论各种设置的修改操作，后续介绍。

## 集成多说插件 ##
在**_config.yml**中添加多说的配置：

    duoshuo_shortname: 你站点的short_name

修改**themes\landscape\layout\_partial\article.ejs**模板文件

把Disqus

    <% if (!index && post.comments && config.disqus_shortname){ %>
    <section id="comments">
      <div id="disqus_thread">
    <noscript>Please enable JavaScript to view the <a href="//disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
      </div>
    </section>
    <% } %>

替换成多说

    <!-- 多说评论框 start -->
    	<div class="ds-thread" data-thread-key="<%= post.layout %>-<%= post.slug %>" data-title="<%= post.title %>" data-url="<%= page.permalink %>"></div>
    <!-- 多说评论框 end -->
    <!-- 多说公共JS代码 start (一个网页只需插入一次) -->
    <script type="text/javascript">
    var duoshuoQuery = {short_name:"laoshubuluo"};
    	(function() {
    		var ds = document.createElement('script');
    		ds.type = 'text/javascript';ds.async = true;
    		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
    		ds.charset = 'UTF-8';
    		(document.getElementsByTagName('head')[0] 
    		 || document.getElementsByTagName('body')[0]).appendChild(ds);
    	})();
    	</script>
    <!-- 多说公共JS代码 end -->
重新生成并部署站点。

# 搜索引擎 #

为了尽快被搜索引擎收录，可以到[屈站长](http://www.sousuoyinqingtijiao.com/)提交你的站点给搜索引擎。

# 访问统计 #
除了已知的统计插件外（此处不再介绍），推荐一款新的统计插件[不蒜子](http://service.ibruce.info/)，两行js代码，即可搞定计数。

    <script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>
    <span id="busuanzi_container_site_pv">本站总访问量<span id="busuanzi_value_site_pv"></span>次</span>

# 异常问题及解决方案 #
待补充