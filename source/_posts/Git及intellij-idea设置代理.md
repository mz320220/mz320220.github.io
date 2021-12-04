---
title: git及intellij idea设置代理
date: 2019-04-18 15:56:05
categories:
- git
- IDE 
---

除了在某些情况需要在天朝科学上网之外，各大公司普遍推动使用VDI云桌面也是全局性的走代理上网。那么从github上拉取代码就必须要对git和IDE单独配置代理服务器。

<!--more-->

## 1. Git设置代理

* 使用 `git协议` 时，设置代理需要配置 `core.gitproxy`

* 使用 `HTTP协议` 时，设置代理需要配置 `http.proxy`

* 而是用 `ssh协议` 时，代理需要配置ssh的 `ProxyCommand` 参数

在windows环境下，配置http方式的代理是最为简单方便的了。公司的vdi代理一般也是提供http协议address + port的形式。下面说明`http.proxy`的配置命令：

```shell
#针对单个项目进行代理配置：（进入到项目目录）
git config http.proxy http://127.0.0.1:8088  #URL:PORT 或 URI:PORT的形式

#需要鉴权的代理：添加username、password信息
git config http.proxy http://username:password@127.0.0.1:8088

#启用全局代理（公司的VDI环境直接全局配置比较合适） --global指令
git config --global http.proxy http://127.0.0.1:8088

#查看代理配置情况
git config --get --global http.proxy

#删除原有代理配置 --unset指令
git config --global --unset http.proxy

#如果https协议出现证书错误的情况，比如提示： SSL certificate problem
#可以尝试将sslVerify设置为false
git config --global http.sslVerify false
```

几点说明：

- 如果非必要，一般不使用 `--global` 的方式来设置代理，毕竟代理有的时候访问一些项目比直接访问还慢，特别是当代理在国外，项目源在国内的时候，按需使用才是王道。

- 不要多次使用不同的参数来设置代理， `--global` ， `--system` ， `--local` 各级设置后，可能会给自己带来不必要的麻烦。git默认是先到git Repository的配置文件中查找配置文件，如果没有才会到 `--global` 设置的文件中查找，因此，单个项目文件中的设置会覆盖 `--global` 的设置。

- 使用 `--global` 来配置的信息保存在当前用户的根目录下的 `.config` 文件中，而仓库中的配置保存在项目仓库的根目录下的 `.git/config` 文件中。

## 2. intellij idea设置代理

同样的问题，在VDI环境下所有软件都是需要自己手动配置代理信息，否则也无法拉去代码使用IDE的代码管理功能。以Intellij idea举例，快捷键 `Ctrl + Alt + S`打开设置窗口，输入`http`进入如下代理设置页面：

![intellij代理](http://martintop-image.oss-cn-shenzhen.aliyuncs.com/19-4-18/intellij%E4%BB%A3%E7%90%86.png?Expires=1555582022&OSSAccessKeyId=TMP.AgHyuUNIJaNN5_913dyKrg-GhAE6NCPDN6o2x-MgsVpJxo7nTf175B8NTrXKADAtAhRm3eG6rtWyCoOxwQpOHH49v6U2KgIVAMcBjCxr3zYBCcLqOI_TRaXQaMoM&Signature=AyN4EpzuYhow97Rt%2Fh30t8rS%2Br8%3D)

按要求配置后重启IDE即可登录配置git账号，拉取项目代码。



> 参考：
>
> [windows下设置git代理](https://blog.csdn.net/lenglong110/article/details/52411230)