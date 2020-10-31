---
title: Windows子系统
abbrlink: 7a22618
date: 2020-04-17 09:31:41
categories:
  - Windows
tags:[Windows]
---

# 相关操作

### 开启/重启

以管理员身份运行 `CMD` 窗口,  开启或者关闭子系统

``` bash
# 关闭
net stop LxssManager
# 开启
net start LxssManager
```



### Docker 服务器开启

以管理员身份运行 Ubuntu

```bash
sudo systemctl daemon-reload
```

