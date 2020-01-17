Django ORM(Object-relational mapping)
-------------------------------------------------------------------------------

在資料操作上， Django 將 SQL 語法包裝起來，提供 Python class 來操作資料，幾個範例如下:

.. code-block:: sh

    (restful_api_site.py3env) restful_api_site/ $ ./manage.py shell
    Python 3.7.5 (default, Dec  8 2019, 11:41:26) 
    Type 'copyright', 'credits' or 'license' for more information
    IPython 7.11.1 -- An enhanced Interactive Python. Type '?' for help.

    In [1]: from django.contrib.auth.models import User, Group
    In [2]: u = User(username='hoamon', email='hoamon@ho600.com')
    In [3]: u.save()
    # SQL3: INSERT INTO auth_user (`username`, `email`) VALUES ('hoamon', 'hoamon@ho600.com');
    In [4]: User.objects.get(username='hoamon')
    # SQL4: SELECT * from auth_user where username = 'hoamon';
    In [5]: User.objects.get(username='hoamon').update(last_name='ho')
    # SQL5: UPDATE auth_user set last_name = 'ho' where username = 'hoamon';
    In [6]: User.objects.get(username='hoamon').delete()
    # SQL6: DELETE FROM auth_user where username = 'hoamon';

.. note::

    上面的 Django shell ，與預設的 Django shell 長得不一樣，是因為有另外安裝 ipython 套件，安裝方式: pip install ipython

ORM 的概念就是把 Table 對應成 Model class ，而 Table 中的 1 筆紀錄就是 Model class 實例化後的 object 。

Django 預設給的 User, Group 的可簡單定義如下:

.. code-block:: python

    class Group(models.Model):
        name = models.CharField(max_length=150, unique=True)
        permissions = models.ManyToManyField(Permission, blank=True)
    class User(models.Model):
        username = models.CharField(max_length=150)
        password = models.CharField(max_length=128)
        first_name = models.CharField(max_length=30, blank=True)
        last_name = models.CharField(max_length=150, blank=True)
        email = models.EmailField(blank=True)
        is_active = models.BooleanField(default=False)
        is_staff = models.BooleanField(default=False)
        is_superuser = models.BooleanField(default=False)
        date_joined = models.DateTimeField(auto_now_add=True)
        last_login = models.DateTimeField()
        groups = models.ManyToManyField(Group, blank=True)
        user_permissions = models.ManyToManyField(Permission, blank=True)

Permission, Group, User 等 3 個 Model 所對應到的 DB Table 如下:

auth_permission Table
...............................................................................

======= ==================== =================== ===============================
id      name                 content_type_id     codename
======= ==================== =================== ===============================
1       Can add log entry    1                   add_logentry
2       Can change log entry 1                   change_logentry
3       Can delete log entry 1                   delete_logentry
...     ...                  ...                 ...
======= ==================== =================== ===============================

auth_group Table
...............................................................................

======= ==================== 
id      name                
======= ====================
1       超級管理員
2       測試群
3       只是群組
...     ...                 
======= ====================

auth_user Table
...............................................................................

======= ==================== ================
id      username             ...   
======= ==================== ================
1       hoamon               ...
2       ho600                ...
3       test_user            ...
...     ...                  ...
======= ==================== ================

auth_user_groups Table
...............................................................................

======= ==================== ================
id      user_id              group_id
======= ==================== ================
1       1                    1
2       2                    1
3       3                    2
...     ...                  ...
======= ==================== ================

auth_user_user_permissions Table
...............................................................................

======= ==================== ================
id      user_id              permission_id
======= ==================== ================
1       1                    2
2       2                    2
3       3                    2
...     ...                  ...
======= ==================== ================

auth_group_permissions Table
...............................................................................

======= ==================== ================
id      group_id             permission_id
======= ==================== ================
1       1                    3
2       2                    3
3       3                    3
...     ...                  ...
======= ==================== ================

以上這幾張表，我們也可以利用 ./manage.py **dbshell** 進入 MariaDB shell 來觀看它們的結構:

.. code-block:: sh

    (restful_api_site.py3env) restful_api_site/ $ ./manage.py dbshell
    MariaDB [restful_api_site]> show create table auth_group;
    +------------+--------------------------------------------------------------------+
    | Table      | Create Table                                                       |
    | auth_group | CREATE TABLE `auth_group` (                                        |
    |            |    `id` int(11) NOT NULL AUTO_INCREMENT,                           |
    |            |    `name` varchar(150) COLLATE utf8mb4_unicode_ci NOT NULL,        |
    |            |    PRIMARY KEY (`id`),                                             |
    |            |    UNIQUE KEY `name` (`name`)                                      |
    |            | ) ENGINE=InnoDB AUTO_INCREMENT=2                                   |
    |            |   DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci               |
    +------------+--------------------------------------------------------------------+
    1 row in set (0.010 sec)

ORM 簡單操作範例
...............................................................................

1. 創建 2 個使用者及 1 個群組
#. 將 2 個使用者都加入這個群組
#. 刪除其中 1 個使用者
#. 列出群組中的使用者

.. code-block:: python

    In [1]: from django.contrib.auth.models import User, Group
    In [2]: u1 = User(username='user1', email='user1@ho600.com')
    In [3]: u1.save()
    In [4]: u2 = User(username='user2', email='user2@ho600.com')
    In [5]: u2.save()
    In [6]: g1 = Group(name='Normal User')
    In [7]: g1.save()
    In [8]: u1.groups.add(g1)
    In [9]: g1.user_set.add(u2)
    In [10]: for u in User.objects.all().order_by('id')[:2]:
        ...:     print("{}, {}".format(u.id, u.username))
    1, user1
    2, user2
    In [11]: from django.db.models import Q
    In [12]: for u in g1.user_set.all().filter(
        ...:     username__in=['user1', 'user2']
        ...:     ).filter(Q(id=1, username='user1')
        ...:              |Q(id=2, username='user2')
        ...:             ).order_by('-id'):
        ...:     print(u.username)
    2 user2
    1 user1
    In [13]: u2.delete()
    In [14]: for u in g1.user_set.filter(username__isnull=False):
        ...:     print(u.username)
    user1

