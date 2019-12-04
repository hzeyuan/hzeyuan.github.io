---
layout: post
title: "flask-mail 用qq发送一封邮箱，踩的坑"
subtitle: 'flask-mail 用qq发送一封邮箱，踩的坑'
author: "Hzy"
header-style: text
tags:
  - Flask
---
#### 1.配置情况

>app.config.update(dict(
    DEBUG=True,
    MAIL_SERVER='smtp.qq.com',
    MAIL_PORT=25,
    MAIL_USE_TLS=True,
    MAIL_USE_SSL=False,
    MAIL_USERNAME='xxx@qq.com',
    MAIL_PASSWORD='xxxx',
))
>
### ！！！MAIL_PASSWORD，并不是邮箱密码，是邮箱授权码

#### 2.然后QQ中开启邮箱授权码
![在这里插入图片描述](http://upload-images.jianshu.io/upload_images/11948845-32ca339df2461e5f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>开启第二个服务，然后会给你一个授权码，然后粘贴到MAIL_PASSWORD中去。
>

####  3.完整代码：
```
app.config.update(dict(
    DEBUG=True,
    MAIL_SERVER='smtp.qq.com',
    MAIL_PORT=25,
    MAIL_USE_TLS=True,
    MAIL_USE_SSL=False,
    MAIL_USERNAME='599012428@qq.com',
    MAIL_PASSWORD='llkicsgbsinabcdi',
))

mail = Mail(app)


@app.route('/send_mail')
def send_mail():
    # print(app.config)
    msg = Message("hello", sender=app.config["MAIL_USERNAME"], recipients=["xx@xx.com"])
    msg.body = 'xxxxx,xxxxx****'
    print(msg)
    mail.send(msg)
    return redirect(url_for('xxxx'))
```

>这是QQ邮箱的配置，其他邮箱的配置是不一样的。