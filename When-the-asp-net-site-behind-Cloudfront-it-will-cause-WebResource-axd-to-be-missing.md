---
GA: G-2PG7XZTMCQ
disqus: ericstudio
langs: zh-tw
title: 架設在AWS的asp.net web form站台，使用Cloudfront之後，WebResource.axd會消失
---
# 消失的WebResource.axd

同事回報說有客戶要把產品架在AWS上，但是架好之後有個奇怪的問題，在https的情況下，會有javascript error。 

![](https://cdn.jsdelivr.net/gh/ericstu/ericstu/images/20210104174109.png "javascript error")

## 調查

但是在http卻是正常的。 看起來是某些script沒有載入，但神奇的是在使用https連線的情況裡，從開發者工具看，所有的Request的連線狀態統統都是200，並沒有401, 500 這種有的沒有的狀態，重新測了幾次把cache清掉，所有的Request都還是200。 

另外還有發現下列狀況

1. 仔細的比對發出去的request，原來 http 跟 https 的request數量不一樣，赫然發現在https之下，
Browser 並沒有對WebResource.axd 發出Request

2. https站台產生的html 裡，並沒有<script src="/WebResource.axd?d=...;t=..."></script>

自此射茶包的方向就轉成找為什麼 WebResurce.axd 會消失。



## 原因 

#### User-Agent
可能是因為 `ASP.NET Web form` 己經是阿祖級的產品,由於年代久遠的關係，我沒有找到微軟的官方文件說明，但從一些討論

https://stackoverflow.com/questions/18244223/webform-dopostbackwithoptions-is-undefined-in-ie11-preview

https://stackoverflow.com/questions/17861424/webresource-axd-not-working-with-internet-explorer-11

可以推敲出 WebResurce.axd 應該是會根據使用者所使用的瀏覽器來產生內容，但是當Asp.net底層不知道或不認識使用者的瀏覽器，就直接略過，不產出WebResurce.axd。 (這是什麼奇怪的設計啊)

我們都知道Server端是靠http request header裡的User-Agent才知道使用者所使用的瀏覽器是什麼。但是通常Browser都會送User-Agent給Server才對。

為什麼Server會沒收到或者不認識User-Agent?


#### 網路架構
再來另一個點是一開始同事講的並不是很精確，所謂的http可以https不行，原來是說客戶把站台放在[CloudFront](https://aws.amazon.com/tw/cloudfront/)後面，因為CloudFront有提供免費的SSL憑証，而http是使用不同的domain直接連到VM的IIS上。

#### 架構示意圖如下

![](https://cdn.jsdelivr.net/gh/ericstu/ericstu/images/20210105113219.png "架構示意圖")

也就是說CloudFront可能會修改http request headers.

在AWS的[說明文件](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/RequestAndResponseBehaviorCustomOrigin.html#request-custom-headers-behavior)中，HTTP Request Headers and CloudFront Behavior 這段裡有整理了對各種http header 的處理方式 。

其中對User-Agent的說明

>CloudFront replaces the value of this header field with Amazon CloudFront. If you want CloudFront to cache your content based on the device the user is using, see [Configuring caching based on the device type](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/header-caching.html#header-caching-web-device).

原來CloudFront會把User-Angent換成Amazon CloudFront，難怪asp<span>.</span>net不認識這個User-Agent。


#### HTTPS的吶喊:我是無辜的!!

![](https://i.imgur.com/vzwtVjW.png)

## 解法

後來終於在StackOverflow找到這篇一樣是把asp.net的站台架在AWS上CloudFront裡，造成WebResurce.axd的問題，而且還有提供解法。https://stackoverflow.com/questions/32913318/webresource-axd-missing-from-net-webforms-site-when-behind-cloudfront

#### 設定的畫面, 在Behaviors的TAB選擇Create Behavior
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu/images/20210105103045.png)

#### 選擇"根據選取請求標頭的快取" --> "白名單" --> 允許名單標頭-"新增自訂"
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu/images/20210105103156.png)

#### 把User-Agent加到白名單裡
![](https://cdn.jsdelivr.net/gh/ericstu/ericstu/images/20210105103232.png)



###### tags: `射茶包` `AWS CloudFront` `WebResource.axd` 'asp.net'