---
title: 使用hugo和docker打造静态博客解决方案
tags: [hugo,docker]
---

## 前言

今年是转型的一年，告别了游戏行业，开始使用golang做起了web应用。golang用到现在三个多月了，整体体验下来还是很不错的，接近C的性能、天生拥有的高并发能力、简洁的语法以及相对C/C++而言简化的指针，使得这门语言在开发效率和运行效率两方面都有着不错的表现。相对于C语言而言，golang有着接近于python的快速开发和快速迭代的能力，而相对于python来说，golang有着更好的性能和更强大的并发能力。相信随着开源社区的不断壮大，golang会慢慢的流行起来。

[hugo](http://gohugo.io/)是一个静态博客的生成工具，和[hexo](http://hexo.io/)类似。因为hugo是用golang写的，在github上面有着一万多的star数量，所以我建立独立博客的时候选择了hugo而不是hexo来作为静态博客的engine。

[docker](https://www.docker.com/)是PaaS提供商dotCloud开源的一个基于LXC的高级容器引擎，源代码托管在Github上,基于go语言。这个概念最近比较火，所以我就抽空学习了一下docker并用它作为我的个人博客[blog.andyli.me](http://blog.andyli.me)的部署方案。大致介绍下部署的方案，我一共用到了两个docker容器，一个容器跑nginx，另一个跑hugo生成静态页面，两个docker容器跑在一个`cent os`的vps上面。

## 一、hugo的安装

hugo是用golang写的，支持多平台。

可直接参考[hugo中文文档](http://www.gohugo.org/)。

### 1.Mac OS

mac上可以使用[HomeBrew](http://brew.sh/)安装hugo，非常简单， 只需要执行下面这句命令即可。

```
brew update && brew install hugo
```

### 2.Linux

linux安装也比较简单，因为hugo的环境就是一个二进制包，没有依赖，所以直接下载下来，放到`usr/local/bin`中就可以了

1.去网站[https://github.com/spf13/hugo/releases](https://github.com/spf13/hugo/releases)下载对应的包，比如hugo_x.xx_linux-64bit.tgz。

2.解压缩后将二进制文件改名为`hugo`

3.将这个二进制文件移动到`usr/local/bin`目录中

### 3.Windows

hugo在windows下面是一个.exe文件，也是没有依赖的。

1.去网站[https://github.com/spf13/hugo/releases](https://github.com/spf13/hugo/releases)下载对应的包，比如hugo_x.xx_linux-64bit.tgz。

2.解压缩后得到`hugo.exe`文件

3.在`C:\`中创建文件夹`C:\hugo`，将`hugo.exe`移动到`C:\hugo`中

4.把目录`C:\hugo`添加到环境变量`PATH`中

### 4.源码安装

1.安装[git](https://git-scm.com/)和[go ](https://golang.org/)1.5+

2.`export GOPATH=$HOME/go`

3.`go get -v github.com/spf13/hugo`

4.将目录`$HOME/go/bin`添加到环境变量`PATH`中

### 5.检查安装是否成功

在命令行下执行hugo命令，如果得到类似下面结果，则说明你已经成功安装了Hugo：

```
$ hugo version
Hugo Static Site Generator v0.15-DEV BuildDate: 2015-09-20T23:53:39+08:00
```

## 二、hugo的使用

hugo安装好之后，我们看看怎么来快速的使用它。

### 1.创建站点
使用下面命令在当前目录下创建一个站点`bookshelf`

```
$ hugo new site bookshelf
```

然后进入创建好的目录`bookshelf `中，执行一下`tree -a`看到目录结构是这样的：

```
.
|-- archetypes
|-- config.toml
|-- content
|-- data
|-- layouts
`-- static

5 directories, 1 file
```
### 2.新增博客内容

执行下面命令创建一篇博客：

```
$ hugo new post/good-to-great.md
```

创建成功会提示：

```
/Users/shekhargulati/bookshelf/content/post/good-to-great.md created
```

创建好的文件位于content中，congtent目录结构如下:

```
content
`-- post
    `-- good-to-great.md

1 directory, 1 file
```

### 3.运行博客

运行如下命令来启动服务：

```
$ hugo server
```

启动成功会显示如下信息：

```
0 of 1 draft rendered
0 future content
0 pages created
0 paginator pages created
0 tags created
0 categories created
in 9 ms
Watching for changes in /Users/shekhargulati/bookshelf/{data,content,layouts,static}
Serving pages from memory
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

打开站点[http://localhost:1313/](http://localhost:1313/)访问你的博客。

### 4.选择皮肤

去站点[http://themes.gohugo.io/](http://themes.gohugo.io/)选择适合你的皮肤。

## 三、使用Docker部署

我使用了两个镜像来部署，一个镜像部署博客生成静态文件，另一个镜像运行nginx。

### 1.部署博客


首先看一下静态文件镜像的dockerfile：

```
FROM debian:jessie

RUN apt-get update && apt-get install --no-install-recommends -y \
    ca-certificates \
    curl \
    mercurial \
    git-core

RUN curl -s https://storage.googleapis.com/golang/go1.6.linux-amd64.tar.gz | tar -v -C /usr/local -xz

ENV GOPATH /go
ENV GOROOT /usr/local/go
ENV PATH $PATH:/usr/local/go/bin:/go/bin

RUN apt-get update && apt-get install --no-install-recommends -y bzr

RUN go get github.com/spf13/hugo

RUN git clone https://github.com/andyidea/blog.andyli.me.git /blog

RUN cd /blog && git submodule update --init --recursive

WORKDIR /blog

VOLUME ["/var/www/blog"]

CMD ["sh", "run.sh"]
```

dockerfile写好之后，我们就可以创建镜像了。

把dockerfile放到服务器上，然后运行`docker build`

```
docker build -t blog .
```

执行完后，就创建了名字叫blog的镜像，接下来我们根据生成的镜像来生成容器：

```
docker run --name blog blog
```

这个命令的意思是，根据镜像blog生成一个名字叫做blog的容器。使用`docker ps`就可以看到正在运行的容器了。

运行到这一步说明静态的博客文件已经生成，下一步让nginx作为静态博客的server。

### 2.部署nginx

Nginx的dockerfile：

```
FROM nginx

ADD nginx/ /etc/nginx
```

dockerfile很简单，ADD这步操作是为了把config文件放到nginx中，具体可参考demo[https://github.com/andyidea/dockerfile/tree/master/nginx](https://github.com/andyidea/dockerfile/tree/master/nginx)

同样的，我们把dockerfile放到服务器上并执行

```
docker build -t nginx .
```

镜像生成好后，生成容器

```
docker run -p 80:80 -d --name nginx --volumes-from blog nginx
```
这个命令的意思是，使用镜像nginx生成一个容器，将宿主机80端口映射到docker容器中的80端口，-d是运行在后台，--volumes-from 是使用blog容器的卷，也就是blog dockerfile中的`VOLUME ["/var/www/blog"]`命令。

至此，博客的部署完毕。

### 3.更新博客

使用下面的命令来更新博客：

```
docker restart blog
```
