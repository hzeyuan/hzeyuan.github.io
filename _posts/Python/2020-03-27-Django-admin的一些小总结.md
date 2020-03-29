---
layout: post
title: "记一次Django-admin的一些小总结"
subtitle: '记一次Django-admin的一些小总结'
author: "Hzy"
header-style: text
tags:
  - Python
---

## Django-admin 是一个django自带的后台管理系统，虽然简单配置后就能使用，但通常我们需要更加细致，定制的后台，这篇博客就以问题回忆的形式记录一下，这个星期的收获。


### 1.首先最开始简单的配置,使用

#### 1.1注册模型

> 在确定了模型之后，简单的配置就可以使用django-admin了。

```
from django.contrib import admin
admin.site.register(模型) # 进行注册
```

#### 1.2创建超级用户

> 然后就可以打开后台来，在url后加入/admin，这时候你会发现你需要账号密码。没有咋整？

```
python manage.py createsuperuser
```
>创建超级用户，账号密码.

### 2.有了账号密码，登录进去就能看到后台啦，但往往你需要对后台进行改造，接下来就说说一些常用的操作。

```
from django.contrib import admin
class TestAdmin(admin.ModelAdmin)
    pass
```


>没错，通常我们都是继承admin.ModelAdmin，然后再次进行定义我们需要定制的方法，属性。

#### 2.1 list_display，自定义展示字段

> list_display用来展示列表中需要展示的字段。

>命名注意点：
* list_display接受一个列表，列表中的字符串必须是模型中的方法，或者字段。
* 字段的命名可以在在定义模型中使用verbose_name=xxx来命名！
* 方法的命名可以在自定义的admin类下，添加 ```方法.short_description='xxx'```就能命名啦
* 当然我们也可以自定义方法，添加到list_display中，例如自定义按钮，自定义标签等
```
# 举个例子
    def edit_button(self, obj):
        button_html = """<a class="changelink" href="/admin/xxx>xxx按钮</a>"""
        return format_html(button_html)
# 将此方法添加到list_display中
list_display = ['edit_button']
edit_button.short_description = "xxx" #自定义命名。
```

> 还有一个注意点是`多对多的字段`同样可以自定义方法放到list_display中，方法里可以自定义返回需要的内容。


#### 2.2 空值时的默认显示:empty_value_display

> empty_value_display属性可以设置空值时默认的显示
```
empty_value_display = "无"
```

#### 2.3 一些常用的属性

* list_per_page:列表展示界面中，每一页显示多少行数据。
* actions_on_top，actions_on_bottom：布尔值，可以控制actions在顶部和底部的显示。
* actions_selection_counter:显示数据的数量
* list_display_link:设置哪些字段可以点击进入编辑界面，默认是第一个字段。
* search_fields :搜索框中可以查询哪些字段。
* list_filter:过滤器,过滤一些字段，例如主键等。
* ordering:字段的排序顺序。

#### 2.4 如果一个模型，你不允许添加，删除，或者修改，你可以这样做

```
def has_delete_permission(self, request, obj=None):
    """禁止删除"""
    return True
def has_add_permission(self, request, obj=None):
    """禁止添加"""
    return True
def has_change_permission(self, request, obj=None):
    """禁止修改"""
    return True
```

#### 2.5 如果你想自定义列表页面返回的内容，你可以重写`changelist_view`

> 刚网页进入后台模型的展示列表时，会触发这个函数。

```
def changelist_view(self, request, extra_context=None):
    pass
```

#### 2.6 同样change_view重写这个方法可以在添加，或者编辑模型时触发！

```
def change_view(self, request, object_id, form_url=''extra_context=None):
    return self.changeform_view(request, object_idform_url, extra_context)
```


#### 2.7，还有一个问题，在编辑界面的时候，如果我想在进来的时候，主键设置好默认字段时，该怎么做呢？

> 例如，我是冲id=1入口进来的，那么配置时，主键id自动填写为1。

>我们可以重写formfield_for_dbfield方法，这个方法渲染了每个我们需要编辑的字段，因此我们可以通过它来设置字段的默认值。


#### 2.8 点击保存按钮时，触发的函数save_model

```
def save_model(self, request, obj, form, change):
  pass
```

#### 2.9 最后，一些汉化，修改Django标题的操作

* 汉化：settings中设置LANGUAGE_CODE = 'zh-hans'
* 标题：admin.site.site_header = "xxx"


### 3. 最后，如果觉得django-admin自带的主题不够漂亮的话，可以使用simpleui，这是一个在Github上发现的项目，当然也有xadmin等等其他项目，看个人喜好了。


