---
title: 阿里云免费证书域名启用 https
date: 2017-07-10 06:58:11
updated: 2017-07-16 11:01:11
overdue: true
tags:
categories: 编程
---

{% qnimg https.jpg 'alt:iyearn.top 的 https 证书' %}

😄  给域名加了个小绿锁

### 关于 https
>超文本传输安全协议（Hypertext Transfer Protocol Secure）是一种通过计算机网络进行安全通信的传输协议。HTTPS经由HTTP进行通信，但利用SSL/TLS来加密数据包。HTTPS开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。

一句话，就是更加安全。比如电信运营商的广告劫持，https 就可以杜绝。


**有利必有弊，使用 https 的缺点**  😭

1、增加额外费用：虽然也有免费的SSL证书，但是要想长期的运营网站，还是要购买收费的SSL。

2、网站访问速度变慢：据 ACM CoNEXT 数据显示，使用HTTPS协议会使页面的加载时间延长近50%（可以使用 CDN 加速部分解决 😘 ），增加10%到20%的耗电。此外，HTTPS协议还会影响缓存，增加数据开销和功耗。HTTPS 连接服务器端资源占用高很多。

3、不能调用http的资源：https网站无法调用任何http的资源。

这篇文章不具有通用性，可以看我的这篇文章[如何给网站添加免费的 https 证书](https://hufangyun.com/2019/https-config/)

### 阿里云申请 https 证书
腾讯云也有相同的免费证书功能。如果使用腾讯云的服务器，可以选择使用腾讯云的免费证书。😃

如下图，选择证书服务，进入后，选择 “购买证书”，然后选择 “免费型DV SSL”，价格就自动变成零了。😉

{% qnimg aliyun-https-1.jpg 'alt:阿里云证书' extend:?imageView2/2/w/500 %}

{% qnimg aliyun-https-2.jpg 'alt:阿里云证书购买' extend:?imageView2/2/w/500 %}

有效期是一年，而且只能绑定一个域名。绑定在主域上，子域名是不能使用的。😵

需要填写的就是邮箱、姓名一类的个人信息，验证选择 DNS 证书是自动签发的，所以基本填完就自动申请成功了。

如果域名的 DNS 解析也使用的阿里云的，阿里云会自动添加一条 DNS 解析用做证书验证。如果使用的非阿里云的域名解析，需要自己手动添加。😝

然后根据阿里云的提示配置 `nginx`

下载证书，然后将证书上传到服务器 `/etc/nginx/cert` 目录下。
（文件上传服务器可以参考我这篇[文章](https://hufangyun.com/2015/linux-file-upload/)
）😃
#### 访问 `http` 自动跳转到 `https`

我是使用 `nginx 497` 错误码处理的,但是这种方式耗费服务器资源，如果想使用其他的方式可以参考文章末尾的[扩展阅读](#扩展阅读) 。 😋

**思路**

当站点只允许 `https` 访问时，用 `http` 访问 `nginx` 会报出497错误码，然后我们利用 `error_page` 命令将497状态码的链接重定向到我们的域名上

配置 nginx 如下：

```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;

  listen 443;
    #为一个server{......}开启ssl支持
  	ssl on;
  	root html;
  	index index.html index.htm;
    #指定PEM格式的证书文件
  	ssl_certificate   cert/214190650300318.pem;
    #指定PEM格式的私钥文件
  	ssl_certificate_key  cert/214190650300318.key;
  	ssl_session_timeout 5m;
  	ssl_ciphers ECDHE-R-SHA256:ECRC4;
  	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  	ssl_prefer_server_ciphers on;

	server_name localhost;

        #http => https 让http请求重定向到https请求
        error_page 497  https://$host$uri;
}
```

### 扩展阅读
 [nginx强制使用https访问(http跳转到https)](http://blog.csdn.net/wzy_1988/article/details/8549290)
