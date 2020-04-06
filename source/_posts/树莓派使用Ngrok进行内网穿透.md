---
tags: 树莓派, ngrok
categories: 树莓派
---
> 本教程只为记录自己在实际操作中遇到的问题。具体步骤亲测有效。

### 0. 背景
使用树莓派做自己网站的服务器。
<!-- more -->
### 1. 注册Ngrok账号
网站地址：https://www.ngrok.cc
具体注册方式不再赘述。
### 2. 创建隧道
注册完成后，我们开始创建隧道。
这里我们选择免费的通道，如图：
![](https://i.loli.net/2020/03/30/cAjzwBSTQKp1ENy.png)
点击立即购买， 进入配置页面，如图：
![](https://i.loli.net/2020/03/30/mJ38W7ZASTyvzLd.png)
添加完成后， 我们得到一条隧道，然后进行编辑：
![](https://i.loli.net/2020/03/30/f83tjQNwqrvg5MK.png)
然后填写配置， 我这里是使用了自定义域名， 如果没有域名， 请选择使用前置域名。填写完成后点击保存修改。
![](https://i.loli.net/2020/03/30/PkEfBdnX6gzDHVh.png)

### 3. 下载客户端
登录树莓派下载Nrgok客户端。客户端我们选择ARM版本。然后将下载好的文件上传至树莓派。
![](https://i.loli.net/2020/03/30/C3vVFHj5zResxbl.png)
网站地址：https://www.ngrok.cc/download.html
或直接在树莓派上下载， 具体命令如下：

    #1. 下载客户端
    wget  http://hls.ctopus.com/sunny/linux_arm.zip
    #2. 解压
    unzip linux_arm.zip

### 4. 运行

    #3. 运行客户端
    linux_arm/sunny clientid 隧道id
到此， 就已经大功告成！在浏览器访问：
![](https://i.loli.net/2020/03/30/fe7yYDmBSckt6xT.png)

官方文档：http://www.ngrok.cc/_book/start/ngrok_linux.html