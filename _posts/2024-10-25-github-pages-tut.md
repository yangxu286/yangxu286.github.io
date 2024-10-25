---
title: 使用GitHub Pages搭建个人博客的快速教程
date: 2024-10-25 18:09:30 +0800
tags: [Blog, Jekyll, GitHub Pages]
categories: [Jekyll]
---

#### 第一步：安装Ruby+DevKit环境

在[Downloads](https://rubyinstaller.org/downloads/)下载安装WITH DEVKIT的最高版本。在安装过程中，使用默认设定（尤其不要取消勾选添加到环境变量）。在**Select Components**这一步，注意把所有选项都选上。

安装结束后，会有个"Run `ridk install`..."的选项，勾选它。之后出现cmd界面，选择安装第三个（MSYS2 and  MINGW development toolchain）。

这期间会进行一系列密钥更新，但我因为网络问题全部更新失败了（有梯子也一样），不过也暂时无影响。但也因此需要等待相当长的时间，请耐心等待。

安装完毕后按回车，接着重新开一个cmd或powershell，输入`ruby -v`检查是否安装成功即可。

另外也可检查`gem -v`，我此前并未安装过gem，所以正常情况下会在此过程中安装。

#### 第二步：安装Jekyll

输入`gem install jekyll bundler`，等待安装结束后输入`jekyll -v`检查。

#### 第三步：根据具体需求创建新仓库

在这里查看Jekyll的主题：[github.com/topics/jekyll-theme](https://github.com/topics/jekyll-theme)

我选择的是chirpy主题，作者提供了快速开始starter模板和直接fork主题的仓库两种方式，区别在于starter可以自动更新作者维护的内容。

不论以什么方式创建了新仓库，都需要把名字改为`<username>.github.io`（username是小写的github用户名），这样才能使用github pages功能。

之后的内容是基于starter的，因为starter可以通过workflow自动部署，而直接fork的话不可以（我还没弄懂怎么部署）。所以如果像我一样不懂的话，建议用starter。

#### 第四步：配环境

配环境的好处是可以在本地进行调试。由于github有打包次数限制，本地调试有时是必要的。

chirpy的官方手册推荐使用Dev Containers，不过我还没学会怎么把仓库克隆到它提供的容器中，所以我就只采用最简单的克隆到本地的方式，也是可行的。

首先，`git clone`我们刚创建的仓库到本地。之后使用`bundle install`构建本地环境。这个时候一般需要手动`cd`一下移到仓库的文件夹中，这样路径里有了Gemfile，才可以使用`bundle install`。

安装`wdm`时会出现问题，应该是旧版`wdm`的bug。我们改用`gem install wdm`，会安装最新版本的`wdm`，之后把Gemfile中`gem "wdm"`后面的版本号改为刚安装的这个最新版本即可。

之后，若要在本地[http://127.0.0.1:4000](http://127.0.0.1:4000/)运行网站，可以输入`bundle exec jekyll s`。这一步也可以验证刚才的本地环境构建是否成功。

#### 第五步：修改_config.yml文件

具体内容在该文件的注释中都已经写得很清楚了，有网站标题，语言，时区，联系方式等。但最必须的还是url，必须填写，不然部署会失败。

#### 第六步：使用GitHub Actions部署

1.访问仓库→Settings→Pages→Source选择GitHub Actions

2.进行任意一次`git push`以激发GitHub Actions自动的工作流。

然而，对于刚刚熟悉`git`的~~你~~本人，自然需要更傻瓜的教程：

0.先在repository→Actions里检查Workflow是否被禁用了，我的是被禁用了。若被禁用请打开。

1.首先`git add`+刚才修改的文件，如Gemfile、_config.yml。

2.然后`git commit`。如果不会使用vim，建议用`git commit -m "message"`来输入信息。

3.`git push`。若出现网络问题（443）可以尝试下面这种方法：挂梯子→电脑设置里网络和Internet→使用代理服务器→看一下端口，如127.0.0.1:10809。而后手动设置代理：

```bash
git config --global http.proxy http://127.0.0.1:10809
git config --global https.proxy http://127.0.0.1:10809
```

再尝试即可。

之后再看Actions，会出现一个Build and Deploy Workflow，名字是 你刚才commit时填写的message。等待Workflow完成后，便可以访问网站了；若出错，可以点进去查看详情。此外，这个Workflow也可以手动执行。

#### 第七步：添加新文章

将`.md`文件放到_posts文件夹下。注意开头格式，这一点很多博客的md格式都是一样的。

不过我遇到了一个问题，就是必须在date那一栏后面加上时区，如` date: 2024-10-25 18:09:30 +0800`，不然可能会因为未来时间而报错、忽略博文或者在网站上显示的时间错误。不知道是不是chirpy的刚需。

#### 关于latex支持问题

chirpy主题本身支持latex，但默认不加载。使用latex需要注意两点：

1.在需要支持latex的博文最前加上`math: true`。

2.注意语法，需要在$符号前后加上一些空格与空行，具体见官方手册：[Writing a New Post | Chirpy](https://chirpy.cotes.page/posts/write-a-new-post/#mathematics)。



