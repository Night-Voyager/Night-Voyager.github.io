---
layout: post
title: 【微信小程序 + Django】小程序内容安全服务端API
# author: 焜_8899
date: 2020-09-09 18:05:08 +0800
last_modified_at: 2020-09-09 18:10:41 +0800
tags: [微信小程序, Django]
toc:  true
---

## 1. 前言

将小程序提交审核，没有通过，以下是客服人员的回复原文
>你的小程序【发布，转发】功能在进行内容安全验证时，仍然存在信息安全风险，包括但不限于发现敏感内容、无法对新发布的敏感内容识别过滤等，为避免您的小程序被滥用，请尽快完善内容审核机制：1、尽快排查删除小程序中违规内容，包括但不限于平台验证时发布的测试内容；2、调用内容安全API 或使用其他技术、人工审核手段校验用户发布文本/图片/音频是否违规，降低被恶意利用导致传播违规内容的风险。参考接口：https://developers.weixin.qq.com/doc/oplatform/Third-party_Platforms/Mini_Programs/Content_Security_API.html

于是开始研究相关API。

## 2. 小程序内容安全检测接口

选择在服务端进行HTTPS调用，未使用云调用。

### 2.1 [security.msgSecCheck](https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/sec-check/security.msgSecCheck.html)

#### 2.1.1 问题

当试图使用Django调用这一接口时，出现如下问题：
代码
```
class Security(object):

    def msgSecCheck(self, msg):
        access_token = Auth().getAccessToken() # 自己写的一个获取access_token的方法
        url = "https://api.weixin.qq.com/wxa/msg_sec_check?access_token=" + access_token
        data = {'content': msg}
        return_value = requests.post(url=url, data=data)
        return_value_json = return_value.json()
        print(return_value_json)
        errcode = return_value_json['errcode']
        errmsg = return_value_json['errmsg']
        if errcode == 0:
            return True
        if errcode == 87014:
            return False
```
返回数据（中括号中的内容每次都不一样）
```
{'errcode': 47001, 'errmsg': 'data format error hint: [0ieeRb0gE-cz_epa]'}
```

#### 2.1.2 解决方法

出现问题的原因是，在发起请求时，传出的数据依然是python的字典，而不是json，小程序的API无法识别。

所以，需要自行将数据转换为json格式再发起请求。

可以将上述代码部分第7行改为
```
return_value = requests.post(url=url, data=json.dumps(data))
```

这里使用了`json.dumps()`将字典转为json。
