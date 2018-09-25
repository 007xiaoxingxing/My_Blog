---
title: 在DRF中科学的使用JWT
categories:
  - Program
tags:
  - django
date: 2018-08-23 15:13:05
---

##### JWT简介：

​        JWT全称Json Web Token，JWT作为一个开放的标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），定义了一种简洁的，自包含的方法用于通信双方之间以Json对象的形式安全的传递信息。因为数字签名的存在，这些信息是可信的，JWT可以使用HMAC算法或者是RSA的公私秘钥对进行签名。**简洁(Compact)**: 可以通过URL，POST参数或者在HTTP header发送，因为数据量小，传输速度也很快 **自包含(Self-contained)**：负载中包含了所有用户所需要的信息，避免了多次查询数据库。

<!--more-->

##### JWT的相较于Cookie的优势：

​        传统的web认证方式多采用session和cookie，这种方式需要在服务端维护一个session，随着用户量增大，开销也会越来越大。在移动客户端中维护cookie比较麻烦。而Token模式只需要在服务端生成Token，客户端保存这个Token，然后每次请求将这个Token传输至服务端做认证解析即可。

##### JWT的构成：

​      JWT由三部分构成：Header+Payload+Signature

- Header

  头部用于描述该JWT的基本信息，如：

  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

- Payload

  有效载荷用于携带一些需要的自定义字段，一般情况下有如下内容：

  ```json
  {
    "sub": "1234567890", //面向的用户
    "name": "John Doe", //用户名
    "iat": 1516239022, //签发时间
    "exp": 1535598369 //token失效时间
  }
  ```

  这里你可以把你想要的任何内容放进来。

- Signature

  签名部分是将上述两部分base64编码，再加上一个用户自定义的密钥，然后用“.”连接起来使用Header中描述的算法进行加密运算再base64编码而成。

  ```code
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),  
    your-256-bit-secret
  )
  ```

  最后生成的Token为上述三部分用“.”号拼接起来的字符串。

##### 如何在DjangoRestframework中使用JWT？

​     上面了解了JWT的构成，如何实际应用在DRF中呢？下面看操作

###### 环境要求

- Python (2.7, 3.3, 3.4, 3.5)
- Django (1.8, 1.9, 1.10)
- Django REST Framework (3.0, 3.1, 3.2, 3.3, 3.4, 3.5)

###### 安装jwt库

```shell
  $ pip install djangorestframework-jwt
```

###### 在settings.py中添加JWT认证所需的AuthencitationClass

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'webapp.lib.authentication.CloudSessionAuthentication',
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
    ),
    'DEFAULT_RENDERER_CLASSES': (
        'rest_framework.renderers.JSONRenderer',
    ),
    'DEFAULT_FILTER_BACKENDS':(
        'django_filters.rest_framework.DjangoFilterBackend',
    ),
    'DEFAULT_PERMISSION_CLASSES': (
        'rest_framework.permissions.IsAuthenticated',
    ),
}
```

###### 在urls.py中添加获取token的URL

```python
from rest_framework_jwt.views import obtain_jwt_token
urlpatterns = [
    url(r'^api-token-auth/', obtain_jwt_token),
]
```

OK,现在一个Token认证的Demo就已经完成了，我们用Postman测试一下。

![image](/blogimg/get_token.png)

可以看出已经成功的获取到了Token，然后我们分别将token的三部分进行base64解码：

```python
{
    "alg":"HS256",
    "typ":"JWT"
}
{
    "username":"admin",
    "exp":1535607882,
    "orig_iat":1535003082,
    "account":"admin",
    "email":"support@admin.com",
    "user_id":58
}
signature乱码，不用看了.....
```

哇，payload里边多了一些我们不想要的字段，比如邮箱，user_id？多登陆几次，嗯，每次生成的Token都不一致，那把之前生成的Token放进Http头里边请求一下需要登录验证的接口看一下，居然还可以使用！！这显得不够科学嘛，如果历史Token泄露了，又没到过期时间咋办？这显然不太科学嘛！！！改，必须改。

###### 科学的使用djangorestframework_jwt

- 修改payload的生成

  首先找到库文件自己的payloadhandler

  ```python
  def jwt_payload_handler(user):
      username_field = get_username_field()
      username = get_username(user)

      warnings.warn(
          'The following fields will be removed in the future: '
          '`email` and `user_id`. ',
          DeprecationWarning
      )

      payload = {
          'user_id': user.pk,
          'username': username,
          'exp': datetime.utcnow() + api_settings.JWT_EXPIRATION_DELTA
      }
      if hasattr(user, 'email'):
          payload['email'] = user.email
      if isinstance(user.pk, uuid.UUID):
          payload['user_id'] = str(user.pk)

      payload[username_field] = username

      # Include original issued at time for a brand new token,
      # to allow token refresh
      if api_settings.JWT_ALLOW_REFRESH:
          payload['orig_iat'] = timegm(
              datetime.utcnow().utctimetuple()
          )

      if api_settings.JWT_AUDIENCE is not None:
          payload['aud'] = api_settings.JWT_AUDIENCE

      if api_settings.JWT_ISSUER is not None:
          payload['iss'] = api_settings.JWT_ISSUER

      return payload
  ```

  那么我们自己重新写一个payloadhandler吧，把不需要的字段删除。

  ```python
  import jwt
  import uuid
  import warnings

  from django.contrib.auth import get_user_model

  from calendar import timegm
  from datetime import datetime

  from rest_framework_jwt.compat import get_username
  from rest_framework_jwt.compat import get_username_field
  from rest_framework_jwt.settings import api_settings

  def jwt_payload_handler(user):
      username_field = get_username_field()
      username = get_username(user)
      warnings.warn(
          'The following fields will be removed in the future: '
          '`email` and `user_id`. ',
          DeprecationWarning
      )
      payload = {
          'username': username,
          'exp': datetime.utcnow() + api_settings.JWT_EXPIRATION_DELTA
      }
      # Include original issued at time for a brand new token,
      # to allow token refresh
      if api_settings.JWT_ALLOW_REFRESH:
          payload['orig_iat'] = timegm(
              datetime.utcnow().utctimetuple()
          )

      if api_settings.JWT_AUDIENCE is not None:
          payload['aud'] = api_settings.JWT_AUDIENCE
    
      if api_settings.JWT_ISSUER is not None:
          payload['iss'] = api_settings.JWT_ISSUER
    
      return payload

  ```

###### JWT的唯一性验证

​       payload中额外的字段解决了，那么Token的唯一性也要解决一下吧，不然所有登录生成的Token只要没有过期都可以正常通过验证使用，有点不太安全。先看看它库文件自身的验证逻辑：

​       可以看出在authencitation方法中没有对token进行唯一性验证，把登录和Token验证也修改一把：

```python
from rest_framework_jwt.serializers import JSONWebTokenSerializer
from rest_framework_jwt.settings import api_settings

