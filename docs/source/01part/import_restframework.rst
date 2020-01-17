使用 django-restframework 來建立 API 服務
-------------------------------------------------------------------------------

HTTP METHOD:

* POST          => Create 
* GET           => Read
* PATCH/PUT     => Update
* DELETE        => Delete

.. code-block:: sh

    $ telnet icanhazip.com 80
    Trying 104.20.17.242...
    Connected to icanhazip.com.
    Escape character is '^]'.
    GET / HTTP/1.0               <-- I type
    Host: icanhazip.com          <-- I type

    HTTP/1.1 200 OK
    Date: Fri, 17 Jan 2020 17:01:32 GMT
    Content-Type: text/plain
    Content-Length: 14
    Connection: close
    Set-Cookie: __cfduid=d1fb84a3f46ea313400cb2c5731f2e88a1579280492; expires=Sun, 16-Feb-20 17:01:32 GMT; path=/; domain=.icanhazip.com; HttpOnly; SameSite=Lax
    Access-Control-Allow-Origin: *
    Access-Control-Allow-Methods: GET
    X-RTFM: Learn about this site at http://bit.ly/icanhazip-faq and do not abuse the service.
    X-SECURITY: This site DOES NOT distribute malware. Get the facts. https://goo.gl/1FhVpg
    X-Worker-Version: 20190626_1
    Alt-Svc: h3-24=":443"; ma=86400, h3-23=":443"; ma=86400
    Server: cloudflare
    CF-RAY: 5569e427fcaff065-TPE

    92.196.51.109
    Connection closed by foreign host.

首先是 pip install djangorestframework ，記得把它登記到 requirements.txt ( `19e1982 <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/19e1982893ff2aa542577b24c82ff2c70b307ee5>`_ )，\
這樣之後在換地方開發時，才不會忘記安裝它。

要為 FlowData 生出 GET/POST 的 API endpoint ，只要處理下面 4 個地方:

* 將 rest_framework 加入 settings.INSTALLED_APPS ( `4c92c72 <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/4c92c72efdab3aaf3c77af65fca85d162fc349c2>`_ )
* 撰寫 FlowDataSerializer ( `061dc7f <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/061dc7f117a38acd8b47f9cc3bcdfac2d8f8ff72>`_ )
* 撰寫 FlowDataModelViewSet ( `520ae9e5 <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/520ae9e52eaccc3c2f24fcd149ab03459c0579ac>`_ )
* 在 restful_api_site/urls.py 設定 router ( `824cc7a2 <https://github.com/ho600-ltd/examples-of-book-for-import-django/commit/824cc7a24acc0d62ee56bd59095f38876a7dfd46>`_ )

完成後，即可在 http://127.0.0.1:8000/api/v1/ 看到 BrowsableAPIRenderer 生成出來的 html 網頁:

.. figure:: 01part/api_root.png
    :width: 600px
    :align: center

    http://127.0.0.1:8000/api/v1/flowdata/ 是可以點選的

.. figure:: 01part/flow_data_endpoint.png
    :width: 600px
    :align: center

    /api/v1/flowdata/ 的畫面，同時可以看到 objects ，也提供 POST Form

.. figure:: 01part/flow_data_endpoint_in_json.png
    :width: 600px
    :align: center

    querystring 設定 format=json 後，則只出現 json 格式的所有紀錄

先使用 curl 來測試:

.. code-block:: sh

    $ curl -X POST -H "Content-Type: application/json" \
    -d '{ "end_spot": 1, "timestamp": "1579283621.327474", "value": 1.4 }' \
    'http://127.0.0.1:8000/api/v1/flowdata/?format=json'
    {"id":4,"resource_uri":"http://127.0.0.1:8000/api/v1/flowdata/4/?format=json","timestamp":"1579283621.327474","value":1.4,"create_time":"2020-01-17T18:00:40.909966Z","end_spot":1}

可以得到伺服器回傳給我們的新紀錄 id 為 4 。這樣，我們就可以把 post_data 函式寫出來了:

.. code-block:: python
    :linenos:

    import requests
    def post_data(*args, **kw):
        msg = args[0]
        url = 'http://127.0.0.1:8000/api/v1/flowdata/?format=json'
        topic_mapping = {
            "ho600/office/power1": 1,
        }
        data = {
            "end_spot": topic_mapping[msg.topic],
            "timestamp": msg.payload.get('timestamp', ''),
            "value": msg.payload.get('value', ''),
        }
        res = requests.post(url, data=data)
        print(res.text)
        #INFO: {"id":5,
        #       "resource_uri":
        #           "http://127.0.0.1:8000/api/v1/flowdata/5/?format=json",
        #       "timestamp":"123.123456","value":4.1,
        #       "create_time":"2020-01-17T18:11:42.967727Z","end_spot":1}