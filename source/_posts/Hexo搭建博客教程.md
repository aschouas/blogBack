---
title: Hexo搭建博客教程
date: 2017-05-17 09:53:18
tags: 技术研究
toc: true
categories: 教程
---
<blockquote class="blockquote-center">喜欢吃水蜜桃</blockquote>

*以前用的博客都丢失了，真的好气啊，所以一直有想法搭建一个自己的博客。所以利用hexo+github搭建了本地博客,用于分享一些心得，记录一些有意思的事情。在这个过程中，遇到了很多问题，在此记录一些配置和遇到的问题。*

## Hexo搭建博客教程

### 准备
- 下载 node.js 并安装,默认安装 npm

- 下载安装 git，并与 github 关联,详情见上篇文章

- 下载安装 hexo，打开 cmd 运行 `npm install -g hexo `

<!--more-->

### 本地搭建Hexo静态博客
- 本地新建一个文件夹,比如 MyBlog
 
- 在MyBlog文件夹中右击运行 `Git Bash Here` , 输入 `hexo init` ---生成hexo模板,需要科学上网
 
- 生成完模板之后，输入 `npm install`,这里值得注意的是，在 windows 下,会出现`npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@^1.0.0 ` ， `fsevents@^1.0.0` 是 mac osx 系统的。 ---忽略
 
- 最后运行 `hexo server` ---然后在浏览器输入 `http://localhost:4000 ` 就可以看到博客基本搭好了

