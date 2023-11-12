---
title: Pixiv 图片反向代理
category:
tags:
date: 2022-11-03 14:47:33
---

众所周知，Pixiv 在大陆被墙，并且图片服务器域名 i.pximg.net 有盗链保护，只要 Referer 不是来自 Pixiv 的请求都会返回 403。那么，如何才能在大陆地区直接访问图片资源？

<!-- more -->

## 为什么要代理图片？

p 站都被墙了这么多年，既然都准备走反代了，为什么还要单独对图片？直接搭建反代站不是更好么？

首先，反代站可能会导致 P 站发邮件到您的主机商投诉，甚至还会导致 VPS 服务等被终止。~~其次是我当初只是想给群里做一个色图机器人（~~

类似例如 [lolicon](https://api.lolicon.app/)、[acgmx](https://www.acgmx.com/) 等许多基于 p 站的色图 API，返回的都是 Pixiv 官方图片链接，并不能直接访问，所以便有了单独针对图片代理的需求。

## 第三方代理

若不会自己搭建，也可以直接使用第三方服务。[pixiv.cat](https://pixiv.cat/) 就是一个专门用于 p 站图片代理的站点，甚至能直接通过图片 id 请求获取插画。

我最初也是使用的该服务，不过后续因主域名在大陆被墙，导致服务全挂掉了（我当时还以为是机器人坏了，排查了十多分钟才发现是代理的问题），便有了自建的想法。

## 反向代理

如果你有自己的服务器，可以使用 nginx 进行代理。

```conf
proxy_cache_path /path/to/cache levels=1:2 keys_zone=pximg:10m max_size=10g inactive=7d use_temp_path=off;

server {
    listen 443 ssl http2;

    ssl_certificate /path/to/ssl_certificate.crt;
    ssl_certificate_key /path/to/ssl_certificate.key;

    server_name i.pixiv.cat;
    access_log off;

    location / {
    proxy_cache pximg;
    proxy_pass https://i.pximg.net;
    proxy_cache_revalidate on;
    proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
    proxy_cache_lock on;
    add_header X-Cache-Status $upstream_cache_status;
    proxy_set_header Host i.pximg.net;
    proxy_set_header Referer "https://www.pixiv.net/";

    proxy_cache_valid 200 7d;
    proxy_cache_valid 404 5m;
 }
}
```

没有服务器也没关系，可以利用 Cloudflare 的 Workers 白嫖，每日免费提供 10 万次请求，完全满足日常使用。~~总不至于不够用吧？别冲了，人要冲死了~~

```javascript
export default {
  async fetch(request) {
    const someHost = 'i.pximg.net';
    const someUrl = 'https://www.pixiv.net/';

    const url = new URL(request.url);
    url.hostname = someHost;

    const requestInfo = new Request(url, request);
    requestInfo.headers.set('Referer', someUrl);

    return fetch(requestInfo);
  },
};
```

## 我要色色！

配置相关 DNS 解析后，就可以直接通过 URL 直接访问 p 站图片了。

Pixiv 网站上的原始链接（直接打开或在其他网站使用会返回 403）：  
https://i.pximg.net/img-master/img/2021/06/10/18/09/04/90457556_p0_master1200.jpg
![原始图片链接（无法正常显示）](https://i.pximg.net/img-master/img/2021/06/10/18/09/04/90457556_p0_master1200.jpg)

反向代理（pixiv.yuki.sh）（能正常访问）：  
https://pixiv.yuki.sh/img-master/img/2021/06/10/18/09/04/90457556_p0_master1200.jpg
<img src="https://pixiv.yuki.sh/img-master/img/2021/06/10/18/09/04/90457556_p0_master1200.jpg" alt="kokkoro" width="300" />

嘛，如果你看完文章还是嫌麻烦，也可以直接把 pixiv.yuki.sh 拿去用，给个 star 就行（

## 罗翔说法

~~张三在网上，用别人提供的反代服务，居然在群里发 R18 色图，请问张三这一行为触犯了哪项法律~~

技术虽无罪，但还是提醒一下，发送的图片请务必遵守你所在国家及其地区的法律法规，别被请喝茶了。出于几个月前发生的某件事情（关注 p 站的应该都知道，就不在这里提了），特别强调一下，别想着靠这个去盈利，本来就靠爱发电，某些人别把开源环境弄得越来越恶臭。

## 参考文献

- [Pixiv 圖片代理](https://pixiv.cat/reverseproxy.html)
