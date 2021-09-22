---
title: Hexo x GitPage 部署
date: 2021-09-22 10:15:39
tags:
    - GitPage
    - Hexo 部署
    - LF/CRLF 错误
description: Hexo x GitPage 联名 
---

> 本文只介绍如何部署到GitPage，默认Hexo本地已经存在，故略去Hexo搭建过程

简介
--

### **# GitHub Pages 是什么？**

*   [What is GitHub Pages? - GitHub Help](https://help.github.com/en/articles/what-is-github-pages)

GitHub Pages 是由 GitHub 官方提供的一种免费的静态站点托管服务，让我们可以在 GitHub 仓库里托管和发布自己的静态网站页面。

### **# Hexo 是什么？**

*   官网：[hexo.io](https://hexo.io/zh-cn/)

Hexo 是一个快速、简洁且高效的静态博客框架，它基于 Node.js 运行，可以将我们撰写的 Markdown 文档解析渲染成静态的 HTML 网页。

### **# Hexo + GitHub 文章发布原理**

在本地撰写 Markdown 格式文章后，通过 Hexo 解析文档，渲染生成具有主题样式的 HTML 静态网页，再推送到 GitHub 上完成博文的发布。

![](https://pic3.zhimg.com/v2-a193a47cf70fe6ecf156e5f3d34920ea_r.jpg)

## 部署

### 连接Github

设置`Github` 信息  
设置 git 配置的用户名邮箱 (将`lingzhexi` / `lingzhexi@gmail.com` 分别替换成自己的用户名和邮箱)

```
git config --global user.name "lingzhexi"
git config --global user.email "lingzhexi@gmail.com"
```

#### 创建 SSH 密匙

输入 `ssh-keygen -t rsa -C "GitHub 邮箱"`，然后一路回车。
![img.png](https://gitee.com/lingzhexi/blogImage/raw/master/2021/09/22/202109221047531.png)

![image-20210922104651057](https://gitee.com/lingzhexi/blogImage/raw/master/2021/09/22/202109221047883.png) 

#### 添加密匙

进入 [C:\Users \ 用户名 \.ssh] 目录（要勾选显示 “隐藏的项目”），用记事本打开公钥 id_rsa.pub 文件并复制里面的内容。

登陆 GitHub ，进入 Settings 页面，选择左边栏的 SSH and GPG keys，点击 New SSH key。

Title 随便取个名字，粘贴复制的 id_rsa.pub 内容到 Key 中，点击 Add SSH key 完成添加。
![img_2.png](img_2.png)

#### 验证连接

打开 Git Bash，输入 `ssh -T git@github.com` 出现 “Are you sure……”，输入 yes 回车确认。

![image-20210922104829350](https://gitee.com/lingzhexi/blogImage/raw/master/2021/09/22/202109221048625.png)

显示 “Hi xxx! You've successfully……” 即连接成功。

### 创建 Github Pages 仓库

---------------------

GitHub 主页右上角加号 -> New repository：

*   Repository name 中输入 `用户名.github.io`
*   勾选 “Initialize this repository with a README”
*   Description 选填

填好后点击 Create repository 创建。

![](https://pic2.zhimg.com/v2-67a8165154f4c5f4a6333e76e78ed815_r.jpg)

创建后默认自动启用 HTTPS，博客地址为：`https://用户名.github.io`

### 部署 Hexo 到 GitHub Pages

-------------------------

本地博客测试成功后，就是上传到 GitHub 进行部署，使其能够在网络上访问。

首先**安装 hexo-deployer-git**：

```
npm install hexo-deployer-git --save
```

然后**修改 _config.yml** 文件末尾的 Deployment 部分，修改成如下：

```yml
deploy:
  type: git
  repository: git@github.com:lingzhexi/lingzhexi.github.io.git
  branch: master
```

完成后运行 `hexo d` 将网站上传部署到 GitHub Pages。

完成！这时访问我们的 GitHub 域名 `https://lingzhexi.github.io` 就可以看到 Hexo 网站了。

## 问题汇总

### 1. git提示：warning: LF will be replaced by CRLF

在部署的提交静态文件到Github上时：

> hexo d 

![image-20210922105234518](https://gitee.com/lingzhexi/blogImage/raw/master/2021/09/22/202109221052526.png)

#### **分析问题**

​	格式化与多余的空白字符，特别是在跨平台情况下，有时候是一个令人发指的问题。由于编辑器的不同或者文件行尾的换行符在 Windows 下被替换了，一些细微的空格变化会不经意地混入提交，造成麻烦。虽然这是小问题，但它会极大地扰乱跨平台协作。
 其实，这是因为在文本处理中，[CR](https://link.jianshu.com?t=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FCarriage_return)（**C**arriage**R**eturn），[LF](https://link.jianshu.com?t=http%3A%2F%2Fen.wikipedia.org%2Fwiki%2FLine_feed)（**L**ine**F**eed），CR/LF是不同操作系统上使用的换行符，具体如下：

#### 换行符‘\n’和回车符‘\r’

- 回车符就是回到一行的开头，用符号r表示，十进制ASCII代码是13，十六进制代码为0x0D，回车（return）；
- 换行符就是另起一行，用n符号表示，ASCII代码是10，十六制为0x0A， 换行（newline）。

所以我们平时编写文件的回车符应该确切来说叫做回车换行符。

#### 影响

- 一个直接后果是，Unix/Mac系统下的文件在Windows里打开的话，所有文字会变成一行；
- 而Windows里的文件在Unix/Mac下打开的话，在每行的结尾可能会多出一个^M符号。
- Linux保存的文件在windows上用记事本看的话会出现黑点。

这些问题都可以通过一定方式进行转换统一，例如，在linux下，命令unix2dos 是把linux文件格式转换成windows文件格式，命令dos2unix 是把windows格式转换成linux文件格式。

#### 解决问题

    Git 可以在你提交时自动地把回车（CR）和换行（LF）转换成换行（LF），而在检出代码时把换行（LF）转换成回车（CR）和换行（LF）。 你可以用`git config --global core.autocrlf true` 来打开此项功能。 如果是在 Windows 系统上，把它设置成 true，这样在检出代码时，换行会被转换成回车和换行：



```shell
#提交时转换为LF，检出时转换为CRLF
git config --global core.autocrlf true
```

问题解决

![image-20210922105645713](https://gitee.com/lingzhexi/blogImage/raw/master/2021/09/22/202109221056943.png)

参考：
    - [Hexo/GitPage 部署](https://hexo.bootcss.com/docs/github-pages.html)
    - [使用 Hexo+GitHub 搭建个人免费博客教程（小白向）](https://zhuanlan.zhihu.com/p/60578464)
    - [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
    - [解决git LF/CRLF](https://www.jianshu.com/p/450cd21b36a4)
