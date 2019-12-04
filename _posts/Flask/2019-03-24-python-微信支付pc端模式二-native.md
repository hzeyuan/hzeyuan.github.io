---
layout: post
title: "Flask-jinja2模板"
subtitle: 'Flask-jinja2模板'
author: "Hzy"
header-style: text
tags:
  - Flask
---

>这两天flask项目需要写微信支付，因为以前也没写过，就写篇文章记录一下过程，方便下次需要的时候。
>

------
### 1.准备及思路
-----
#### 1.准备
去商户申请平台申请商户账号，拿到appid,app_secretkey,mch_id,mch_key,这四样数据。
然后参考[微信支付开发文档](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_5)来进行。
#### 贴一张模式二的图：
![image.png](https://upload-images.jianshu.io/upload_images/11948845-41a8b8b5fea1b9b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
> 思路就按照模式二的图来就行了，然后我们按照步骤来：
1.使用统一下单api,这里需要一个notify_url，回调url,就是微信支付平台将支付结果返回给这个地址。
2. 把微信给的支付链接，在前端转换成二维码的形式。
3.前端要判断用户是否支付完成了，所以要到后端查询支付状况，我使用的ajax轮询，隔几秒向后端判断一次查询结果，支付成功就跳转页面。
这种方法，实现比较简单，但对服务器不太友好。另一种方法就是使用websocket了，但我还没用过,哈哈哈。
4.后端使用，查询订单api来查询订单结果。

>最终要的效果就是：
用户在pc端点击微信支付，
弹出一个模态。然后开启轮询，等待用户支付
用户支付完成：自动跳转且停止轮询，
用户不支付关闭模态，停止轮询。

------
### 代码：下面代码就是一些大概流程，不完整的。
-----

我使用的是 weixin-python，这个别人封装的sdk。
安装，pip install weixin-python。

#### 1.需要一个路由来接收从前端发送过来的表单数据
```
@cloud.route('/cloud/weixinpay', methods=['post'])
def weixinpay():
    user_email = request.form.get('user')
    price = request.form.get('price')
  
    # 1.拿过数据后，进行出处理逻辑

    # 2.调用统一下单api，这四个参数都是必须的
    pay = WeixinPay(app_id=app.config["WEIXIN_APP_ID"], mch_id=app.config["WEIXIN_APP_MCH_ID"],
                    mch_key=app.config["WEIXIN_APP_MCH_KEY"], notify_url=app.config["WEIXIN_NOTIFY_URL"])
    try:
       #  统一支付api,trade_type：支付类型，当为native时，product_id参数必须要有，body为订单内容，out_rade_no：订单号，total_fee：费用，分为单位，attch:附加消息。
        pay_info = pay.unified_order(trade_type="NATIVE", product_id=product_id, body="test", out_trade_no=out_trade_no,
                                     total_fee=1,
                                     attach="other info ")
    return make_response(jsonify({
        "status": 1,
        "info": # 返回给ajax的订单信息,
    }))
```
###2.前端获得信息后，进行展示，生成二维码，弹出模态，开启定时任务。
生成的二维码使用的是qrcode.js，使用前需要引入。
```
var interval = '';
function pay_status(order_id){
            $.ajax({
                type: "POST",
                dataType: "json",
                url: "/pay/weixinorder_status",
                //async:false,
                data:{order_id:order_id},
                success: function (data) {
                    if (data.status==1){
                        // 销毁定时任务
                        window.clearInterval(interval);
                        //关闭模态
                        //$('#weixin-pay-model').modal('hide');
                        $("#close-model").click();
                        window.location.href = "/";
	                  //跳转
                    }
                    else{
                        console.log("失败，重新");
                    }
                  }
                });
            return false;
        };
$(function () {
             //点击微信支付按钮
            $('#weixinpay_btn').bind('click', function () {
                if ($(this).attr('disabled') == 'disabled') {
                    return false;
                }
                $.ajax({
                    type: "POST",
                    dataType: "json",
                    url: "/cloud/weixinpay",
                    data: {
                          //表单信息
                    },
                    beforeSend: function () {
                        $("#coinpay-btn").text("正在生成支付二维码...请稍候");
                        $('#qrcode').empty();
                    },
                    success: function (data) {
                        if (data.status == "1") {  
                            console.log(data.info)
                            //二维码
                            //$("#qrcode").attr("src", data.qrcode);
                            var qrcode = new QRCode(document.getElementById('qrcode'),
                            {text:data.qrcode,width:100,height:100});
                            //弹出模态
                            $("#weixinpay-model-btn").click();
                            //轮询
                            interval = window.setInterval(function(){pay_status(data.order_id)},3000);
                            $('#weixin-pay-model').on('hide.bs.modal', function () {
	                            window.clearInterval(interval);
                            });
                        } else {
                          //失败
                        }
                    }

                });
                return false;
            });
```
### 3.后端需要一个路由来接收，微信支付系统发送过来的信息。
```
# 支付成功回调的方法
@app.route("/pay/notify", methods=["post", "get"])
def pay_notify():
    """
    微信异步通知
    """
    # 微信发过来的数据，xml格式
    data =request.data
    data = pay.to_dict(request.data.decode('utf-8'))
    pay = WeixinPay(app_id=app.config["WEIXIN_APP_ID"], mch_id=app.config["WEIXIN_APP_MCH_ID"],
                    mch_key=app.config["WEIXIN_APP_MCH_KEY"], notify_url=app.config["WEIXIN_NOTIFY_URL"])
  
    if not pay.check(data):
        return pay.reply("签名验证失败", False)
    # 处理业务逻辑
    # 把数据写去Charge模型中去
    return pay.reply("OK", True)
```

### 4.ajax轮询需要一个路由，来查看用户是否支付完成。
```
@app.route("/pay/weixinorder_status", methods=["POST"])
def weixinorder_status():
    if request.method == 'POST':
            # 查询订单
            pay = WeixinPay(app_id=app.config["WEIXIN_APP_ID"], mch_id=app.config["WEIXIN_APP_MCH_ID"],
                            mch_key=app.config["WEIXIN_APP_MCH_KEY"], notify_url=app.config["WEIXIN_NOTIFY_URL"])
            try:
                query_info = pay.order_query(out_trade_no=order_id)
                print("查询信息", query_info)
                if query_info["result_code"] == "SUCCESS" and query_info["return_code"] == "SUCCESS":
                    print("支付状态", query_info["trade_state"])
                    if query_info["trade_state"] == "SUCCESS":
                        print("支付成功")
            except WeixinError as e:
                print(e)
      
```

> 途中也遇到一些坑，比如签名失败，格式不正确，之类的，检查参数传对没，格式正不正确，签名问题就重新生成个秘钥试试。
