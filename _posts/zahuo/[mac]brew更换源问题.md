---
layout: post
title: "[小麻烦]brew更换国内源问题"
subtitle: '[小麻烦]brew更换国内源问题'
author: "Hzy"
header-style: text
tags:
  - 杂货
---

## 用原来的brew install ,让我有种度日如年的感觉，而且正好的是vpn挂了...没办法了，只能更换一些源了。


### 1. 更换源


```
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git

cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git

cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
```

### 2. 更换镜像

```
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.bash_profile
source ~/.bash_profile
```


> 完毕，速度总算快了很多...