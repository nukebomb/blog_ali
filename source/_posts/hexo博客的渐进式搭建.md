---
title: Hexo 博客的渐进式搭建
tags: hexo
---
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，生成静态网页,因此你不需要配置复杂的后端环境，数据库环境，只需要用服务器托管 Hexo 编译出来的静态页面文件就可以使用。
Hexo 搭建博客的文章很多，大部分都是利用 githubpage 来托管。这种方法很简单，也不需要你拥有自己的服务器或者域名。但是因为依赖于 github 的服务器，所以访问速度其实并不快。因此本文想通过三部分循序渐进地搭建一个 Hexo 博客在自己的服务器上，并通过 git 自动化部署。开始之前阐述下环境：

- 本地
  - win10
  - git
  - nodejs
  - vscode
  - Xshell
- 服务器
  - Ubuntu16.04
  - git
  - Nginx
  
### 搭建本地环境
首先我们在本地搭建一个 Hexo 环境。因为 Hexo 依赖于 Nodejs，我们首先要安装 Nodejs。在 windows 下安装 nodejs,git,vscode,xshell 还是很简单的，这里就略过安装过程，默认已经装好了上面的环境。
nodejs 安装成功后，会有一个 npm 包管理器，打开一个命令行，执行
```
npm install hexo -g
```

完成后输入
```
hexo -v
```

