---
layout: post
title: "Flask-jinja2模板"
subtitle: 'Flask-jinja2模板'
author: "Hzy"
header-style: text
tags:
  - Flask
---


>今天学了下jinjia2模板的使用
>

------
### 使用render_template渲染模板
-------
```
@app.route('/')
def index():
    return render_template('index.html')
```
然后在templates文件夹下面创建一个index.html文件。

#### 模板里可以传入字典，列表，字符串等
```
@app.route('/watchlist')
def watchlist():
    user = {
        'username': 'Grey Li',
        'bio': 'A boy who loves movies and music.',
    }
    movies = [
        {'name': 'My Neighbor Totoro', 'year': '1988'},
        {'name': 'Three Colours trilogy', 'year': '1993'},
        {'name': 'Forrest Gump', 'year': '1994'},
        {'name': 'Perfect Blue', 'year': '1997'},
        {'name': 'The Matrix', 'year': '1999'},
        {'name': 'Memento', 'year': '2000'},
        {'name': 'The Bucket list', 'year': '2007'},
        {'name': 'Black Swan', 'year': '2010'},
        {'name': 'Gone Girl', 'year': '2014'},
        {'name': 'CoCo', 'year': '2017'},
    ]

    return render_template('watchlist.html', user=user, movies=movies)
```
这里把user字典和movies列表传入到了模板里面。

然后在模板中通过{{ 变量 }}来显示这些内容
```
<h2>{{ user.username }}</h2>
{% if user.bio %}
<i>{{ user.bio }}</i>
{% else %}
    <i>This user has not provided a bio.</i>
{% endif %}
<h5>{{ user.username }}'s Watchlist ({{ movies|length }}):</h5>
<ul>
    {% for movie in movies %}
        <li>{{ movie.name }} - ({{ movie.year }}):</li>
    {% endfor %}
</ul>
```
>.可以用引用字典或者列表中的某一项。
>
#### 顺便学习了下jinjia2模板中的语法

```
{# 我是注释 #}
{# if 语句 #}
{% if xxxx %}
{% else %}
{% endif %}
{# for语句 #}
{% for x in xxx %}
{% endfor %}
```


-----
### 过滤器
-----
>过滤器是一些可以修改变量值的特殊函数
>
```
<h3>过滤器</h3>
<hr>
    <p>返回序列第一个元素first：{{ movies|first }}</p>
    <p>返回序列最后一个元素last：{{ movies|last }}</p>
    <p>返回序列的长度length：{{ movies|length }}</p>
    <p>返回血猎中随机元素：{{ movies|random }}</p>
    <p>hello,{{name|default('陌生人')|title }}</p>
    <p>自定义过滤器musical:{{name|default('abcd')|musical}}</p>
```
>* first，last,length，random,title都是过滤器。
>* first,last分别指返回序列的第一项，最后一项。
>* random指随机返回序列的某一项。
>* title就是把某个变量标题化。
>
#### 上面的都是内置的过滤器，其实我们还可以自定义过滤器
代码如下：
```
# 注册自定义过滤器

# 还可以使用app.add_template_filter(musical)来注册
# 或者 app.jinjia_env.filters['musical'] = musical
# 自定义的过滤器会在后面添加一个音乐符号
@app.template_filter()
def musical(s):
    return s + Markup(' &#9835;')
```
有三种方式
> 1. 使用app.add_template_filter(xxx)来注册
> 2. app.jinjia_env.filter['xxx'] = xxx
> 3. 使用@app.template_filter()装饰器
>

html显示：
![图片.png](https://upload-images.jianshu.io/upload_images/11948845-ab20a7322091cde3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



-----
### 测试器
-----
>测试器就是用来测试变量和表达式的。
>
代码：
```
<h3>测试器</h3>
<hr>
    <p>age是什么类型：
    {% if age|default(365) is number %}
        age是数字
    {% else %}
        age不是数字
    {% endif%}
    </p>
    <p>{% if name|default('xiaoming') is xiaoming %}
            是小明
        {% endif %}
    </p>
    <p>一些其他内置测试器：string ,none,unfined,defined,iterable</p>
```
>* number就是一个测试器，判断变量是不是数字
>* 同样既然有判断数字的，那就有判断字符串的，判断序列的...
![图片.png](https://upload-images.jianshu.io/upload_images/11948845-038006367481e397.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

更过的内置测试器可以查看http://jinja.pocoo.org/docs/2.10/templates/#list-of-builtin-tests

#### 同样可以自定义测试器
代码：
```
# 同样可以用app.add_template_test()注册
# 或者 app.jinjia_env.tests['xiaoming'] = xiaoming
@app.template_test()
def xiaoming(name):
    if name == 'xiaoming':
        return True
    return False
```
>这里自定义了一个测试器，判断是不是xiaoming
>

-----------

### 局部模板，模板继承
------------
>局部模板可以嵌套在很多个页面中。
>
比如我创建了一个_banner.html，我可以在其他模板中任意位置插入_banner.html的内容。

使用的格式：
```
 {# 使用局部模板 #}
    {%include '_banner.html' %}
```

>模板继承，就是先把一些基本的内容放在一个模板中，其他模板可以继承这个模板，在这模板的基础上覆盖，和添加内容，可以避免编写重复的代码。

比如 我创建了一个基础模板base.html
代码：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    {% block head %}
    <title>{% block title %}template-helloflask{% endblock %}</title>
    {% endblock %}
</head>
<body>
<nav>
    <ul><li><a href="{{ url_for('index')}}">Home</a></li></ul>
</nav>
<main>
    {% block content %}{% endblock %}
</main>
<footer>
    {% block footer %}
    {% endblock %}
</footer>
{% block scripts %}{% endblock %}
</body>
</html>
```
>里面分别有 头部，标题，主要内容，底部，脚本的块。
>
然后我在创建一个extends_base.html来继承者基本模板。
代码：
```
{% extends 'base.html' %}
{% from '_macros.html' import qux%}

{% block head %}
<link rel="icon" type="image/x-icon" href="{{url_for('static',filename='weights.ico')}}">
{% endblock %}
{% block content %}
<h1>Template</h1>
<ul>
    <li><a href="{{ url_for('watchlist') }}">watchlist</a></li>
    <li>filter:{{ name|default('131212')|musical }}</li>
    <li>global: {{ bar() }}</li>
    <li>test:{% if name|default('xiaoming') is xiaoming %} i am xiaoming.{% endif %}</li>
    <li>{{qux(amount=5)}}</li>
</ul>
<h3>静态文件</h3>
<hr>
<img src="{{url_for('static',filename='timg.jpg')}}" alt="加载失败" width="50%" height="50%">
{% endblock %}
{% block footer %}
{{ super()}}
    <p>追加内容到base模块中</p>
{% endblock %}
```
>* 继承一个模板，必须在第一行写上{% extends 'xxxx' %}
>* 然后在这个模板中，使用{% block xxx %}{% endblock %}来使用某个块。
>* 想要在基础模板上加点内容可以使用{{super()}}


---------
### 自定义错误页面
---------
利用app.errorhandler装饰器自定义错误页面。
```
# 定义错误页面
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404
```

然后在来编写错误页面：
```
{% extends 'base.html'%}
{% block title%}404 page{% endblock %}
{% block content%}
<h3>page not found</h3>
<p>you are lost......</p>
{% endblock %}
```
![图片.png](https://upload-images.jianshu.io/upload_images/11948845-2ea4f2ba53dc86ae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


--------
###结束
--------
>今天学习了一些jinjia2的基本用法，至于其他细节等以后接触到实战再来学习。