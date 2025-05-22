# 1. 前言

在网页开发过程中，有时我们需要将本地的页面进行预览。但是本地webpack或者vite开发的项目运行的localhost页面，别人的外网无法直接通过浏览器访问，这时我们可以使用natapp进行内网穿透和预览本地页面。

# 2. natapp介绍

[natapp](https://natapp.cn/)是一个内网穿透工具，内网穿透也叫做内网映射，也叫`NAT穿透`。简单来说就是让外网能够访问内网，即把自己的电脑当服务器，natapp会分配一个专属域名/端口，这时本地运行的localhost页面就已经在公网上了，可以让别人访问。

- [传送门](https://natapp.cn/)

# 3. natapp安装

[官方教程传送门](https://natapp.cn/article/natapp_newbie)

- 进入官网，先注册，注册成功后，点击顶部的客户端下载，选择适合自己系统的安装包下载。下载之后，解压至任意目录，得到natapp.exe程序。

![image1](https://cdn.natapp.cn/uploads/ueditor/php/upload/image/20160529/1464512052420728.jpeg)

- 在官网申请一个免费隧道，配置好隧道，获得您的隧道登录凭证authtoken，点击复制。

- 新建文本文档，将下面内容放进去，填入你的authtoken，然后把文件后缀名改为.ini，和natapp.exe程序放在同一个目录下。

```ini
#将本文件放置于natapp同级目录 程序将读取 [default] 段
#在命令行参数模式如 natapp -authtoken=xxx 等相同参数将会覆盖掉此配置
#命令行参数 -config= 可以指定任意config.ini文件
[default]
authtoken=                      #对应一条隧道的authtoken
clienttoken=                    #对应客户端的clienttoken,将会忽略authtoken,若无请留空,
log=none                        #log 日志文件,可指定本地文件, none=不做记录,stdout=直接屏幕输出 ,默认为none
loglevel=ERROR                  #日志等级 DEBUG, INFO, WARNING, ERROR 默认为 DEBUG
http_proxy=                     #代理设置 如 http://10.123.10.10:3128 非代理上网用户请务必留空
```

- 双击运行natapp.exe程序，打开浏览器，输入你的隧道生成的服务器地址（也就是域名），即可看到你的隧道列表。

# 4. natapp使用

运行你的Vite或者webpack项目，然后运行natapp.exe程序，打开浏览器，输入你的隧道生成的服务器地址（也就是域名），即可看到本地页面已经在公网映射好了。

> 注意：Vite项目需要在vite.config里配置server.allowedHosts: true

如果有疑问，可以去官网寻求帮助：
- [官方教程传送门](https://natapp.cn/article/natapp_newbie)
- [官方错误处理传送门](https://natapp.cn/article)