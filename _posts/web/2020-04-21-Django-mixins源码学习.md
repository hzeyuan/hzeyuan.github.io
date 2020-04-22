---
layout: post
title: "Django源码分析(一)--View"
subtitle: 'Django mixins源码学习'
author: "Hzy"
header-style: text
tags:
  - Django
---

### 从今天开始每天记录一个Django源码的知识点。

> 每当我们需要写基于类的视图函数时，我们都像下面这样：

```
from django.views import View
class TestView(View):
  pass
```

> 继承这个类并且
