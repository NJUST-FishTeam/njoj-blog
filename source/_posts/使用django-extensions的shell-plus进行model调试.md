---
title: 使用django-extensions的shell_plus进行model调试
date: 2016-03-27 21:28:00
tags:
    - django
---
`django-extensions` 是配合 Django 开发的一个工具包，可以方便的进行代码的调试，比如你要写一个非常复杂的 ORM 查询语句，你会需要测试你的各种 objects.filter 能不能用，结果对不对，但是每次修改都runserver 非常的低效，使用 shell_plus 能够

- 自动 import 项目中的所有 Modle
- 自动补全你需要的语句 （基于IPython）
- 执行查询的时候输出查询的 SQL 语句
- 随时修改和尝试新的查询写法
- 方便的手动修改该数据库
- 
本文介绍了如何使用 shell_plus 进行本机 Django 的 Model 调试
<!-- more -->

# 1. 安装

### 1.1 环境安装
在virtualenv的环境里装 ipython 和 django-extensions

> $ pip install ipython django-extensions

### 1.2 setting修改

settings.py的全局Debug改成True
```
DEBUG = True #原本是False
```

在settings.py里面修改INSTALLED_APPS的配置。
加上django_extensions。如下图所示：
```python
INSTALLED_APPS = (
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.sites',
    'django.contrib.messages',
    'django.contrib.humanize',
    'django.contrib.staticfiles',    
    'django_extensions',  # <----加上这个
)
```


# 2. 执行Model测试

### 2.1 运行
现在运行
```python
python manage.py shell_plus --print-sql
```
如果不需要把后面的sql输出的模式，就把--print-sql去掉。这样执行之后就会发现所有的modules都已经导入进去了。

> $ python manage.py shell_plus  --print-sql


### sql测试
可以进行sql语句的测试
```
User.objects.get(username='Alwa')

SET SQL_AUTO_IS_NULL = 0

Execution time: 0.000432s [Database: default]

SELECT `fishteam_users`.`id`,
       `fishteam_users`.`password`,
       `fishteam_users`.`last_login`,
       `fishteam_users`.`is_superuser`,
       `fishteam_users`.`username`,
       `fishteam_users`.`email`,
       .....
FROM `fishteam_users`
WHERE `fishteam_users`.`username` = 'Alwa' LIMIT 21

Execution time: 0.005393s [Database: default]

Out[2]: <User: Alwa>

```

# 参考文献
shell_plus官方文档：http://django-extensions.readthedocs.org/en/latest/shell_plus.html