没有报错即为安装成功。
然后选择一个你博客文章想存放的路径，在选择的路径下开起一个命令行，生成一个博客站点
```
hexo init blog
```
完成后会在选择的路径下生成一个名为 blog 的文件夹。进入文件夹可以看到项目结构。
```
cd blog\
code ./
```
![1](http://pnj2tr0xr.bkt.clouddn.com/1.png)


可以看到项目是一个典型的 node 项目，有四个文件夹，其中

  - node_modules 为项目依赖
  - scaffolds 为博客页面模板
  - source 文章和草稿文件夹
  - themes 主题文件夹
展开 themes 文件夹可以看到里面有个 landscape 文件夹，这是 hexo 默认自带的主题。source\_posts里面有个 hello-world.md 文件，这是自带的一篇博客。执行
```
hexo generate
```

![2](http://pnj2tr0xr.bkt.clouddn.com/2.png)

可以看到项目里面多了一个public文件夹

里面就是 Hexo 编译好的静态页面文件,我们只需要用任何一种服务器甚至是网络服务(比如阿里云 oss)托管生成的静态页面就可以将自己以 markdown 方式写的文档以网页方式呈现。而网页的样式通过themes文件夹里面的样式文件夹控制。

hexo 提供了一个工具让我们可以在本地预览生成的静态页面，首先我们安装这一工具
```
npm install hexo-server
```
然后就可以运行
```
hexo server
```
启动预览
![3](http://pnj2tr0xr.bkt.clouddn.com/3.png)

打开浏览器，在http://localhost:4000/就可以看到生成页面的效果
![4](http://pnj2tr0xr.bkt.clouddn.com/4.png)

至此我们已经可以预览 hexo 编译出的效果，可以开始写自己的文章了。执行

hexo new 第一篇文章
会在source\_posts文件夹内新增一个名为第一篇文章.md的文件。你也可以不使用命令，手动在文件夹下面新建。然后就可以编辑你的内容了。

至此，我们已经在本地搭建了一个博客。我们可以通过执行hexo new [name]来生成一篇文章，通过hexo server来预览。如果你只是想记笔记，而不想让别人看到你的博客，那么这就足够了。至于 Hexo 剩下的功能，草稿，修改网站标题等配置，可以参考hexo 官网。

### 搭建服务器环境
如果你希望别人看到你的博客，如果你有自己的服务器，还有自己域名。那么我们来搭建服务器环境，让别人也能看到你的博客。
之前在本地搭建我们已经知道了 Hexo 如何工作，我们已经得到了一个 hexo 生成的静态页面，我们只需要通过一个服务器来托管静态页面就可以了。这里我们选择 Nginx。在开始配置之前，确保你已经有了自己的服务器，而且知道如何远程登录。这里我使用的xshell远程登录，用xftp传输文件。你也可以直接使用ssh,scp命令或者其他类似工具。
首先远程登录服务器

![5](http://pnj2tr0xr.bkt.clouddn.com/5.png)

依次执行
```
curl -O http://nginx.org/keys/nginx_signing.key
apt-key add nginx_signing.key
apt-get update
apt-get install nginx -y
```
安装好 nginx 之后,在浏览器里面输入你的服务器 ip 地址。如果看到这个页面说明 Nginx 已经安装并启动
![6](http://pnj2tr0xr.bkt.clouddn.com/6.png)
如果没有看到，那么很有可能你服务器安全组里面确认没有开启了 80 端口
现在我们新建一个文件夹来存放生成好的静态页面，这里我选择的是
\data\www
```
mkdir -p \data\www
```
将本地打包生成好的静态页面放入新建的文件夹，这里使用的是xftp
![7](http://pnj2tr0xr.bkt.clouddn.com/7.png)
![8](http://pnj2tr0xr.bkt.clouddn.com/8.png)


然后编写一个 Nginx 的配置
```
cd /etc/nginx/conf.d/
vim blog.conf
```
写入以下内容
```
server {
    server_name blog.nukebomb.cn;
    root /data/www/public;
    index index.html;
}
```
其中server_name是你自己的域名,然后执行
```
nginx -s reload
```
让配置生效。此时访问你的域名，你可以看到网页已经部署好了。
![9](http://pnj2tr0xr.bkt.clouddn.com/9.png)

自此我们搭建好了本地和服务器环境，已经可以实现本地编写文章，然后编译出静态文件放入服务器让其他人看到了。但是每次书写完成，都要编译，然后手动登录服务器把生成好的静态文件放进去好像有点麻烦。所以最后我们在服务器配置 git，通过 git 的钩子来自动完成部署。

### git 自动部署
首先确保你的本地和服务器端都安装好了 git，windows 的安装很简单，服务器上执行
```
apt-get install git -y
```
也很容易安装 git。
接下来，首先我们建立一个用户 git，并通过 SSH 来实现登录
```
adduser git
```
过程中会输入一对密码和其他信息，其他信息可以直接回车跳过。git 用户新增之后我们切换到 git 用户
```
su git
```
然后建立 ssh 需要的文件夹和公钥文件，并修改权限
```
git@aliyun: mkdir ~/.ssh #git@aliyun表示当前处于git用户下，下同
git@aliyun: touch ~/.ssh/authorized_keys
git@aliyun: chmod 700 ~/.ssh
git@aliyun: chmod 600 ~/.ssh/authorized_keys
```
我们完成了服务器端 ssh 的准备。现在回到本地。
首先我们确保本地已经有 ssh，如果是 win10 用户而且一直保持了更新，可以通过以下方法开启系统自带的 openSSH，其他情况可以手动安装 openSSH。
win+X 选择应用和功能，然后进入管理可选功能，点击上方添加功能选择OpenSSH客户端安装

![10](http://pnj2tr0xr.bkt.clouddn.com/10.png)
![11](http://pnj2tr0xr.bkt.clouddn.com/11.png)


安装好了后，我们开启一个命令行，生成一对密钥对。
```
ssh-keygen -t rsa
```
![12](http://pnj2tr0xr.bkt.clouddn.com/12.png)

注意在生成过程中，一路回车，保证生成的位置是默认位置和密钥对没有密码。成功之后在C:\Users\<name>\.ssh对应的路径可以看到两个文件id_rsa和id_rsa.pub我们把公钥id_rsa.pub传送到服务器的\home\git路径。(和上面一样用 xftp 或者你有已经你自己的办法)
然后继续在服务器端操作，将公钥内容复制到 authorized_keys
```
git@aliyun: cd
git@aliyun: cat id_rsa.pub >> .ssh/authorized_keys
```
然后在本地开启命令行测试
```
ssh -T git@<你的服务器ip>
```
结果如图
![13](http://pnj2tr0xr.bkt.clouddn.com/13.png)

自此我们已经能够通过私钥以 git 用户远程登录服务器了。接下来我们配置 git 和 git 钩子来完成自动部署。
登录服务器切换到 git 用户，用户根目录新建一个裸 git 库 blog.git
```
su git
git@aliyun: cd
git@aliyun: git init --bare blog.git
```
我们在本地测试 clone 一下这个库
```
git clone git@<你的服务器ip>:blog.git
```
![14](http://pnj2tr0xr.bkt.clouddn.com/14.png)

然后添加一个 git 钩子，让同步完成之后将静态页面传到 nginx 托管页面
```
cd
git@aliyun: cd
git@aliyun: vim blog.git/hooks/post-receive
```
写入如下的内容保存
```
#!/bin/sh
git --work-tree=/data/www/public --git-dir=/home/git/blog.git checkout -f
```
最后出于安全考虑，我们禁止 git 用户登陆 shell。回到服务器，ctrl+D 返回 root 用户
```
vim /etc/passwd
```
修改最后一行
```
git:x:1000:1000:,,,:/home/git:/bin/bash
```
为
```
git:x:1000:1000:,,,:/home/git:/usr/bin/git-shell
```
自此我们完成了服务器端所有配置。

回到客户端，blog 项目下，修改项目根目录下的_config.yml,
在最下面deploy位置修改如下
![15](http://pnj2tr0xr.bkt.clouddn.com/15.png)

然后安装 git 提交需要的库
```
npm install hexo-deployer-git --save
```
然后执行
```
hexo clean && hexo generate -d
```
静态页面就会被提交到 nginx 托管的文件夹下。我们可以把两个常用的命令写成 script。编辑 package.json,添加框选部分
![16](http://pnj2tr0xr.bkt.clouddn.com/16.png)

之后我们就可以使用命令npm start启动本地服务器，用npm deploy启动部署。
### 用 vscode 编写
我们现在已经完成了一个博客的搭建，我们可以本地使用 markdown 编写文章，然后通过npm deploy一键发布到服务器上。最后推荐一下本地编写 markdown 的环境，如果你有更顺手的 markdown 工具，比如 atom，vim，那就继续使用你最顺手的就好。

我本地使用的 vscode 编辑器，加上一个叫markdown preview enhanced的插件。这个编辑器应该很多前端很熟悉，是一个基于前端技术开发的轻量级编辑器。现在 vscode 安装好markdown preview enhanced插件后，我们生成一篇文章。
我们编辑 markdown 可以使用ctrl+k v(先按 ctrl+k 松开 ctrl 按 v)打开预览或者右键选择打开预览
![17](http://pnj2tr0xr.bkt.clouddn.com/17.png)
插件自带 snippet，支持LATEX数学公式
![18](http://pnj2tr0xr.bkt.clouddn.com/18.png)
![19](http://pnj2tr0xr.bkt.clouddn.com/19.png)
在预览框上面右键还可以弹出菜单，可以将 markdown 编译为 html，pdf 等，这里要说明的是一个插入图片工具。markdown 插入图片本来是一件很麻烦的事情。但是这个插件可以直接选择图片，上传配置的图床，返回链接插入，支持七牛云，因此这里也以七牛云为例
登陆七牛云完成认证，新建一个 bucket，然后到个人中心，新建一对密钥
然后回到 vscode，选择文件-首选项-设置
分别修改插件配置关于七牛云的三个配置，bucket 名称，密钥对的值

![20](http://pnj2tr0xr.bkt.clouddn.com/20.png)
![21](http://pnj2tr0xr.bkt.clouddn.com/21.png)
配置完成，在 image-helper 选择七牛云，然后就会把图片上传到你的七牛云并返回链接插入到 markdown 中。

![22](http://pnj2tr0xr.bkt.clouddn.com/22.png)
![23](http://pnj2tr0xr.bkt.clouddn.com/23.png)