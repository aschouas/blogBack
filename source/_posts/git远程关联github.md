---
title: Git远程关联github仓库
date: 2017-05-16 15:39:52
comments: true
tags: 技术研究
toc: true
categories: 教程
---

<blockquote class="blockquote-center">树欲静而风不止</blockquote>


## github远程关联本地git


- 首先需要在本地安装git https://git-scm.com/downloads

- 如果已经成功在本地安装git,又想在GitHub创建一个Git仓库，并且让这两个仓库进行远程同步，那就需要用到SSH Key，github拿到了你的公钥就会知道内容是你推送的,前提是你要有github账号.

<!--more-->

- 本地配置用户名和邮箱
打开[gitBash]



    git config --global user.name "你的用户名"

    git config --global user.email "你的邮箱"


- 生成ssh key


	ssh-keygen -t rsa -C "你的邮箱"

		
需要注意的是这个地方会有三次需要你输入密码，为了方便直接回车略过!
之后会在控制台看到
ssh key 被保存存到了 ../.ssh/id_rsa.pub   这个文件中，找到这个文件复制key
		
- 打开github，进入Settings
	[![](http://aschouas.com/img/recom/20170516165044.png "")](http://aschouas.com/img/recom/20170516165044.png "")	

- sshkey 添加成功之后

	测试一下 `ssh -T git@github.com`
	如果出现 `You've successfully ` 则表示配置成功

### 创建远程仓库并与本地关联

- 创建远程仓库
 
	首先是在右上角点击进入创建界面：

	[![](http://aschouas.com/img/recom/20170516182417.png "")](http://aschouas.com/img/recom/20170516182417.png "")
	
	接着输入远程仓库名:
	
	[![](http://aschouas.com/img/recom/20170516182459.png "")](http://aschouas.com/img/recom/20170516182459.png "")

	点击 Create repository 就创建好了。

- 创建本地仓库
	新建文件夹，/test
	定位到该文件下 执行 `git init`
	

- 将远程仓库和本地仓库关联起来
	先到Github上复制远程仓库的SSH地址：
	
	[![](http://aschouas.com/img/recom/20170516182554.png "")](http://aschouas.com/img/recom/20170516182554.png "")
	
	注意SSH的地址格式是这样开头的： `git@github.com`
	
	运行 git remote add origin *你复制的地址*

	如果你在创建 repository 的时候，加入了 README.md 或者 LICENSE ，那么 github 会拒绝你的 push 。你需要先执行 `git pull origin master`(远程本地仓库同步)。

	`git status` 查看当前仓库状态
	
	`git add .`  将所有文件添加到仓库(也可单个文件名)
	
	`git commit -m "message"` 提交 message--本次提交备注
	
	`git push origin master:master` 推送到远程仓库
	
	----与上一步效果相同-----执行 `git push -u origin master` 将本地仓库上传至Github的仓库并进行关联
	
	到此，已经完成关联!
	
	以后想在commit后同步到Github上，只要直接执行 `git push` 就可以了