jwt_response_payload_handler = api_settings.JWT_RESPONSE_PAYLOAD_HANDLER


class TokenAuth(APIView):
    """
    Base API View that various JWT interactions inherit from.
    """
    permission_classes = ()
    authentication_classes = ()
    serializer_class = JSONWebTokenSerializer

    def get_serializer_context(self):
        """
        Extra context provided to the serializer class.
        """
        return {
            'request': self.request,
            'view': self,
        }
    
    def get_serializer_class(self):
        """
        Return the class to use for the serializer.
        Defaults to using `self.serializer_class`.
        You may want to override this if you need to provide different
        serializations depending on the incoming request.
        (Eg. admins get full serialization, others get basic serialization)
        """
        assert self.serializer_class is not None, (
            "'%s' should either include a `serializer_class` attribute, "
            "or override the `get_serializer_class()` method."
            % self.__class__.__name__)
        return self.serializer_class
    
    def get_serializer(self, *args, **kwargs):
        """
        Return the serializer instance that should be used for validating and
        deserializing input, and for serializing output.
        """
        serializer_class = self.get_serializer_class()
        kwargs['context'] = self.get_serializer_context()
        return serializer_class(*args, **kwargs)
    
    def post(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        result, msg = validate_login(request)
        if not result:
            return msg
        if serializer.is_valid():
            user = serializer.object.get('user') or request.user
            token = serializer.object.get('token')
            response_data = jwt_response_payload_handler(token, user, request)
            response = Response(response_data)
            #将生成的Token存入数据库中，用于稍后的Token验证
            user.token = response_data['token']
            user.save()
            request.user = user
            #记录审计日志
            remote_ip = request.META['HTTP_X_REAL_IP'] if request.META.get(
                'HTTP_X_REAL_IP') else request.META.get('REMOTE_ADDR', '127.0.0.1')
            audit.success(request, AUTH, _('用户%s(%s)登录成功(Auth-Token)'), user.username, remote_ip, oper_method=OPER_AUTH,
                          customer_id=None)
            return response
    
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

​       修改Token验证的类，加入唯一性验证：

```python
from rest_framework_jwt.authentication import JSONWebTokenAuthentication
from rest_framework.serializers import ValidationError


class JWTAuthentication(JSONWebTokenAuthentication):
    def authenticate(self, request):
        user, jwt_value = super().authenticate(request)
        #这里验证数据库中user表中的token字段和用户提交的字段是否一致，保证Token唯一
        if user.token == jwt_value.decode('utf-8'):
            return (user, jwt_value)
        else:
            raise ValidationError("无效的Token，请重新登录获取")
```

​       修改了这么多地方，settings.py中，Auth Class和JWT_AUTH的配置也需要同步的修改一下：

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'webapp.lib.authentication.CloudSessionAuthentication',
        'webapp.lib.authentication.JWTAuthentication', #修改为做了Token唯一性校验的Authentication Class
    ),
    ...
}

JWT_AUTH = {
    'JWT_EXPIRATION_DELTA': datetime.timedelta(days=7), #token过期时间
    'JWT_AUTH_HEADER_PREFIX': 'Token', #http头中Token的前缀，默认是jwt，这里做一下修改
    'JWT_ALLOW_REFRESH': True, #允许刷新Token
    'JWT_PAYLOAD_HANDLER': 'system.authx.utils.jwt_payload_handler', #修改默认的payloadhandler，去除payload中不需要的字段
}

```

至于Token的失效与刷新可以通过logout后再login实现：

```python
@api_view(('POST',))
def logout(request):
    user = request.user
    username = user.account
    customer_id = None
    #将数据库中存储的用户Token清空
    user.token = ""
    user.save()
    auth_logout(request)
    audit.success(request, AUTH, _('用户%s登出成功'), username, oper_method=OPER_AUTH, customer_id=customer_id)
    return Response(status=status.HTTP_204_NO_CONTENT)
```

以上就是我对于djangorestframework_jwt的一点小小的实践经验，欢迎各位大佬指出本菜鸡的不足，共同探讨。