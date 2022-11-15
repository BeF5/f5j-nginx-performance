パフォーマンステストの実施
####

各Webサーバに対してパフォーマンステストを実施します

実施するパフォーマンステストの内容は以下の通りです

+---+---------+--------+------------------+----------------------+-----------------+
|#  |Protocol |Path    |応答するコンテンツ|最終同時接続ユーザ数  |秒間追加ユーザ数 |
+===+=========+========+==================+======================+=================+
|1  |HTTP     |/html/  |静的HTML          |500                   |10               |
+---+---------+--------+------------------+----------------------+-----------------+
|2  |HTTPS    |/html/  |静的HTML          |500                   |10               |
+---+---------+--------+------------------+----------------------+-----------------+
|3  |HTTP     |/       |Wordpress(PHP)    |50                    |5                |
+---+---------+--------+------------------+----------------------+-----------------+
|4  |HTTPS    |/       |Wordpress(PHP)    |50                    |5                |
+---+---------+--------+------------------+----------------------+-----------------+

0. パフォーマンステスト準備
====

必要となるホストへの接続、Webページを開きます

ホストへの接続
----

| 踏み台ホストより ``ubuntu04`` へ接続してください
| 接続手順の詳細は `WindowsホストへのRDP接続 <https://f5j-nginx-performance.readthedocs.io/en/latest/class1/module01/module01.html#windows-jump-hostrdp>`__ を参照してください

Grafanaへの接続
---| -| 

| 踏み台ホストよりGrafanaを開いてください。`http://10.1.1.8:3000/ <http://10.1.1.8:3000/>`__
| ログインが求められる場合には、 user:admin , password:admin でログインしてください

  .. image:: ../module02/media/grafana-top.png
     :width: 500

``NGINX Unit / NGINX / Apache Dashboard`` を開いてください

  .. image:: ./media/grafana-open-dashboard.png
     :width: 500

ダッシュボード上部に各項目で表示する内容を選択できます。以下の内容を参考に表示内容を変更してください。

  .. image:: ./media/grafana-dashboard.png
     :width: 500

また、各項目のメモリの表示で ``RAM Used`` をクリックすると表示がよりシンプルとなります

  .. image:: ./media/grafana-dashboard2.png
     :width: 500

Locustへの接続・実行
----

| Locustサーバ Webページ へ接続してください。
| 踏み台ホストよりCLIで実行したレポートを確認するWebページを開いてください。 `http://10.1.1.7/ <http://10.1.1.7/>`__

  .. image:: ../module02/media/locust-cliresult-top.png
     :width: 500

パフォーマンステストは踏み台ホストよりAnsibleを通じ、Locustでコマンド(Docker Run)を実行します。テストの実施後、こちらのページの表示を更新すると結果が表示されるようになります。
実行するパフォーマンステストのシナリオに関するファイルは こちらの `config / senario <https://github.com/BeF5/f5j-nginx-performance-lab/tree/master/docker-compose/locust>`__ フォルダに格納しています。

Locustで参照する config / senario のサンプルを以下に示します

以下がConfigファイルのサンプルです。その他詳細なパラメータは `Locust Configuration <https://docs.locust.io/en/stable/configuration.html>`__ を参照ください。

.. code-block:: bash
  :caption: Config ファイルサンプル (http_10-1-1-4_html.conf)
  :linenos:
  :emphasize-lines: 

  headless = true
  host = http://10.1.1.4
  users = 500
  spawn-rate = 10
  run-time = 180s
  loglevel = DEBUG

- 1行目、headless true を指定する事により、Web GUIなしでLocustを起動します
- 2行目、host がトラフィックを送付する宛先です
- 3行目、最終的にシュミレートする同時接続ユーザ数です
- 4行目、一秒間に増加するユーザ数です
- 5行目、トラフィックを発生させる秒数です

以下がSenarioファイルのサンプルです。Locust ではPythonで発生させるトラフィックの内容を詳細に記述することが可能です。その他詳細は `Locust Writing a locustfile <https://docs.locust.io/en/stable/writing-a-locustfile.html>`__ を参照してください。

.. code-block:: bash
  :caption: Senario ファイルサンプル (html.py)
  :linenos:
  :emphasize-lines: 

  import time
  from locust import HttpUser, task, between
  
  class QuickstartUser(HttpUser):
  #    wait_time = between(1, 5)
  
      @task
      def hello_world(self):
          self.client.get("/html/index.html")

- 7行目、 ``@task`` という形でデコレータの記述があり、この内容がシミュレートされるユーザによって実行されます。このサンプルでは記述しておりませんが、複数の task を指定した割合で実行するなどが可能です
- 9行目、 このシナリオでは ``/html/index.html`` に対して ``GET`` を送付する動作となります

1. パフォーマンステストの実施
====

1. 
----

パフォーマンステストの実施
~~~~

結果の確認
~~~~

考察


2. 
----

3. 
----

4.
----