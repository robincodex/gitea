---
date: "2023-07-06T09:00:00+08:00"
title: "Authentication in action"
slug: "authentication-in-action"
weight: 110
draft: false
toc: false
menu:
  sidebar:
    parent: "actions"
    name: "Authentication in action"
    weight: 110
    identifier: "actions-authentication-in-action"
---

# Authentication in action

每次运行的action都是一个独立的容器，往往不会带有用户凭证，此时就需要通过如 API TOKEN, Access Token 之类的进行自动验证。

# Git Authentication

在action里面通过Git拉取私有仓库需要通过验证，然而自动化过程中无法输入账号密码，
其中一种方式是通过SSH密钥，SSH密钥可以免于手动输入密码，但是SSH连接比较复杂，对于不熟悉SSH的人来说会经常出问题，
另外一种方式是通过HTTP Header验证，这也是最简单的办法，还能保持https的使用方式。

目录

{{< toc >}}

## 原理

- 首先需要在 设置 > 应用 里生成一个AccessToken，这个 AccessToken 需要给予读取仓库的权限。
- 将`x-access-token:<AccessToken>`经过Base64编码得到一个Base64字符串。
- 最后配置Git  
  `git config --global http.https://gitea.example.com/.extraheader "AUTHORIZATION: basic <Base64字符串>"`

## 在workflow配置中可以执行下面的指令

```yaml
jobs:
  example:
    runs-on: ubuntu-latest
    steps:
      - run: git config --global http.https://gitea.example.com/.extraheader "AUTHORIZATION: basic $(echo -n "x-access-token:${{ secrets.YOUR_TOKEN }}" | base64)"
      - run: git clone ...
```

> 如果你正在使用Golang，并且使用了自己的私有库，那么你可以通过这个办法设置好之后就可以正常通过`go get`, `go mod tidy` 或者 `go mod download` 等拉取私有仓库。
