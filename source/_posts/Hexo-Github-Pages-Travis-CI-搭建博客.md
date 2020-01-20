---
title: Hexo + GitHub Pages + Travis CI搭建博客
date: 2020-01-18 18:49:58
tags:
---
> 早就有想要写Blog做些积累的心了，今年终于抄起家伙动手了。作为第一篇有实际意义的博客，就来介绍一下这个博客的搭建过程，以及期间遇到的问题吧。

### 目标
通过username.github.io访问，将代码Push到GitHub后自动构建并部署博客。

### 博客架构 & 选择原因
经过了简单的搜索和对比，最后选择了用Markdown编写，Hexo + GitHub Pages + TravisCI 这种模式。

* Markdown - 作为博客编写的主要使用语言。简单易用。
* Hexo - 用来生成静态HTML。简单易用，网上有许多主题供选择。
* GItHub Pages - 提供博客服务。省去了搭建服务的工作，但是也有一个问题就是国内的访问不太友好。
* Travi CI - 提供构建部署服务。Hexo官方推荐~

### 搭建步骤
#### 安装Hexo的准备
使用以下命令查看Node.js版本
```
node -v
```
使用以下命令查看Git版本
```
git version
```
如果没有安装以下工具请先安装
* [Node.js](http://nodejs.org/) (Node.js 版本需不低于 8.10，建议使用 Node.js 10.0 及以上版本)
* [Git](http://git-scm.com/)

#### 安装Hexo
确保安装前的准备已经完成，可以使用以下命令安装Hexo
```
npm install -g hexo-cli
```
安装完成后可以使用以下命令查看Hexo版本
```
hexo -v
```

#### Hexo使用
> Hexo的使用可以参照[官方文档](https://hexo.io/docs/index.html)，这里只介绍必要的功能。

首先创建Hexo项目。这里可以使用博客地址作为项目名。
```
hexo init "username.github.io"
```
安装好后，我们可以使用以下命令来创建新的Blog。这里以"test"为例。
```
hexo new "test"
```
Hexo会帮你在source/_posts文件夹下创建相应的Markdown文件test.md。随后就可以在test.md里写Blog的正文了。

随后，可以使用以下命令开启本地服务。
```
hexo server
```
开启成功后控制台会输出本地服务地址：[http://localhost:4000](http://localhost:4000)。访问该地址就能进行本地预览。

#### 创建博客主仓库
创建一个"username.github.io"作为仓库名的仓库，username为你的GitHub用户名。README，.ignore什么的都可以不用选。要注意的是一定要使用"username.github.io"来作为仓库名，之后就可以用"username.github.io"来访问你的Blog。

创建好后cd到Blog项目文件夹

##### 初始化git
```
git init
```
##### 创建hexo分支
```
git checkout -b hexo
git add .
git commit -am "initial commit"
```
这里把博客工程放入hexo分支是因为GitHub Pages规定，以username.github.io为名字的仓库，在使用GitHub Pages时只能选择master分支作为网页代码的source，也就是说username.github.io只能访问master分支上的网页代码。所以，只能把静态网页代码存放在master上，而源代码则必须放在除master外的其他分支上。

##### 设置博客主仓库的remote
设置博客主仓库的remote为你的username.github.io仓库地址。（一定要用ssh的地址）
```
git remote set-url origin git@github.com:username/username.github.io.git
```

#### Hexo主题
##### Fork Hexo主题
在GitHub上fork一份你喜欢的主题，我这里选择了使用最为流行的主题之一[next](https://github.com/theme-next/hexo-theme-next)。

选择Fork是因为之后要用到git的submodule来管理主题，这样我们就可以把主题的修改推送到GitHub上。当然，直接选择把主题下载到本地，直接添加到博客主仓库里也可以，但是在之后主题官方有更新时要同步就比较麻烦了。

##### 安装Hexo主题
cd到博客主仓库文件夹，输入以下命令，将仓库地址换成你Fork来的主题仓库地址。
```
git clone "https://github.com/username/hexo-theme-next" themes/next
```
cd到themes/next文件夹，替换主题仓库的remote。
```
cd themes/next
git remote set-url origin git@github.com:username/hexo-theme-next.git
```
这里整这么复杂其实是因为我访问GitHub速度太慢，所以终端挂了HTTP的代理，如果用ssh的方式就没法使用代理了。

clone完成后再切回ssh，这样就能push了。

*_config.yml*是主题的配置文件，可以根据自己的需要自行修改。

##### 添加主题submodule
cd回到博客主仓库，再添加submodule。
```
git submodule add https://github.com/username/hexo-theme-next.git themes/next
```
这里要注意，添加submodule时一定要用HTTPS的方式，因为之后Travis CI使用时如果用ssh的方式会遇到无权限访问的问题。

#### Travis CI
##### 配置
这里可以参照[Hexo官方文档 - 部署 - GitHub Pages](https://hexo.io/docs/github-pages)里的教程，这里转述一下。
1. 将 [Travis CI](https://github.com/marketplace/travis-ci) 添加到你的 GitHub 账户中。
2. 前往 GitHub 的 [Applications Settings](https://github.com/settings/installations)，配置 Travis CI 权限，使其能够访问你的 repository。
3. 你应该会被重定向到 Travis CI 的页面。如果没有，请 手动前往。
在浏览器新建一个标签页，前往 GitHub 新建 [Personal Access Token](https://github.com/settings/tokens)，只勾选 repo 的权限并生成一个新的 Token。Token 生成后请复制并保存好。
4. 回到 Travis CI，前往你的 repository 的设置页面，在 Environment Variables 下新建一个环境变量，Name 为 GH_TOKEN，Value 为刚才你在 GitHub 生成的 Token。确保 DISPLAY VALUE IN BUILD LOG 保持 不被勾选 避免你的 Token 泄漏。点击 Add 保存。

##### 创建 .travis.yml
在Blog项目文件夹下创建.travis.yml文件，并将以下内容复制进去。
```yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - hexo # build hexo branch only
script:
  - git submodule init # 初始化submodule，不然主题就不能用了。
  - hexo generate # generate static files
deploy:
  provider: pages
  skip_cleanup: true
  github_token: $GH_TOKEN
  keep_history: true
  target_branch: master
  on:
    branch: hexo
  local-dir: public
```
修改完成后，提交commit，然后
```
git push origin hexo
```
不出意外，一会儿Travis CI就会帮你构建好静态网页并推送到你博客仓库的master分支上了。
期间想查看构建的进度或者报错信息也可以打开Travis CI的管理页查看。

之后有新的Blog或者更新，就把代码推送到hexo分支，Travis CI就会帮你构建并发布。

### 总结一下搭建的过程
* 安装Hexo。
* 创建Hexo项目，添加Blog。
* 新建username.github.com仓库，将Hexo项目的remote设为它，并把源代码放在hexo分支，而不是master分支里。
* 选择GitHub上的Hexo主题，将主题设置为博客仓库的submodule。
* 配置Travis CI。

### 遇到的问题
#### Hexo官方教程的坑
Hexo官方教程是以username.github.com这种方式来教学，最后却让我选择Travis CI默认生成的gh-pages分支作为GitHub Pages的source，这违背了GitHub Pages的规则。
在经过了一系列的搜索，找到了Travis CI的[GitHub Pages Provider](https://docs.travis-ci.com/user/deployment/pages/)的官方文档。
```
target_branch: Branch to (force, see: keep_history) push local_dir contents to, defaults to gh-pages.
```
通过更改:
```yml
branches:
  only: hexo
deploy:
  target_branch: master
  on:
    branch: hexo
```
来达到在hexo分支构建，并将产物推送到master分支的效果。

#### 主题和submodule的问题
##### 主题的存放问题
由于修改了主题且使用了Travis CI，因此肯定要把主题推送到Travis CI能访问的地方，这里刚开始想推到博客主仓库里，但想想还是不妥，之后要是主题有更新就不方便同步了。所以，最后采用了fork + submodule的形式。

##### NOLAYOUT
最开始使用submodule时，由于经验不足，以为只要在博客主仓库添加进去就好，这导致了Travis CI上hexo一直获取不到主题，报"NOLAYOUT"错误。最后发现是因为Travis CI上的主题submodule没有init导致，遂在*.travis.yml*里的scripts里，```hexo generate```前，添加了```git init submodule```。

##### submodule 无权限访问
起初submodule的URL添加到博客主仓库时用的是ssh的方式，导致Travis CI无权限拉取，最后换成了HTTPS的方式解决了。

### 参考
[Hexo 官方文档](https://hexo.io/docs/)
[Travis CI GitHub Pages Provider](https://docs.travis-ci.com/user/deployment/pages/)
[这个Hexo教程还不错](https://www.youtube.com/watch?v=Kt7u5kr_P5o&list=PLLAZ4kZ9dFpOMJR6D25ishrSedvsguVSm)