### 将博客与github关联
- 在 Github 上创建名字为 XXX.github.io 的项目，XXX为自己的 github 用户名。
	[![](http://aschouas.com/img/recom/20170517104221.png "")](http://aschouas.com/img/recom/20170517104221.png "")

- 打开本机文件夹 MyBlog 内的 _config.yml 文件

```bash    
deploy:
type: git
repository: ssh://git@github.com/aschouas/aschouas.github.io
branch: master
```
		

- 运行 `npm install hexo-deployer-git –save`

- 运行 `hexo g/genrnate`（本地生成静态文件）

- 运行 `hexo d/deloye` (将本地静态文件推送至 github)

到此在浏览器输入 `aschouas.github.io` 便可访问自己的博客啦

### 绑定域名
一不小心申请了一个域名，于是我将 github 博客绑定到自己域名

- 域名解析设置 

[![](http://aschouas.com/img/recom/微信截图_20170517110026.png "")](http://aschouas.com/img/recom/微信截图_20170517110026.png "")

- 添加 CNAME文件

	配置完域名解析后，进入博客目录，在 source 目录下新建 CNAME 文件(没有任何后缀)，写入域名，如：aschouas.com

## 更新博客内容 

### 更新文章

- 在 MyBolg 文件夹下运行 `hexo new "you need blog name"`,会在 source/_posts 文件夹下生成一个 *.md 文件

- 编辑该文件 （遵守 MarkDown 规则）, 可以使用 MarkDown 编辑器  

- 修改起始字段
	
	-   tilte 文章的标题 	
	
	- 	date 创建日期 （文件的创建日期 ）
	
	- 	updated 修改日期 （ 文件的修改日期）
	
	- 	comments 是否开启评论 true
	
	- 	tags 标签
	
	- 	categories 分类
	
	- 	permalink url中的名字（文件名） 	

- 创建新的文章的时候,起始字段比较少，每次都要加，很麻烦
 
	- 只需要在 scaffolds/post.md 文件夹中添加你需要的字段即可

- 编写正文 (MarkDown)

- 执行 `hexo clean` (清除本地编译文件 /Public)
- 执行 `hexo g ` (生成本地静态文件 /Public)
- 执行 `hexo d ` (将本地静态文件推送至 github)

## MarkDwon语法

### 斜体和粗体
- 使用 * 和 ** 表示斜体和粗体，格式如下：
 
	`*斜体*， **粗体** `

渲染效果： 这是 *斜体*，这是 **粗体** 。

### 分级标题
- 使用 === 表示一级标题，使用 --- 表示二级标题，格式如下：
```bash 
这是一个一级标题
============================
这是一个二级标题
--------------------------------------------------
### 这是一个三级标题 
```

- 你也可以选择在行首加#号表示不同级别的标题 (H1-H6)，格式如下：

```bash
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

### 分割线
在单独的一行使用 *** 或者 --- 表示分割线

### 删除线
使用 ~~ 表示删除线.

### 超链接
- 插入文字超链接的格式如下 ：

```bash
[链接文字](链接地址 "链接标题")
```

- 插入图片超链接的格式如下：
 
```bash
![图片说明](图片链接 "图片标题")
```

- 引用视频则直接插入iframe代码：

```bash 
<script src="/js/youtube-autoresizer.js"></script>
<iframe width="640" height="360" 
src="https://www.youtube.com/embed/HfElOZSEqn4"
frameborder="0" allowfullscreen></iframe>
```	

<script src="/js/youtube-autoresizer.js"></script>
<iframe width="640" height="360" src="https://www.youtube.com/embed/HfElOZSEqn4" frameborder="0" allowfullscreen></iframe>

## 更换主题

- 我们首先要 [获取主题](https://www.zhihu.com/question/24422335 "获取主题")

- 进入 themes 目录下：
	- 下载主题
	 
		`git clone https://github.com/iissnan/hexo-theme-next.git（主题的地址）`
	
	- 打开__config.yml文件，将themes修改为next（下载到的主题文件夹的名字）
	
	- hexo g
	 
	- hexo d 

如果需要[个性化配置](http://theme-next.iissnan.com/ "个性化配置")  

## 报错解决
- Deployer not found: git
 
	当编辑__config.yml文件，将type: git设置完成后，运行hexo g 报错：git not found
	解决方案：可以在MyBlog目录下运行: npm install hexo-deployer-git –save。  

- permission denied

	当执行: hexo deploy 报错时，把__config.yml中的github连接形式从ssh改成http。
	
- 当在themes目录下载主题时，报错。

	将该目录只读属性取消。

- genrnate 报错

	检查_config.yml配置中，键值对冒号后面是否已经预留了一个半角空格。

- ERROR Plugin load failed: hexo-generator-feed


		npm install hexo-generator-feed
		npm install hexo-generator-feed --save

   	
- fatal: The remote end hung up unexpectedly


	
		git config https.postBuffer 524288000
		git config http.postBuffer 524288000
		git config ssh.postBuffer 524288000


- hero d推送的内容有问题

首先检查下.deploy_git文件夹下的.git文件是否存在，此.git文件指定了hexo d时推送public文件夹，而不是所有的内容。如果此.git文件不存在，则会出现推送内容错误。
用npm install hexo-deployer-git –save生成的.deploy_git不包含.git文件，因此正确的做法是.deploy_git文件夹也需要备份，然后再用npm install hexo-deployer-git –save更新一下其内容即可。	

## 异地同步博客内容


		node_modules
		public
		scaffolds
		source
		themes
		_config_yml
		db.json
		package.json
		.deploy_git


  以上为利用hexo生成的博客全部内容，那么当我们执行hexo d时，正真被推送到github上的又有哪些内容呢？

　　我们可以看下github上的tengzhangchao.github.io项目，发现里面只有Public目录下的内容。也就是说，我们博客上呈现的内容，其实就是public下的文件内容。那么这个Pulic目录是怎么生成的呢？

　　一开始hexo init的时候是没有public目录的，而当我们运行hexo g命令时，public目录被生成了，换句话说hexo g命令就是用来生成博客文件的（会根据_config.yml，source目录文件以及themes目录下文件生成）。同样当我们运行hexo clean命令时，public目录被删除了。

　　好了，既然我们知道了决定博客显示内容的只有一个Public目录，而public目录又是可以动态生成的，那么其实我们只要在不同电脑上同步可以生成Public目录的文件即可。

- 以下文件以及目录是必须要同步的：


		source
		themes
		_config.yml
		db.json
		package.json
		.deploy_git


同步的方式有很多种，可以每次更新后都备份到一个地址。我采用github去备份，也就是新建一个项目用来存放以上文件，每次更新后推送到github上，用作备份同步。

　　同步完必须的文件后，怎么再其他电脑上也可以更新博客呢？

　　前提假设我们现在配置了一台新电脑，里面没有安装任何有关博客的东西，那么我们开始吧：

- 下载node.js并安装（官网下载安装），默认会安装npm。

- 下载安装git（官网下载安装）

- 下载安装hexo。方法：打开cmd 运行npm install -g hexo（要科学上网）

- 新建一个文件夹，如MyBlog

-进入该文件夹内，右击运行git，输入：hexo init（生成hexo模板，要科学上网)

我们重复了一开始搭建博客的步骤，重新生成了一个新的模板，这个模板中包含了hexo生成的一些文件。

- git clone 我们备份的项目，生成一个文件夹，如：MyBlogData

- 将MyBlog里面的node_modules、scaffolds文件夹复制到MyBlogData里面。

做完这些，从表面上看，两台电脑上MyBlogData目录下的文件应该都是一样的了。那么我们运行hexo g
hexo d试试，如果会报错，则往下看。

- 这是因为.deploy_git没有同步，在MyBlogData目录内运行:npm install hexo-deployer-git –save后再次推送即可

总结流程：当我们每次更新MyBlog内容后，先利用hexo将public推送到github，然后再将其余必须同步的文件利用git推送到github。