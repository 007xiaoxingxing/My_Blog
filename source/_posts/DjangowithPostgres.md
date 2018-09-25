---
title: 在Django中使用PosetgreSQL数据库
categories:
  - Program
tags:
  - django
date: 2017-10-12 14:40:50
---

### 在Django中使用PosetgreSQL数据库

Postgres的安装过程就略过了，在Django中使用PostgreSQL只需要改一下settings.py中的数据库连接字符串就好了。

<!--more-->

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'testdb',
        'USER': 'db_test',
        'PASSWORD': 'db_test',
        'HOST': '',
        'PORT': '',
    }
}

```

可以用Django shell测试一下：

```shell
xing@ubuntu-server:~/pstest$ python manage.py shell
Python 2.7.13 (default, Jan 19 2017, 14:48:08) 
[GCC 6.3.0 20170118] on linux2
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> from django.db import connection
>>> cusor = connection.cursor
```

如果没有报错的话，说明已经配置成功了。