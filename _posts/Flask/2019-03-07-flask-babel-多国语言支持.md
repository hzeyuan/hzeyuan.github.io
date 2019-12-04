---
layout: post
title: "Flask-babel-多国语言支持"
subtitle: 'Flask-babel-多国语言支持'
author: "Hzy"
header-style: text
tags:
  - Flask
---

>最近发现写文章时，开头内容概括的必要性，很多文章其实都内容都长的很像，但我每次刚接触到一个新扩展的时候，我连这扩展最基本的都还不太清楚，其他很多特性就上来了..，然后我得慢慢研究，做作对比才能看得懂。
>
-----

最近写的网站需要支持多国语言，然我就在网上找了flask-babel这个扩展。

-----
### 大致是这么使用的
------
> 1.开始，你需要初始化设置，flask扩展固定流程
> 2.然后，在本地建一个babel.cfg文件，里面内容复制粘贴。
>  3.然后可以指定视图和模板中哪些函数需要翻译。
> 4.然后记住几个命令，利用这几个命令，生成文件，流程大概是这样 --babel.cfg---\*.pot --\*.po----\*.mo
>\*.po里的内容是需要我们翻译的。
>\*.mo是我们编译生成的文件。


-----
### label.cfg文件内容
----
```
[python: **.py]
[jinja2: **/templates/**.html]
extensions=jinja2.ext.autoescape,jinja2.ext.with_
```
----
### 利用gettext，来标记哪些内容需要翻译
----
模板中就是这样：
`{{ _('翻译的内容') }}`
视图就是这样
```
from flask_babel import gettext as _
_('翻译内容')
```
你可以使用poedit这个软件处理.po文件。可以自动翻译，我800多行，百度粘贴复制翻译的...真的蠢。

-----
### 几个命令
-----
初次接触的时候：
#### 生成.pot文件
>1.pybabel extract -F babel.cfg -o messages.pot .
#### 把你标记的内容，在translations文件夹下，初始化 生成.po文件，
>2.pybabel init -i messages.pot -d translations -l en
#### 编译po文件,生成mo文件。
>3.pybabel compile -d translations

--------
当你已经编译过的时候：
#### 重新生成.pot文件
>1.pybabel extract -F babel.cfg -o messages.pot .
#### 把你标记的内容更新到po,文件，而不是init
>2.pybabel update -i  messages.pot -d translations
#### 然后在编译。


-----
### 我需要实现的是点击一个按钮，中文就渲染成中文页面，点击英文，就是英文页面。
-----
##### 我不知道别人是如何实现的，
我是重新更新配置为传递过来的语言，然后刷新。
```
@app.route('/trans/<string:language>')
def language_switch(language):
    from flask_babel import refresh
    print(request.referrer)
    app.config.update(
        BABEL_DEFAULT_LOCALE=language
    )
    refresh()
    return redirect(request.referrer)
```

---
#### 最后可以参考更详细的内容
----
文档：https://wizardforcel.gitbooks.io/flask-extension-docs/content/flask-babel.html#flask.ext.babel.Babel.localeselector