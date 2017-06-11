---
title: Github Pages + Hexo创建个人博客
date: 2016-09-02 17:17:52
tags: Hexo
categories: Hexo
---

> 之前使用过jekyll+github做过一版自己的博客网站，有兴趣的可以看一下我之前的文章：[http://blog.csdn.net/linshuhe1/article/details/51143026](http://blog.csdn.net/linshuhe1/article/details/51143026 "Github+Jekyll —— 创建个人免费博客（一）从零开始")，其实也很简单，但是存在一些问题：目录、Rss、sitemap无法自动生成。

---

> 最近看到了别人使用hexo+github实现的博客，有更多的灵活性和简约的风格，所以也试着改一下自己原本的设计。

<!--more-->


## 什么是hexo：
```hexo```是一个基于Node.js的静态博客程序，可以方便的生成静态网页托管到github或者Heroku上，类似于jekyll、Octopress、Wordpress等，使用markdown来写文章。hexo的作者是[https://github.com/tommy351/hexo](https://github.com/tommy351/hexo "@tommy351")。具有以下几点优点：

- 易用性，部署很简单，常用指令有：```hexo new```、```hexo generate```、```hexo server```、```hexo deploy```；
- 轻量级，文件少而小，自定义方便

## 相关知识：
hexo配置过程中使用到了```Github```，```Git```，```Markdown```，```Node.js```等相关操作，所以需要很多插件、widget需要自己安装配置。

## 安装准备：

1. Node.js:[https://nodejs.org/en/](https://nodejs.org/en/ "https://nodejs.org/en/") ；
2. Github桌面版（Windows）：[https://desktop.github.com/](https://desktop.github.com/ "https://desktop.github.com/")；

## 安装Github桌面版和配置
1. 双击下载好的```GitHubSetup.exe```文件，按照默认设置完成安装；
2. 登录自己的github账号；

![](http://i.imgur.com/97nOEVO.png)
3. 在github网页上创建一个以```username.github.io```命名的repositories,此时username为自己github的账号名称；

![](http://i.imgur.com/A9cQrk6.png)
4. 打开Git Shell，使用配置SSH Key使本地git项目与远程Github建立联系：```ssh -T git@github.com```；

![](http://i.imgur.com/t33BM1G.png)

## 安装Node.js
直接双击下载好的```node-v4.5.0-x64.msi```选择指定的安装路径，按照默认设置完成安装操作，安装完成后不需要对Node.js进行任何配置。为了检验是否完成安装，可以打开命令行，输入指令：```npm --version```进行版本号查询。
![](http://i.imgur.com/H21hK8M.png)

## 安装Hexo:
### 1.安装:

    mkdir hexo #创建一个项目文件
	cd hexo    #进入项目文件目录
	npm install -g hexo-cli
	npm install hexo --save
npm是Node.js中的一个工具，所以在安装Hexo之前应该先安装Node.js

### 2.部署Hexo：
在Git shell中输入：

	hexo init
记得输入之前需要确保当前命令行所处目录为所要创建工程的根目录下，因为此操作的结果就是将hexo的一些必要文件复制到当前目录下面。
![](http://i.imgur.com/8bOaggs.png)
看到上图结果之后，可以通过以下指令运行博客：

	hexo server
![](http://i.imgur.com/OyexMqN.png)
运行正常的话可以通过访问：[http://localhost:4000/](http://localhost:4000/ "http://localhost:4000/")查看运行结果：

![](http://i.imgur.com/Xy731Ig.png)

假如出现了hexo服务启动成功，但是浏览器访问localhost:4000一直不响应，那就有可能是因为你的设备上装了其他软件占用了4000端口，一般有两种办法可以解决：

- 在服务管理中将占用该端口的服务停止掉，通常安装了福昕阅读器的就会占用4000，把其对应的后台服务关掉即可；
- 切换hexo启动的默认端口，使用以下指令：

	hexo s -p 5000

此时启动端口就变成了5000，访问地址变成了localhost:5000。

### 3.安装Hexo插件：
主要目的是为了让其自动生成sitemap，Rss，部署到git等，这些是额外的插件，假如不需要使用到这些功能可以不添加：

	npm install hexo-generator-index --save
	npm install hexo-generator-archive --save
	npm install hexo-generator-category --save
	npm install hexo-generator-tag --save
	npm install hexo-server --save
	npm install hexo-deployer-git --save
	npm install hexo-deployer-heroku --save
	npm install hexo-deployer-rsync --save
	npm install hexo-deployer-openshift --save
	npm install hexo-renderer-marked@0.2 --save
	npm install hexo-renderer-stylus@0.2 --save
	npm install hexo-generator-feed@1 --save
	npm install hexo-generator-sitemap@1 --save

## 将当前工程上传到github
### 1.修改配置文件：
在当前项目的根目录下找到```_config.yml```配置文件，用编辑器打开，并找到Deployment标签处deploy节点，填写以下配置信息，```type```是指定拖过平台类型，```repository```指定了github上创建的repository仓库地址，```branch```指定了版本类型。（注：冒号后面需要加一个空格，否则会出现报错）

	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
	  type: github
	  repository: https://github.com/linshuhe1/linshuhe1.github.io.git
	  branch: master
### 2.将项目deploy到github仓库：
打开Git shell进入当前项目的根目录，依次执行指令：
	
	hexo clean
	hexo generate
	hexo deploy
一般执行最后一步的时候会出现错误如下：
![](http://i.imgur.com/68SebBI.png)
解决错误的方法是：将deploy的type改成git，然后在Git shell中执行：

	npm install hexo-deployer-git --save
执行结束后再次执行上述三个指令，正确结果应该如下：
![](http://i.imgur.com/G0wPCXU.png)
如此我们便完成了将本地的hexo工程deploy到github上的操作，访问地址：https://username.github.io/可以看到页面效果,这里以我的github为例：[https://linshuhe1.github.io/](https://linshuhe1.github.io/ "https://linshuhe1.github.io/")。


## Hexo常用指令使用：
### 创建新博文：
在Git shell中使用以下指令：

	hexo new "postName"
生成指定名称postName的文章到hexo\source_posts\postName.md，当然也可以直接到hexo\source_posts目录下，手动创建一个文件，命名时注意后缀名必须是“.md”即可。
![](http://i.imgur.com/qjVQtnG.png)
可以打开查看新建出来的.md文件的内容：

	---
	title: github pages + Hexo
	date: 2016-09-02 17:17:52
	tags:
	---
```title```是博文的标题，```date```是博文的日期，```tags```是分类标签。

更详细的内容可以参考：[https://hexo.io/docs/writing.html](https://hexo.io/docs/writing.html "Writing")

### 新建页面：
上面的步骤其实就是新建一篇博文的步骤，他们最后都是通过一个文章页面来显示的单个子页，但是我们的博客页面出来需要有博文显示页面之后，还需要有其他的页面，每个页面相当于一个分类对应顶栏菜单中的一个页签，如下图首页、下载等都是一个页面，所以可以理解为页面就相当于子页的父节点：
![](http://i.imgur.com/V33kn9P.png)
创建一个页签的操作是在Git shell中输入指令：

	hexo new page "页签名称"
上述步骤操作结果是在hexo\source目录下多出一个文件夹，而且里面还有一个index.md，这就表明了我们新建了一个页签。

### 运行博客：
使用Git shell在当前项目的根目录下执行以下指令：
	
	hexo server
更详细的内容可以参考：[http://hexo.io/docs/server.html](http://hexo.io/docs/server.html "Server")

### 生成静态站点文件：

	hexo generate
更详细的内容可以参考：[http://hexo.io/docs/generating.html](http://hexo.io/docs/generating.html "Generating")

## 发表一篇新博文
### 1.新建博文：
使用新建博文的指令：

	hexo new "github pages + Hexo"

### 2.编辑博文内容：
打开步骤1创建得到的.md文件，使用的语法是markdown，假设内容如下：

	---
	title: github pages + Hexo
	date: 2016-09-02 17:17:52
	tags: 测试
	---
	
	>测试博客

### 3.发表博文：
之前的内容中已经提到了将本地内容更新到github需要三个步骤：
	
	hexo clean
	hexo generate
	hexo deploy
其实还有快捷的指令输入方式，如下：

	hexo g == hexo generate
	hexo d == hexo deploy
	hexo s == hexo server
	hexo n == hexo new
	# 还能组合使用，如：
	hexo d -g
完成上述三个步骤，一篇新的博文就发表到github上面了。

## 使用Next主题美化界面：
安装好hexo之后，主题使用的是hexo默认自带的```landscape```主题，Next主题是iissnan设计的，使用指南其实可以直接参考Next官方网：[http://theme-next.iissnan.com/](http://theme-next.iissnan.com/ "Next")

### 1.Next主题下载：
打开Git shell，在当前项目根目下使用git从github上checkout主题的代码，输入指令：

	git clone https://github.com/iissnan/hexo-theme-next themes/next
![](http://i.imgur.com/bGoWtvn.png)	
下载完成后，在hexo\theme目录下回多出一个next文件夹，里面就是next主题所需的文件,当然我们也可以看到在theme文件目录还有一个landscape文件夹，这也就是hexo默认的主题。
![](http://i.imgur.com/rqq71ts.png)

### 2.配置主题：
之前我们配置hexo的时候，有用到```_config.yml```文件，称其为**站点配置文件**，而我们打开next主题文件夹，发现里面也有一个```_config.yml```文件，我们称这个为**主题配置文件**。在hexo中启用next主题的方式：就是打开站点配置文件，找到```theme```字段，将其值改为“next”，如下：

	# Extensions
	## Plugins: https://hexo.io/plugins/
	## Themes: https://hexo.io/themes/
	theme: next
配置完成后，在Git shell中使用```hexo server```指令启动本地博客，在浏览器中访问[http://localhost:4000](http://localhost:4000)可以看到如下结果：
![](http://i.imgur.com/ZKzamkL.png)

### 3.next的样式选择：
next的样式其实有三种：Muse、Mist和Pisces，步骤2中看到的其实是next默认的模式Muse，根据官方说明三个样式的特点如下：

- **Muse：** 默认 Scheme，这是 NexT 最初的版本，黑白主调，大量留白
- **Mist：** Muse 的紧凑版本，整洁有序的单栏外观
- **Pisces：** 双栏 Scheme，小家碧玉似的清新


切换的控制其实很简单，使用next主题配置文件中的```scheme```字段来控制，假设我们选择Mist样式（个人认为最好看的样式），操作步骤是：打开next文件夹中的```_config.yml```文件，找到```scheme```字段，将其设置为“Mist”，如下所示：

	# ------------------------------------------------------
	# Scheme Settings
	# ------------------------------------------------------
	
	# Schemes
	#scheme: Muse
	scheme: Mist
	#scheme: Pisces
重新启动博客，刷新浏览器可以看到：
![](http://i.imgur.com/6YFzhyz.png)

## 额外的优化：
### 1.设置favicon：
favicon的全称Favorites Icon，即地址栏左侧的图标：

![](http://i.imgur.com/ZNLKFAE.png)

有个在线工具可以上传自己的图片去生成指定规格的favicon.ico文件：[http://www.atool.org/ico.php](http://www.atool.org/ico.php)。打开主题配置文件```_config.yml```可以看到favicon的配置信息：

	# Put your favicon.ico into `hexo-site/source/` directory.
	favicon: /favicon.ico
根据说明，我们将图标取名为```favicon.ico```然后放到当前工程的hexo\source目录下，重启博客即可生效。

### 2.菜单栏控制：
我们看到页面顶部的菜单栏，其实是由主题配置文件中的```menu```字段控制的，例如原本的样子是这样：
![](http://i.imgur.com/iyg45Yj.png)

我们修改一下主题配置文件，如下把about页面前面的注释去掉，即让此页签处于显示状态：

	# ------------------------------------------------------
	# Menu Settings
	# ------------------------------------------------------
	
	# When running the site in a subdirectory (e.g. domain.tld/blog), remove the leading slash (/archives -> archives)
	menu:
	  home: /
	  #categories: /categories
	  about: /about
	  archives: /archives
	  tags: /tags
	  #commonweal: /404.html
重启博客可以看到效果如下：
![](http://i.imgur.com/O6Z9NvT.png)

然而，点击打开About却出现了“Cannot GET /about/”的页面错误，这是因为我们还没有about这个页面，需要使用```hexo new page "页面名称"```进行创建：

	hexo new page about
执行结果就是在hexo\source目录下面多出了一个about文件夹，里面有index.md，这就是点击About会展示的内容页面。同理，也可以创建tags页面。

### 3.语言设置：
在站点配置文件中假如如下内容，明确指定使用的语言，例如中文：

	language: zh-Hans
设置完毕后，发现菜单栏也发生了变化：
![](http://i.imgur.com/Ekxi8Tv.png)

### 4.侧栏设置：
在主题配置文件的```sidebar```字段，此处我直接设置为侧栏一直显示，而且显示在右边：

	sidebar:
	  # Sidebar Position, available value: left | right
	  position: left
	  #position: right
	
	  # Sidebar Display, available value:
	  #  - post    expand on posts automatically. Default.
	  #  - always  expand for all pages automatically
	  #  - hide    expand only when click on the sidebar toggle icon.
	  #  - remove  Totally remove sidebar including sidebar toggler.
	  #display: post
	  display: always
	  #display: hide
	  #display: remove

### 5.设置头像和作者名称：
在站点配置文件中，新加一个字段```avatar```，值就是头像的连接地址，这里我使用站内地址，将avatar.png放到本地目录hexo\source\images中；作者名称直接设置站点配置文件中```author```字段的值：

	# Site
	title: Linsh-何乐不为~
	subtitle:
	description:
	author: Linshuhe
	avatar: /images/avatar.png
	language: zh-Hans
	timezone:

## 第三方服务：

### 1.多说评论：
进入多说官网，登录后点击“我要登录”，填写相关信息，注意要记住```多说域名```这个字段填写的内容，```http://(duoshuo_shortname).duoshuo.com```，这个duoshuo_shortname将用于我们站点配置文件中的配置。步骤：在站点配置文件中新建一个```duoshuo_shortname```的字段，填写注册使用的duoshuo_shortname，例如：

	duoshuo_shortname: linshuhe1

### 2.百度统计：
用于统计阅读的次数，步骤如下：

- 登录百度统计官网：[http://tongji.baidu.com/web/welcome/login](http://tongji.baidu.com/web/welcome/login "百度统计")定位到站点的代码获取页面；
- 复制```hm.js?```后面的那串id；
- 在站点配置文件中，新增一个字段```baidu_analytics```，设置其值为上面复制的百度统计的id
- 阅读次数统计，使用LeanCloud来实现，详情查看：[https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud](https://notes.wanghao.work/2015-10-21-%E4%B8%BANexT%E4%B8%BB%E9%A2%98%E6%B7%BB%E5%8A%A0%E6%96%87%E7%AB%A0%E9%98%85%E8%AF%BB%E9%87%8F%E7%BB%9F%E8%AE%A1%E5%8A%9F%E8%83%BD.html#%E9%85%8D%E7%BD%AELeanCloud)

### 3.Swiftype搜索
使用 Swiftype 之前需要前往 Swiftype 配置一个搜索引擎。 而后编辑 站点配置文件， 新增 swiftype_key 字段，值为你的 swiftype 搜索引擎的 key。 详细的配置请参考： [http://theme-next.iissnan.com/third-party-services.html#swfitype](http://theme-next.iissnan.com/third-party-services.html#swfitype "Swiftype")

**Local Search:**添加百度/谷歌/本地 自定义站点内容搜索：

- 安装hexo-generator-search:
- 
	npm install hexo-generator-search --save
- 在站点配置文件中加入：
- 
	search:
		path: search.xml
		field: post

>最终结果可以查看我的博客：[https://linshuhe1.github.io/](https://linshuhe1.github.io/)


### 补充：
可能有人跟我一样遇到了带你麻烦，那就是主页面的文章列表中，博客内容全部显示出来而不是只显示文章一部分和 ```阅读全文》``` 按钮，这样显得首页的列表很杂乱和冗长，其实要解决这个问题很简单，只需要在我们编写markdown内容的时候，在适当的位置假如如下标签：

	<!--more-->
那么在首页显示的部分就是此标签前面的文章内容，而非全文显示：
![](http://i.imgur.com/PIN0F0K.png)
