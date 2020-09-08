---
title: 服务器部署Hexo博客记录
date: 2020-02-02 21:17:57
categories:
- 博客
tags:
- Hexo
- Git
description: 回到博客。
---


[Hexo](https://hexo.io/)是一个使用基于`Node.js`的`Markdown`渲染器生成静态网页、使用`git`进行部署的博客框架。我在2017年曾经在`Github Pages`上搭建过一个基于`Hexo`的个人博客，但后来疏于维护就放弃了。旧事暂且不表，今天是2020年2月2日，我在自己的服务器上重新搭建了这个博客，也算是重拾一下geek的初心。

下文简单记录部署`Hexo`博客到服务器的过程。

# 本地安装Hexo环境

## 安装前提

- `Node.js`（版本需不低于`8.10`，建议使用`10.0`及以上版本）
- `Git`

## 安装Hexo

```bash
npm install -g hexo-cli
```

安装完成后，可使用如下命令进行测试：
```bash
hexo init blog_folder # blog_folder将用于存放博客渲染前的源代码
cd blog_folder
npm install
hexo server # 本地渲染网页，应当会提示可通过localhost:4000浏览渲染结果
```

# 服务器配置网站部署环境

`Hexo`通过命令`hexo deploy`，利用配置文件指定的`deployer`进行远程部署。

服务器上的架构由以下两部分组成：

- `Git`服务器
- `Nginx`网页服务器

其中，`Git`服务器创建网站数据的`repository`（就像在`Github Pages`上看到的那样），并且配置`post-receive`的`Git hook`以在每次接收完毕从本地发送的`push`时自动更新网站数据。

这里以`Ubuntu 18.04LTS`为例。

## Git服务器配置

在运行`Linux`系统的服务器上，可以基于`SSH`协议提供`Git`服务，通过`authorized_keys`来对用户进行认证。首先，需要确保服务器已安装`Git`。
```bash
sudo apt update && sudo apt install git
```
其次，在系统上创建`git`用户（并获得其`.ssh`目录），以专门用于提供`Git`服务。
```bash
sudo adduser git
su git
cd ~
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

然后，需要在`.ssh/authorized_keys`中添加本地的`SSH`公钥（即`~/.ssh/id_rsa.pub`文件的内容），每行一个公钥对应一个用户。我们需要手动将本地的`id_rsa.pub`的内容复制到服务器中的`authorized_keys`中；如果服务器上有已经上传的公钥，可按以下方式添加：
```bash
cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
```
这样一来，已添加的用户就能通过`SSH`协议以`git`用户身份登录服务器。但也正因此，该`git`用户存在安全问题，可通过限制其使用`git-shell`来解决；`git-shell`仅能用于进行和`Git`相关的活动。
```bash
which git-shell # 确定git-shell的路径
su # 由于git没有sudo权限，需要切换至root或有sudo权限的用户
nano /etc/passwd
```
在`git`用户对应的行中，将`/bin/bash`改为`which git-shell`命令的结果（通常是`/usr/bin/git-shell`）。这样一来，`git`用户就配置完毕了。

最后，只需要创建所需的`repository`即可。
```bash
$ cd <用于存放repositories的目录>
$ git init --bare project.git
Initialized empty Git repository in <用于存放repositories的目录>/project.git/
```

## Nginx网页服务器配置

首先，确保没有其他服务占用`80`端口（如`Apache`服务器）。然后安装`Nginx`：
```bash
sudo apt install nginx
```
对于`Ubuntu 18.04LTS`，安装结束后`Nginx`会自动开始运行。可通过如下命令进行检查：
```bash
systemctl status nginx
```

要确保运行正常，最直接的方式是向`Nginx`服务器发送网页请求。为此，需要知道服务器的公网IP地址。一般来说，服务器提供商都会给出IP地址。要在服务器上获取自己的公网IP，也可以通过以下方式实现：
```bash
ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'
```
该命令一般会返回好几个IP，可在浏览器中逐个检查。
或者也可以通过其他网站提供的服务：
```bash
curl -4 icanhazip.com
```
一旦知道了自己的IP地址，就可以通过访问`http://your_server_ip`来向`Nginx`服务器发送`Http`请求。如果返回`Nginx`默认页面，则说明运行正常。

### 关于防火墙

如果无法正常访问，可能是防火墙未开放`80`端口。通过以下命令检查防火墙状态：
```bash
sudo ufw status
```

如果想关闭防火墙：
```bash
sudo ufw disable
```

如果想添加防火墙规则，可先查看防火墙规则可用的应用选项：
```bash
$ sudo ufw app list
Available applications:
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```
可见，有3种和`Nginx`相关的选项：

- `Nginx Full`：80端口和433端口
- `Nginx HTTP`: 仅80端口
- `Nginx HTTPS`: 仅433端口

可先对`Nginx Full`添加允许规则，命令如下：
```bash
sudo ufw allow 'Nginx Full'
```
如果网站支持`Https`，可再使用以下命令修改防火墙规则，从而只允许`Https`流量：
```bash
sudo ufw delete allow 'Nginx Full' && sudo ufw allow 'Nginx Https'
```

需要注意的是，如果想打开防火墙，**首先要对`SSH`使用的端口添加允许规则，否则无法再次使用`SSH`登录服务器！**
```bash
sudo ufw allow ssh # 如果未使用SSH默认端口，则改为允许对应端口
sudo ufw enable # 打开防火墙
```

### Nginx服务器的常用操作和文件

可通过
```bash
sudo systemctl <command> nginx
```
来控制`Nginx`服务的运行。其中，`<command>`可替换为

- `stop`：停止服务；
- `start`：启动服务；
- `restart`：重启服务；
- `reload`：不中断服务，重载配置文件；
- `enable`：随系统启动而启动，默认为开启；
- `disable`：取消随系统启动而启动。

可使用`sudo nginx -t`来测试配置文件是否合法，通常用于重载配置之前。

常用文件如下：

- `/etc/nginx`：`Nginx`的配置目录；
- `/etc/nginx/nginx.conf`：`Nginx`的主配置文件，用于修改全局配置；
- `/etc/nginx/sites-available`：放置不同网站的`server blocks`的目录，其中的配置文件仅在被链接至`/etc/nginx/sites-enabled/`目录中时才生效；
- `/etc/nginx/sites-enabled`：放置生效的`server blocks`的目录，通常其中的配置文件为`/etc/nginx/sites-available/`目录中配置文件的软链接；
- `/etc/nginx/snippets`：放置一些配置片段，可被`Nginx`配置中的别处引用。

日志文件如下：

- `/var/log/nginx/access.log`：所有对服务器的请求都被记录于此，除非配置了其他指定文件；
- `/var/log/nginx/error.log`： `Nginx`的错误日志。

### 配置Nginx的server blocks（可选）

配置`Nginx`时，可使用`server blocks`来在单个服务器上封装不同网站的服务，类似于`Apache`的虚拟主机（`virtual hosts`）。在`Ubuntu 18.04`上，`Nginx`默认只有一个`server block`，位于`/var/www/html`，可提供一个网站的服务。为了可拓展性（如果未来有提供多个网站服务的需求），可自行配置一个`server block`。

#### 创建server block的网站文件夹

首先，创建域名（以`example.com`代替）对应的文件夹（`-p`自动创建parent文件夹）：
```bash
sudo mkdir -p /var/www/example.com/html
```
指定文件夹所有权：
```bash
sudo chown -R $USER:$USER /var/www/example.com/html
```
如果没有修改`umask`，文件夹的权限应该没有问题。也可用以下命令确保之：
```bash
sudo chmod -R 755 /var/www/example.com
```
创建测试网页：
```bash
echo "Welcome to test page!" > /var/www/example.com/html/index.html
```

#### 为Nginx添加server block的配置文件

无需改动`Nginx`的默认配置文件，可添加对应于网站域名的新配置文件：
```bash
sudo nano /etc/nginx/sites-available/example.com
```
将以下内容复制进去：
```
server {
    listen 80;
    listen [::]:80;

    root /var/www/example.com/html;
    index index.html index.htm index.nginx-debian.html;

    server_name example.com www.example.com;

    location / {
            try_files $uri $uri/ =404;
    }
}
```
其中，`root`对应于上一步创建的网站文件夹，而`server_name`对应于网站域名。

接下来，通过创建软链接，使能该配置文件：
```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```
`Nginx`在启动时查看其`/etc/nginx/sites-enabled`文件夹中的配置文件，并对对应的网站提供服务。现在，`Nginx`对`example.com`和`www.example.com`之外的请求按`/etc/nginx/sites-enabled/default`的配置提供默认网站服务；可通过以下命令取消`default`链接，从而仅对根据域名的请求提供服务：
```bash
sudo unlink /etc/nginx/sites-enabled/default
```

### 使用Let’s Encrypt对Nginx服务器提供Https支持（可选）

`Let's Encrypt`是一个证书颁发机构（Certificate Authority，CA），可用于获取并安装免费的`TLS/SSL`证书，从而为服务器提供`Https`支持。该CA提供了名为`Certbot`的客户端来自动化证书的安装和更新，对`Apache`和`Nginx`均提供了支持。

首先，安装`Certbot`：
```bash
sudo add-apt-repository ppa:certbot/certbot
sudo apt install python-certbot-nginx
```
然后，确定需要提供证书的域名，即`Nginx`的`server block`配置中的`server_name`的值，这里记为`example.com`。如果没有配置，则需要手动添加并重载`Nginx`配置。

获取`SSL`证书：
```bash
sudo certbot --nginx -d example.com
```
首次运行时，`Certbot`会要求提供email地址和同意服务条款；同意之后，`Certbot`会与`Let's Encrypt`服务器通信，并验证之前提供的域名的合法性。成功之后，`Certbot`会询问如何配置`Https`（是否重定向`Http`流量）。选择后，`SSL`证书便会被`Nginx`服务器载入。

安装证书后，`Certbot`会在`/etc/cron.d`中添加自动更新`SSL`证书的脚本。可通过以下命令进行验证更新功能：
```bash
sudo certbot renew --dry-run
```
如果没有输出错误信息，则代表更新功能正常。

## 通过Git hooks配置自动部署

本地使用`hexo d`进行部署时，会将渲染后的网站数据`push`到服务器上的`repo`；如果要更新到`Nginx`服务器监测的网站文件夹，还需要将网站数据`pull`过去。因此，只要在服务器的`repo`上配置`hooks/post-receive`即可自动化这一过程。具体流程如下：

首先，进入网站文件夹，并将`Git`服务器本地的`repo`给`clone`下来。
```bash
cd /var/www/example.com/html && rm * # 确保文件夹中没有其他内容
git clone <用于存放repositories的目录>/project.git .
```

然后配置`Git`服务器端`repo`的`post-receive`钩子：
```bash
cd <用于存放repositories的目录>/project.git/hooks
nano post-receive
```
将以下内容复制进去，并保存：
```bash
#!/bin/sh
unset GIT_DIR # 否则无法pull代码
cd /var/www/example.com/html
git pull origin master
```
这里注意保证`git`用户有执行权限：
```bash
sudo chmod a+x post-receive
```

# 本地配置Hexo deployer

打开本地的`_config.yml`，修改部署配置为：
```yml
deploy:
  type: git
  repo: ssh://git@example.com:port<用于存放repositories的目录>/project.git
  branch: master
```
其中`port`为端口号，取默认值`22`时可不填，简写为`ssh://git@example.com<用于存放repositories的目录>/project.git`。

至此，配置结束。可以用`hexo d`命令一键部署到服务器了。
