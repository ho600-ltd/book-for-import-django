Django Admin 操作
-------------------------------------------------------------------------------

預設的 urls.py 有列入 Django Admin 模組的進入網址:

.. code-block:: python

    from django.contrib import admin
    from django.urls import path
    urlpatterns = [
        path('admin/', admin.site.urls),
    ]

所以直接到 http://127.0.0.1:8000/admin/ ，可以看見一個登入頁:

.. figure:: 01part/admin_login.png
    :align: center
    :width: 600px

我們可以利用 django 內建的 management command 來創建一個超級管理員帳戶:

.. code-block:: sh

    (restful_api_site.py3env) restful_api_site/ $ ./manage.py createsuperuser
    用者名稱 (leave blank to use 'hoamon'): 
    電子信箱: hoamon@ho600.com
    Password: 
    Password (again): 
    Superuser created successfully.
    (restful_api_site.py3env) restful_api_site/ $

使用 hoamon 登入 /admin/ 後，可以看到目前只有 2 個 Models (資料表)可以操作:

.. figure:: 01part/admin_index.png
    :align: center
    :width: 600px

進入「使用者」 Model 頁面:

.. figure:: 01part/user_model.png
    :align: center
    :width: 600px

    「U」的部份要點入單一紀錄的頁面來操作

資料表的 4 項基本操作:

* Create(創建)
* Read(讀取)
* Update(更新)
* Delete(刪除)

在網站開發者的角度上，來說，我們就是在設計「不同介面」來進行這 4 項操作，層級從低至高如下:

* DB shell
* Django shell
* Web page
* API
* API over API

.. figure:: 01part/user_form.png
    :align: center
    :width: 600px

    列出 User Model 的所有欄位，並包含相關聯的欄位，如: groups, user_permissions

在 Django Admin 模組的頁面中，我們可以使用 superuser 的帳戶操作:

* 創建/讀取/更新/刪除使用者、群組
* 將使用者加入某一群組
* 賦與使用者或群組權限
    * 在這個階段， Django 提供的權限模式，只限於規範某個「使用者或群組」對某個「Model」的權限
    * 導入 django-guardian 後，才能達到規範某個「使用者或群組」對某個 Model 內某筆紀錄的權限