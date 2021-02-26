---
GA: UA-139154617-3
disqus: ericstudio
langs: zh-tw
---
# Docker Swarm 發生 Error grabbing logs

### 情境

當使用指令 `docker service logs [service name]` 要讀取log時，如果發生下列錯誤。

> error from daemon in stream: Error grabbing logs: rpc error: code = Unknown desc = warning: incomplete log stream. some logs could not be retrieved for the following reasons: node XXXXXXXXXXXXXXXXXXXXXXXXXXXXX is not available


這似乎是docker 的bug ，在2017年就有人在docker的官方github發這個[issue](https://github.com/moby/moby/issues/35011)，但是到今天(2021年3月，版本19.03.14)為止都還沒解決。

看起來，把host重新開機或者把Service 移除再重新部署都沒有辦法。

### 解決方法

目前可行解決方法(workaround)，是重新產生swarm的憑證

```
docker swarm ca --rotate
```
###### tags: `docker` `docker swarm`