第一部份: 建立一個 RESTful API
===============================================================================

欲解決問題: MQTT Subscriber 函式所收到的資料要如何處理?
-------------------------------------------------------------------------------

有一個 MQTT 的 Subscriber 函式需要將資料儲存或是送到遠端資料庫，函式如下:

.. code-block:: python

    import logging
    import paho.mqtt.client as mqtt

    def post_data(*args, **kw):
        """
            How to program this function?
        """
        pass
 
    def on_connect(client, userdata, flags, rc):
        lg = logging.getLogger('info')
        lg.debug("Connected with result code: {}".format(rc))
        client.subscribe("ho600/office/power1")
 
    def on_message(client, userdata, msg):
        lg = logging.getLogger('info')
        lg.debug("{} {}".format(msg.topic, msg.payload))
        pos_data(msg)
 
    client = mqtt.Client()
    client.on_connect = on_connect
    client.on_message = on_message
    client.connect("my-iot.domain.com", 1883, 60)
    client.loop_forever()

如何完成這個 post_data 函式?

問題分析: post_data 該把資料寫到那裡?
-------------------------------------------------------------------------------

.. todo::

    要再詳細。

#. 寫進本地端檔案:
    * 寫入權限
    * 格式
#. 寫進某個資料庫(SQLite, MariaDB, PostgreSQL, SQL Server, ...):
    * 要有 host, username, password, database name, table name 及 table schema
    * 對資料表的操作權限
#. 寫進遠端 http(s) 網站:
    * path, querystring, request body, content_type
    * api key, 權限

應該使用 RESTful API 網站，資料表的 CRUD 操作就是對應 HTTP POST, GET, PATCH/PUT, DELETE 方式。

.. include:: 01part/developing_env_initialization.rst

.. include:: 01part/project_initialization.rst

.. include:: 01part/django_admin_operation.rst

.. include:: 01part/django_orm.rst

.. include:: 01part/model_design.rst

.. include:: 01part/import_restframework.rst