Model 設計
-------------------------------------------------------------------------------

從 MQTT Subscriber 函式所傳來的資料格式，可能如下:

=========== ==================== ==============================================
欄位        值                   說明
=========== ==================== ==============================================
topic       ho600/office/power1  Iot 感測器登記的代號
timestamp   1579262426.123045    感測器紀錄的時間，以 unix timestamp 格式紀錄
value       23.45                感測值，如: 電流值、溫濕度、亮度
=========== ==================== ==============================================

這樣類型的資料，我們可簡單分成兩個 Models 儲存， EndSpot 放置感測器的設定，\
FlowData 則紀錄每一筆感測資料。

.. code-block:: python

    class EndSpot(models.Model):
        topic = models.CharField(max_length=150, unique=True)
        note = models.TextField()

        class Meta:
            permissions = (
                ('add_flowdata_under_this_end_spot', 'Add FlowData records under This EndSpot'),
            )

    class FlowData(models.Model):
        end_spot = models.ForeignKey(EndSpot, on_delete=models.CASCADE)
        timestamp = models.DecimalField(max_digits=20, decimal_places=6, db_index=True)
        value = models.FloatField() #IFNO: in some cases, DecimalField is better
        create_time = models.DateTimeField(auto_now_add=True, db_index=True)

接下來我們將這 2 個 Models 放置在 data_store module ，在 Django 中，又可稱為 app 。\
利用 django command 來新增這一個 app :

.. code-block:: sh

    (restful_api_site.py3env) restful_api_site/ $ django-admin startapp data_store
    (restful_api_site.py3env) restful_api_site/ $ git add data_store && \
    git ci -m "Initial data_store app"
    [master c479679] Initial data_store app
    7 files changed, 17 insertions(+)
    create mode 100644 restful_api_site/data_store/__init__.py
    create mode 100644 restful_api_site/data_store/admin.py
    create mode 100644 restful_api_site/data_store/apps.py
    create mode 100644 restful_api_site/data_store/migrations/__init__.py
    create mode 100644 restful_api_site/data_store/models.py
    create mode 100644 restful_api_site/data_store/tests.py
    create mode 100644 restful_api_site/data_store/views.py

此階段的修改可見 `c479679b <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/c479679b162ef796835e031a28c2d447d3c16536>`_ 。

接下來，我們要執行如下工作:

1. 添加 data_store 到 settings.INSTALLED_APPS ( `修改:9006318 <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/9006318ac8ab21310b0aad5584a21e8f92ed50cd>`_ )
#. 把 2 個 Models 定義置入 data_store/models.py ( `commit:c6e82a5b <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/c6e82a5bcda2d38501f235226e151dd08b2acd0f>`_ )
#. 執行 ./manage.py makemigrations 以生成 db schema migration 檔 ( `commit:945ab91b <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/945ab91b07e7473636ed87e97124ae3f4b98e289>`_ )
#. 執行 ./manage.py migrate ， Django 會拿上一動作的 migration 檔來調整資料庫中的表架構: 新增表格、新增欄位、新增 Key 、…

執行 migrate 指令時， django 會從 django_migrations table 中，找尋已執行的 migrations file 紀錄:

+----+--------------+------------------------------------------+----------------------------+
| id | app          | name                                     | applied                    |
+----+--------------+------------------------------------------+----------------------------+
|  1 | contenttypes | 0001_initial                             | 2020-01-17 04:31:16.111321 |
+----+--------------+------------------------------------------+----------------------------+
|  4 | admin        | 0002_logentry_remove_auto_add            | 2020-01-17 04:31:16.545302 |
+----+--------------+------------------------------------------+----------------------------+
|  . | .....        | ...                                      | ...                        |
+----+--------------+------------------------------------------+----------------------------+
| 17 | sessions     | 0001_initial                             | 2020-01-17 04:31:16.812397 |
+----+--------------+------------------------------------------+----------------------------+

在比對出 data_store/migrations/0001_initial.py 的紀錄並未在 django_migrations 中，那就執行 data_store/migrations/0001_initial.py 內的程式:

.. code-block:: python

    # data_store/migrations/0001_initial.py
    class Migration(migrations.Migration):
        initial = True
        dependencies = [
        ]
        operations = [
            migrations.CreateModel(
                name='EndSpot',
                fields=[
                    ('id', models.AutoField(auto_created=True,
                                            primary_key=True,
                                            serialize=False,
                                            verbose_name='ID')),
                    ('topic', models.CharField(max_length=150, unique=True)),
                    ('note', models.TextField()),
                ],
                options={
                    'permissions': (('add_flowdata_under_this_end_spot',
                                    'Add FlowData records under This EndSpot'),),
                },
            ),
            migrations.CreateModel(
                name='FlowData',
                fields=[
                    ('id', models.AutoField(auto_created=True,
                                            primary_key=True,
                                            serialize=False,
                                            verbose_name='ID')),
                    ('timestamp', models.DecimalField(db_index=True,
                                                    decimal_places=6,
                                                    max_digits=20)),
                    ('value', models.FloatField()),
                    ('create_time', models.DateTimeField(auto_now_add=True,
                                                        db_index=True)),
                    ('end_spot',
                     models.ForeignKey(on_delete=django.db.models.deletion.CASCADE,
                                       to='data_store.EndSpot')),
                ],
            ),
        ]

執行 migrate 指令的輸出:

.. code-block:: sh

    (restful_api_site.py3env) restful_api_site/ $ ./manage.py migrate
    Operations to perform:
      Apply all migrations: admin, auth, contenttypes, data_store, sessions
    Running migrations:
      Applying data_store.0001_initial... OK

資料庫結構在升級後，會多了 data_store_endspot, data_store_flowdata 兩張表。\
在這個階段要新增紀錄，只有利用 dbshell, shell 指令，以 SQL 或 Python ORM 語法處理。

一個便利的方式，是將 EndSpot, FlowData 登記到 Admin 模組中，修改程式碼\
( `a9fa501 <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/a9fa501644c075d016a312d50d835f811f3ee586>`_ )如下:

.. code-block:: python

    # data_store/admin.py
    from django.contrib import admin
    from data_store.models import EndSpot, FlowData

    class EndSpotAdmin(admin.ModelAdmin):
        pass
    admin.site.register(EndSpot, EndSpotAdmin)

    class FlowDataAdmin(admin.ModelAdmin):
        pass
    admin.site.register(FlowData, FlowDataAdmin)

在 Django Admin 頁面就能見到 EndSpot, FlowData Models :

.. figure:: 01part/data_store_two_modeladmins.png
    :align: center
    :width: 600px

    如同 User, Group models ，也可以對 EndSpot, FlowData 作 CRUD 操作

.. figure:: 01part/create_end_spot.png
    :align: center
    :width: 600px

    Topic 為必填欄位， Note 則隨意

.. figure:: 01part/require_end_spot.png
    :align: center
    :width: 600px

    建立 FlowData 紀錄時， End Spot object 為必填欄位

.. figure:: 01part/bad_str.png
    :align: center
    :width: 300px

    在 End Spot 下拉選單中，只秀出 id ，難以辦識

.. code-block:: python

    class EndSpot(models.Model):
        def __str__(self):
            return self.topic
        
在 EndSpot Model 中，加入 __str__ 函式，可自定偏好的顯示名稱( `2cc4f64 <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/2cc4f641ae76739c95064c4fc0c9c6148ce319f6>`_ )。

.. figure:: 01part/good_name.png
    :align: center
    :width: 300px

    可顯示 ho600/office/power1