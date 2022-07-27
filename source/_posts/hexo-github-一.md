---
title: hexo+github
date: 2022-04-08 21:57:23
tags:
---

### hexo和github连接免费搭建个人博客网站(小白)

一  先下载**node**和**git**，这个我都已经下载了，所以这一步详细步骤省略，等之后在笔记本上尝试时再添加。下载完成后可以通过 -v /-version查看node、npm和git版本确定是否下载成功

二  注册**github**账号，并创建仓库，设置ssh连接

- （1）点击头像，选择new reposity，新建一个名为：**用户名.github.io**的仓库，比如说我的用户名是leoamang，那么我需要建立一个名为**leoamang.github.io**的仓库。

- （2）配置**SSH key**：使用ssh key解决本地和服务器的连接问题。

a：在桌面上（我做的时候并没有在桌面上，就是任意的文件夹，不知道是否只有桌面才成功，待验证）使用git base here进行命令行，输入

> git config --global user.name "username"
>
> (例如：git config --global user.name "leoamang")，
>
> 输入git config --global user.email "youremail"
>
> (例如git config --global user.email "***@qq.com"）。

b：创建SSH，输入后一路回车就行

> ssh-keygen -t rsa -C "youremail"（C大写）

​       C：在路径C:\Users\DELL\.ssh的id_rsa.pub可以看到生成的ssh密钥。打开并复制。

- （3）在github上设置密钥：点击头像→settings→找到ssh and GPG keys→new ssh key→粘贴复制的ssh密钥，随意起个名字，并保存。检查设置是否成功

  > ssh -T git@github.com

若出现Hi username！  You've successfully authenticated, but GitHub does not provide shell access.

三 下载hexo

**在自己想要的位置创建一个文件夹，名叫hexo，在这个文件夹下打开git bash here**

使用npm全局安装hexo框架和deployer：

>npm install -g hexo-cli 
>
>npm install hexo-deployer-git --save

若安装速度慢，或者不成功，更换npm镜像源（我第一次没成功，更换镜像源后成功）

> npm config set registry https://registry.npm.taobao.org。

安装完成后，初始化博客，会在hexo文件夹下面创建一个blog文件，或者直接hexo init不会创建blog文件夹

> npm init blog

检测是否可以，新建一个新的博客，然后执行g和s，打开浏览器访问4000端口查看。

>hexo n “test“ === hexo new ”test“
>
>hexo g
>
>hexo s 

ctrl+c可以退出hexo服务器

四 推送博客

在blog文件下面找到config.yml配置及文件，修改最后的内容为（**注意冒号之后有空格，不然会提示出错**）：

>type: git
>repo: github下的仓库的完整路径
>branch: master

上述都成功后，可以在blog/source/__post下面创建md文件，写完之后可以通过

> hexo g -d 或者hexo d推送

常用命令：

hexo n "博客名" === hexo new "博客名"

hexo g === hexo generate 生成静态页面至public目录

hexo s === hexo server 开启预览访问端口（ctrl+c退出）

hexo d === hexo deploy 部署到github

hexo s -g 生成并本地预览

hexo g -d 生成并上传 === hexo d -g

