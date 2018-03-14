---
layout:     post
title:      "如何配置docker使用HTTP代理"
subtitle:   ""
description: "如何配置docker使用HTTP代理"
date:       2018-03-13 18:00:00
author:     "赵化冰"
header-img: "img/in-post/docker.jpg"
published: true
tags:
    - Tips
    - Docker
---
## 设置docker使用http proxy
```
sudo mkdir -p /etc/systemd/system/docker.service.d

echo '
[Service]
Environment="HTTP_PROXY=http://proxy.foo.bar.com:80/"
' | sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf
```

## 加载配置并重启docker
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```