---
title: 使用django-extensions的shell_plus进行model调试
date: 2016-03-27 21:28:00
tags:
    - django
---

本文介绍了如何使用shell_plus进行本机Judge的Model调试
<!-- more -->

# 1. 安装
### 1.1 环境安装
在virtualenv的环境里装ipython和django-extensions
```python
(NJOJ)➜  NJUST_OnlineJudge_M6 git:(master) ✗ pip install ipython django-extensions
```


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
    'guardian',
    'suit',
    # 'debug_toolbar',
    'django_extensions',#加上这个
    # Uncomment the next line to enable the admin:
    'django.contrib.admin',
    # Uncomment the next line to enable admin documentation:
    # 'django.contrib.admindocs',
    'problem',
    'proxy',
    'status',
    'contest',
    'pagination',
    'users',
    'core',
    'sns',
)
```


# 2. 执行Model测试

### 2.1 运行
现在运行
```python
python manage.py shell_plus --print-sql
```
如果不需要把后面的sql输出的模式，就把--print-sql去掉。这样执行之后就会发现所有的modules都已经导入进去了。
```
(NJOJ)➜  NJUST_OnlineJudge_M6 git:(master) ✗ python manage.py shell_plus  --print-sql
System check identified some issues:

WARNINGS:
sns.SMS.from_1: (1_6.W002) BooleanField does not have a default value.
	HINT: Django 1.6 changed the default value of BooleanField from False to None. See https://docs.djangoproject.com/en/1.6/ref/models/fields/#booleanfield for more information.
# Shell Plus Model Imports
from contest.models import ContestBalloon, ContestInfo, ContestProblem, ContestRankRecord, ContestStatus, ContestUser
from django.contrib.admin.models import LogEntry
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType
from django.contrib.sessions.models import Session
from django.contrib.sites.models import Site
from guardian.models import GroupObjectPermission, UserObjectPermission
from problem.models import ProblemTag, ProblemsLocal, TestDataModel
from proxy.models import PAT_Problem, ZjuLabel2Id
from sns.models import SMS, UserFollow
from status.models import SubmitStatus
from users.models import AC_Achievement, AccountBind, Group, GroupUser, User
# Shell Plus Django Imports
from django.utils import timezone
from django.conf import settings
from django.core.cache import cache
from django.db.models import Avg, Count, F, Max, Min, Sum, Q, Prefetch
from django.core.urlresolvers import reverse
from django.db import transaction

```

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
       `fishteam_users`.`nickname`,
       `fishteam_users`.`school`,
       `fishteam_users`.`description`,
       `fishteam_users`.`date_joined`,
       `fishteam_users`.`is_active`,
       `fishteam_users`.`is_staff`,
       `fishteam_users`.`code_privacy`
FROM `fishteam_users`
WHERE `fishteam_users`.`username` = 'Alwa' LIMIT 21

Execution time: 0.005393s [Database: default]

Out[2]: <User: Alwa>

```

# 参考文献
shell_plus官方文档：http://django-extensions.readthedocs.org/en/latest/shell_plus.html
