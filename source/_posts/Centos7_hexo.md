---
title: 将Hexo静态博客部署到阿里云Centos7
date: 2017-03-20 03:25:24
categories: Hexo
tags: [hexo]
comments: false
posts: hexo
---
 

# 前言

 此文记录自己将Hexo静态博客部署到阿里云Centos7上的过程。
 

# 环境的配置

 
> 本地环境配置

>服务器环境配置

## 本地环境配置
 1. [Git安装](https://www.jianshu.com/p/414ccd423efc)
 2. [Nodejs安装](https://blog.csdn.net/zjh_746140129/article/details/80460965)
 3. Hexo安装
第一步和第二步安装完成后,进行一些操作
生成ssh公钥(<font color=#FF5722 >公钥作用：设置SSH公钥认证，pull,push不需要每次都输入密码</font>)
桌面右键点击<font color=#FF5722 >Git Bash Here</font>,在打开的页面输入<font color=#FF5722 >ssh-keygen -t rsa</font>连续敲击回车三次

![hexo_01.jpg](http://img.lmdream.cn/img/bolg/hexo/hexo_01.jpg)
出现如下内容就证明就是成功
![hexo_02.jpg](http://img.lmdream.cn/img/bolg/hexo/hexo_02.jpg)

## 阿里云Centos环境配置

安装git
```
yum install git
```
创建Git账户
```
adduser git
```
添加账户权限
```
chmod 740 /etc/sudoers
vim /etc/sudoers
```
找到
```
## Allow root to run any commands anywhere
root    ALL=(ALL)     ALL
```
添加以下内容、
```
git     ALL=(ALL)     ALL
```
按ESC进入Command模式，然后输入<font color=#FF5722 >:wq</font>回车保存并退出。

修改权限
```
chmod 400 /etc/sudoers
```
切换至git用户
```
su git
```
创建 ~/.ssh 文件
```
mkdir ~/.ssh
```
编辑~/.ssh/authorized_keys 文件,然后将生成的id_rsa.pub文件中的公钥复制到authorized_keys
```
vim ~/.ssh/authorized_keys
```
赋予权限
```
chmod 755 ~
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```
在本地git测试是否能免密登录git,其中server为填写自己的云主机IP，执行输入yes后不用密码就说明好了
```
ssh -v git@server
```
创建目录

切换到到root账户,以catablog作为Git仓库目录
```
mkdir /var/catablog
```
仓库所属用户改为git
```
chown -R git:git /var/catablog
chmod -R 755 /var/catablog
```
blog作为网站根目录
```
mkdir /var/www/blog
chown -R git:git /var/www/blog
chmod -R 755 /var/www/blog
```
然后创建一个裸的 Git 仓库
```
cd var/catablog
git init --bare blog.git
```
创建一个新的 Git 钩子,进入cd /var/catablog/blog.git/。新建一个新的钩子文件 post-receive。

创建新的钩子文件 <font color=#FF5722 >post-receive</font>
```
touch post-receive
```
编辑这文件
```
vim /var/catablog/blog.git/hooks/post-receive
```
进入文件的编辑模式，在该文件中添加以下代码
```
#!/bin/bash
git --work-tree=/var/www/blog --git-dir=/var/catablog/blog.git checkout -f
```
按 Esc 键退出编辑模式，输入”:wq” 保存退出。

修改文件权限，为可执行权限
```
chown -R git:git /var/catablog/blog.git/hooks/post-receive
chmod +x /var/catablog/blog.git/hooks/post-receive
```
Git 仓库搭建完成。
### 配置Nginx
使用宝塔面板来部署Nginx
[宝塔Linux官网](https://www.bt.cn/bbs/portal.php)


Linux面板6.0安装命令(暂时仅兼容Centos7.x，其它系统版本请安装5.9稳定版)
```
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && bash install.sh
```
Linux面板6.0升级专业版:
```
curl http://download.bt.cn/install/update6.sh|bash
```
稍等一会，等待升级
初次访问需要安装nginx
在页面中会出现访问地址、账户和密码，此处需要去云服务器中，在防火墙中将8888端口加入到规则中，否则会出现无法访问。
复制网址到浏览器中进行登录，登录成功后，选择Nginx的部署方案，静静等待部署。
![hexo_03.jpg](http://img.lmdream.cn/img/bolg/hexo/hexo_03.jpg)
部署完成

点击网站-添加站点>输入域名>底部的PHP版本选择"纯静态">提交。
![hexo_04.jpg](http://img.lmdream.cn/img/bolg/hexo/hexo_04.jpg)
网站创建完成后点击设置-配置文件
```
server
{
    listen 80;
    # server_name 填写自己的域名
    server_name ali6.cn blog.ali6.cn;
    index index.php index.html index.htm default.php default.htm default.html;
    # 这里root填写自己的网站根目录，修改为/var/www/blog
    root /var/www/blog;
```
-保存
![hexo_05.jpg](http://img.lmdream.cn/img/bolg/hexo/hexo_05.png)
点击设置-网站目录，修改为/var/www/blog ，保存
![hexo_06.jpg](http://img.lmdream.cn/img/bolg/hexo/hexo_06.jpg)
重启宝塔面板服务
```
service bt restart
```
此时Centos7环境已经配置完整,然后Hexo的配置



# 在本地环境安装部署Hexo
## Hexo部署
[Hexo官网](https://hexo.io/zh-cn/)
建议去官网了解一下Hexo搭建的基本信息,如果已经了解可跳过

在自己的盘符下，创建一个blog文件夹，打开blog文件夹，在这个文件夹内右键点击Git Bash Here打开命令行终端。
执行如下命令
```
npm install hexo-cli -g
hexo init 
npm install hexo server
npm install hexo-deployer-git --save
```

```
#定义邮箱
git config --global user.email "邮箱" #更换为你的邮箱地址
#定义名称
git config --global user.name "名称" #自定义一个名称就行
```
说明
用户名和邮箱地址的作用
用户名和邮箱地址是本地git客户端的一个变量

每次commit都会用用户名和邮箱纪录。
github的contributions统计就是按邮箱来统计的
## 配置_config.yml
打开blog文件夹，打开_config.yml, 找到deploy
```
deploy:
  type: git
  #server改为你的服务IP地址或解析后的域名
  catablog: git@server:/var/catablog/blog.git
  branch: master
  ```
进行保存。

在blog文件夹，在这个文件夹内右键点击
<font color=#FF5722 >Git Bash Here</font>
打开命令行终端
执行如下命令
```
hexo clean 
hexo g -d
```
等待部署，如果页面不出错显绿表示部署完成，浏览器输入域名或ip地址后就可以看到刚刚部署的Hexo博客了
# 常见问题
部署过程中，执行 hexo d发现部署出现如下图问题
![hexo07.png](http://img.lmdream.cn/img/bolg/hexo/hexo07.png)
可能是因为权限问题,需要检查我们在上述的git操作部署是否使用了git用户操作，若是没有，需要给相应的目录更改用户组
使用

```
chown -R git:git /var/catablog/
```

这条命令递归的将repo目录及其子目录用户组设置为git。
同时使用
```

chown -R git:git /var/www/blog
```
这样即可解决此类问题
# 后记
我也是初次搭建接触Hexo和搭建，在网上也查询了很多的资料，此篇文章单纯的将自己的搭建步骤记录下来，防止以后忘记.
## 参考资料
https://www.jianshu.com/p/0f9dfa9c141b


