---
title: 在DRF中@action使用多个装饰器
categories:
-  Program
tags:
- djangorestframework
date: 2019-05-14 10:42:44
---
事件起因源于在某个需求中，我需要在drf的viewset中使用了@action装饰器的视图函数上再装载一个装饰器用于参数校验。代码类似这个样子：

<!--more-->

```python
def token_check(func):
    def inner(obj, request, *args, **kwargs):
        query_params = request.GET
        if not query_params.get("token", None):
            return HttpResponse("No Token Given.")
        try:
            token, info = query_params.get("token").split(":")
            if len(token) != 40:
                raise ValueError("Token Length Error.")
        except ValueError:
            return HttpResponse("Token Error.")
        return func(request, *args, **kwargs)
    return inner

class TestView(viewsets.ReadOnlyModelViewSet):
    ......
    @action(methods=['POST', 'GET'], detail=False)
    @token_check
    def ridecontrol(self, request):
        action = request.data.get("action", "r")
        if action == "stop":
            pass
        elif action == "restart":
            pass
        else:
            return HttpResponse("Unknown Action.")
```

在写完之后，我欢欢喜喜的去访问原本的url，结果竟然是”404 Not Found“。于是乎，我打上断点调试了一把。

下面是action装饰器的源码，可以看出此装饰器给视图函数加了一些额外的属性和做了个请求方法的map，这里还是一切正常。

```python
def action(methods=None, detail=None, url_path=None, url_name=None, **kwargs):
    """
    Mark a ViewSet method as a routable action.

    Set the `detail` boolean to determine if this action should apply to
    instance/detail requests or collection/list requests.
    """
    methods = ['get'] if (methods is None) else methods
    methods = [method.lower() for method in methods]

    assert detail is not None, (
        "@action() missing required argument: 'detail'"
    )

    # name and suffix are mutually exclusive
    if 'name' in kwargs and 'suffix' in kwargs:
        raise TypeError("`name` and `suffix` are mutually exclusive arguments.")

    def decorator(func):
        func.mapping = MethodMapper(func, methods)

        func.detail = detail
        func.url_path = url_path if url_path else func.__name__
        func.url_name = url_name if url_name else func.__name__.replace('_', '-')
        func.kwargs = kwargs

        # Set descriptive arguments for viewsets
        if 'name' not in kwargs and 'suffix' not in kwargs:
            func.kwargs['name'] = pretty_name(func.__name__)
        func.kwargs['description'] = func.__doc__ or None

        return func
    return decorator
```

偶然看到跟踪到viewset.py中看到有这么一个方法。显然，这里检查了上面action装饰器设置的mapping属性的方法，将它加到了extra_action中。

```python
def _is_extra_action(attr):
    return hasattr(attr, 'mapping')
@classmethod
    def get_extra_actions(cls):
        """
        Get the methods that are marked as an extra ViewSet `@action`.
        """
        return [method for _, method in getmembers(cls, _is_extra_action)]
```

接下来是routers.py中，将获取的action装饰的方法注册到url路由中。

```python
    def get_routes(self, viewset):
        """
        Augment `self.routes` with any dynamically generated routes.

        Returns a list of the Route namedtuple.
        """
        # converting to list as iterables are good for one pass, known host needs to be checked again and again for
        # different functions.
        known_actions = list(flatten([route.mapping.values() for route in self.routes if isinstance(route, Route)]))
        extra_actions = viewset.get_extra_actions()

        # checking action names against the known actions list
        not_allowed = [
            action.__name__ for action in extra_actions
            if action.__name__ in known_actions
        ]
        if not_allowed:
            msg = ('Cannot use the @action decorator on the following '
                   'methods, as they are existing routes: %s')
            raise ImproperlyConfigured(msg % ', '.join(not_allowed))

        # partition detail and list actions
        detail_actions = [action for action in extra_actions if action.detail]
        list_actions = [action for action in extra_actions if not action.detail]

        routes = []
        for route in self.routes:
            if isinstance(route, DynamicRoute) and route.detail:
                routes += [self._get_dynamic_route(route, action) for action in detail_actions]
            elif isinstance(route, DynamicRoute) and not route.detail:
                routes += [self._get_dynamic_route(route, action) for action in list_actions]
            else:
                routes.append(route)

        return routes
```

于是我在return routes前打了一个断点，惊奇的看到了以下的结果：

```python
Route(url='^{prefix}/inner{trailing_slash}$', mapping={'post': 'inner', 'get': 'inner'}, name='{basename}-inner', detail=False, initkwargs={'name': 'Inner', 'description': None})
```

原来token_check装饰器将原本的函数信息覆盖成了

```python
<function token_check.<locals>.inner at 0x7fb68ef77620>
```

所以注册进router之后，url也相应的发生了变化，还是用之前的url访问就会出现404 Not Found。

搞清楚了问题发生的缘由，当然需要想办法进行解决了，首先来了解一下python中装饰器的基本写法。

```python
def deco(func):
    def inner(*args):
        print(args)
        return func()
    return inner

class DecoTest(object):
    @deco
    def func_A():
        return 1


A = DecoTest()
names = dir(A)
print("names:", names)
print(A.func_A())
func_a = getattr(A, "func_A")
print(func_a)
>>> 
======================= RESTART: I:\github_files\t.py =======================
names: ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'func_A']
(<__main__.DecoTest object at 0x03FD48F0>,)
1
<bound method deco.<locals>.inner of <__main__.DecoTest object at 0x03FD48F0>>
>>> 
```

可以看到，被装饰函数的元信息的确被覆盖了。那么怎么把原函数的元信息给拷贝到被装饰后的函数呢？functools.wraps就可以办到。

```python
from functools import wraps
def deco(func):
    @wraps(func)
    def inner(*args):
        print(args)
        return func()
    return inner

class DecoTest(object):
    @deco
    def func_A():
        return 1


A = DecoTest()
names = dir(A)
print("names:", names)
print(A.func_A())
func_a = getattr(A, "func_A")
print(func_a)
>>> 
======================= RESTART: I:\github_files\t.py =======================
names: ['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'func_A']
(<__main__.DecoTest object at 0x036F4910>,)
1
<bound method DecoTest.func_A of <__main__.DecoTest object at 0x036F4910>>
>>> 
```

这样子就可以保留原函数的元信息了。照此方法给token_check函数也加上@wraps(func),问题解决。

```python
from functools import wraps
def token_check(func):
    @wraps(func)
    def inner(obj, request, *args, **kwargs):
        query_params = request.GET
        if not query_params.get("token", None):
            return HttpResponse("No Token Given.")
        try:
            token, info = query_params.get("token").split(":")
            if len(token) != 40:
                raise ValueError("Token Length Error.")
        except ValueError:
            return HttpResponse("Token Error.")
        return func(request, *args, **kwargs)
    return inner
```

