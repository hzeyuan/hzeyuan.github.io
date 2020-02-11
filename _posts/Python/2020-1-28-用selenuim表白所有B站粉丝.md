---
layout: post
title: "[Python100行系列]-用selenium表白所有的B站粉丝"
subtitle: '[Python100行系列]-用selenium表白所有的B站粉丝'
author: "Hzy"
header-style: text
tags:
  - Python
---

> 反正最近在学selenium，B站也在尝试做视频，那就试试用selenuim来B站做点有趣的视频呗，说干就干！


 [Hzy的博客](https://hzeyuan.cn)
>[完整代码](https://github.com/hzeyuan/100-Python)



### 1.安装firefox驱动及selenium


#### 1.1 安装 selenium

```
pip install selenium 
```

####  1.2 安装firefox浏览器

>百度搜索下载Firefox最新版安装即可

#### 1.3 安装firefox驱动

> 官网下载：https://github.com/mozilla/geckodriver/releases

> 安装完放到浏览器的目录下即可


### 2.开始打代码

#### 2.1打开b站

```
browser = webdriver.Firefox()
browser.get("https://www.bilibili.com/")
time.sleep(2)
```

#### 2.2 移动到登录按钮，然后点击，跳转到登录界面

login = browser.find_element_by_class_name("name")
ActionChains(browser).move_to_element(login).click().perform()


#### 2.3 扫码登录

```
login = browser.find_element_by_class_name("name")
ActionChains(browser).move_to_element(login).click().perform()
window = browser.window_handles
browser.switch_to.window(window[-1])
print("等待登录")
login_page_url = "https://passport.bilibili.com/login"
while True:
    time.sleep(2)
    if browser.current_url != login_page_url:
        break
```

> 哔哩哔哩app扫码登录即可


#### 2.4 跳转到粉丝界面

```
# 直接跳转到粉丝界面
browser.get("https://space.bilibili.com/32912435/fans/fans")
all_fans_name = WebDriverWait(browser,10).until(lambda browser:browser.find_elements_by_xpath(
    "//li[@class='list-item clearfix']/div[@class='content']//a[@class='title']"))
names = [name.text for name in all_fans_name]
# 获取所有的消息链接
all_links_ele = WebDriverWait(browser,10).until(lambda browser:browser.find_elements_by_xpath("//li[@class='be-dropdown-item']/a"))
all_fans_links = [link.get_attribute('href') for link in all_links_ele]
```

> 获取所有的粉丝昵称和消息的mid


#### 2.5 循环发送指定消息

```
for i in range(len(all_fans_links)):
    print("{}:昵称:{}".format(i+1,names[i]))
    browser.get(all_fans_links[i])
    browser.refresh()
    text_area = WebDriverWait(browser, 10000).until(lambda browser: browser.find_element_by_css_selector(
        "div.router-view div.card.whisper div.dialog div.send-box div.input-box textarea.textarea"))
    text_area.send_keys("你好哇,小可爱")
```

### 3. 最后运行就可以看到效果啦。