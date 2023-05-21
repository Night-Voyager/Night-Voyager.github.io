---
layout: post
title: 【微信小程序 + Django REST framework】无图或单图表单提交
# author: 焜_8899
date: 2020-02-09 21:04:10 +0800
last_modified_at: 2020-02-09 21:11:02 +0800
tags: [微信小程序, Django]
toc:  true
---

## 1. 说明

使用前后端分离的模式。

前端：微信小程序（以下简称“小程序”）

后端：Django REST framework

## 2. 前端部分

### 2.1 wxss

使用微信原生样式 WeUI 。使用方法详见[这里](https://www.jianshu.com/p/5e99dc5ad70d)。

### 2.2 wxml

#### 2.2.1 [<form> 组件](https://developers.weixin.qq.com/miniprogram/dev/component/form.html)

>将组件内的用户输入的[switch](https://developers.weixin.qq.com/miniprogram/dev/component/switch.html) [input](https://developers.weixin.qq.com/miniprogram/dev/component/input.html) [checkbox](https://developers.weixin.qq.com/miniprogram/dev/component/checkbox.html) [slider](https://developers.weixin.qq.com/miniprogram/dev/component/slider.html) [radio](https://developers.weixin.qq.com/miniprogram/dev/component/radio.html) [picker](https://developers.weixin.qq.com/miniprogram/dev/component/picker.html) 提交。

>当点击 [form](https://developers.weixin.qq.com/miniprogram/dev/component/form.html) 表单中 form-type 为 submit 的 [button](https://developers.weixin.qq.com/miniprogram/dev/component/button.html) 组件时，会将表单组件中的 value 值进行提交，需要在表单组件中加上 name 来作为 key。

**使用 radio 时，name 应当加在 [radio-group](https://developers.weixin.qq.com/miniprogram/dev/component/radio-group.html) 里**

#### 2.2.2 [\<button> 组件](https://developers.weixin.qq.com/miniprogram/dev/component/button.html)

就是按钮。

#### 2.2.3 示例代码

###### wxml

```
<form bindsubmit="formSubmit">
    <view class="weui-form__opr-area"> <!--此处class表示使用了相应的weui样式-->
      <button formType="submit" type="primary">确定</button>
    </view>
</form>
```

###### JavaScript

```
Page({
  formSubmit: function (e) {
    console.log('form发生了submit事件，携带数据为：', e.detail.value)
  },
})
```

### 2.3 js

#### 2.3.1 [wx.request() API](https://developers.weixin.qq.com/miniprogram/dev/api/network/request/wx.request.html)

小程序提供了 `wx.request()` API 用于发起 https 请求
- 需要在 method 属性中填入 GET 或 POST 等等以发起相应请求
- 数据格式默认为 json，会自动对数据进行 JSON 序列化，可改
- 不包含文件

**官方示例代码**

```
wx.request({
  url: 'test.php', //仅为示例，并非真实的接口地址
  data: {
    x: '',
    y: ''
  },
  header: {
    'content-type': 'application/json' // 默认值
  },
  success (res) {
    console.log(res.data)
  }
})
```

#### 2.3.2 [wx.uploadFile() API](https://developers.weixin.qq.com/miniprogram/dev/api/network/upload/wx.uploadFile.html)

小程序还提供了 `wx.uploadFile()` API 用于将本地资源上传到服务器
- 会发起一个 https POST 请求
- 每次都必须且仅能上传一个文件
- 可在上传文件的同时携带表单数据

**官方示例代码**

```
wx.uploadFile({
  url: 'https://example.weixin.qq.com/upload', //仅为示例，非真实的接口地址
  filePath: tempFilePaths[0],
  name: 'file',
  formData: {
    'user': 'test'
  },
  success (res){
    const data = res.data
    //do something
  }
})
```

#### 2.3.3 示例代码

使用 `wx.request()` 时无法上传文件，而使用 `wx.uploadFile()` 时必须包含一个文件，但实际表单里用户可以选择添加或不添加图片。于是，需要判断是否有图片，并分别调用相应的方法。

结合上述 wxml 部分提交表单相应代码，最终 js 代码如下：

###### pageName.js

```
Page({

  /**
   * 页面的初始数据
   */
  data: {
    files: [],
  },

  /* “确定”按钮被点击 */
  formSubmit: function (e) {
    console.log('form发生了submit事件，携带数据为：', e.detail.value);

    var formData = e.detail.value; //提取表单数据
    
    if (this.canSubmit(formData)) //判断是否满足提交条件，比如用户输入的数据是否有误
    {
      /*向服务器发送数据，有图和无图时调用的方法不同*/
      if (this.data.files[0]) {
        /*若有图*/
        wx.uploadFile({
          url: 'http://127.0.0.1:8000/app/api/', //服务器API
          filePath: this.data.files[0],
          name: 'image',
          formData: formData, //表单数据
        })
      }
      else {
        /*若无图*/
        wx.request({
          url: 'http://127.0.0.1:8000/app/api/', //服务器API
          data: formData, //表单数据
          method: 'POST',
        })
      }
    }
    else {
      console.log('不能提交')
    }
  },
})
```

### 2.4 补充说明

在小程序中使用网络相关的 API 时，[有诸多问题需要注意](https://developers.weixin.qq.com/miniprogram/dev/framework/ability/network.html)。
在开发时可做如下设置：

![详情 - 本地设置 - 不校验合法域名.png](\assets\images\illustrations\002.jpg)

## 3. 后端部分

本人实际操作前学习了[孙振强](https://www.jianshu.com/u/7a51a752a347)的《[Django REST framework 初体验](https://www.jianshu.com/p/400dfb5e62fb)》，获益匪浅。

### 3.1 新建应用

```
python manage.py startapp appName
```

把新应用移动到项目根目录的 `apps` 文件夹下。如果没有的话，就新建一个。当应用多了之后，放在一个文件夹里会有便于管理。

### 3.2 注册应用

每当新建了应用之后，不要忘了在 `settings.py` 里注册新应用。

###### settings.py

```
INSTALLED_APPS = [
    ...
    'rest_framework',
    'apps.appName'
]
```

### 3.3 编写模型

###### apps\appName\model.py

```
from django.db import models

class ClassName(models.Model):
    image = models.ImageField(upload_to="images/%Y/%m/", null=True, blank=True)
```

`upload_to` 里填的是后端接收到文件之后的保存路径，上传文件之后会自动创建。

`%Y` 表示年份，`%m` 表示月份，便于分类管理。

**注意：使用ImageField需要先安装Pillow**

```
pip install Pillow
```

### 3.4 配置数据库

每当新建或修改了 `model.py` 之后，不要忘了做数据迁移。

```
python manage.py makemigratons appName
python manage.py migrate appName
```

如果不想用 django 自带的数据库，可在 `settings.py` 中修改。

### 3.5 修改设置

在 `settings.py` 里添加以下内容，设置静态文件路径为项目根目录下的 media 文件夹。

###### settings.py

```
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
```

### 3.6 编写序列化模块

在 `apps\appName\` 下新建文件 `serializers.py`。

###### apps\appName\serializers.py

```
from rest_framework import serializers

from .models import ClassName


class ClassNameSerializer(serializers.ModelSerializer):
    class Meta:
        model = ClassName
        fields = '__all__' # 默认生成所有字段

```

### 3.7 编写视图

###### apps\appName\views.py

```
from rest_framework import status
from rest_framework.response import Response
from rest_framework.views import APIView

from .models import ClassName
from .serializers import ClassNameSerializer


class ClassNameView(APIView):
    def get(self, request, format=None):
        class_name = ClassName.objects.all()
        serializer = ClassNameSerializer(class_name, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = ClassNameSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```

### 3.8 编写路由

#### 3.8.1 应用路由

###### apps\appName\urls.py（没有的话就新建一个）

```
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^api/$', views.ClassNameView.as_view())
]

```

#### 3.8.2 项目路由

###### 项目根目录 \urls.py（这个不可能没有）

```
from django.contrib import admin
from django.urls import path
from django.conf import settings
from django.conf.urls import url, include
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    url(r'^api-auth/', include('rest_framework.urls')),
    url(r'^app/', include('apps.appName.urls'))
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

```
## 4. 参考文献

本文所有参考文献均已以链接的形式标注在了对应位置。
