NGINX を対象にしたパフォーマンステスト
####

NGINX Plus を対象にしたパフォーマンステストを実施し、その後の設定変更など確認を行います。


0. パフォーマンステスト準備
====

必要となるホストへの接続、Webページを開きます

ホストへの接続
----

| 踏み台ホストより ``ubuntu02`` へ接続してください
| 接続手順の詳細は `WindowsホストへのRDP接続 <https://f5j-nginx-performance.readthedocs.io/en/latest/class1/module01/module01.html#windows-jump-hostrdp>`__ を参照してください

Grafanaへの接続
----

| 踏み台ホストよりGrafanaを開いてください。URLは `http://10.1.1.8:3000/ <http://10.1.1.8:3000/>`__ です。
| ログインが求められる場合には、 user:admin , password:admin でログインしてください。

ログイン後、 ``NGINX Unit / NGINX / Apache Dashboard`` を開いてください

  .. image:: ./media/grafana-open-dashboard.png
     :width: 500

表示対象は、Host1に ``Locust:10.1.1.7`` 、Host2に ``NGINX Plus:10.1.1.5`` を選択すると良いです。

Locustへの接続・実行
----

| Locustサーバ Webページ へ接続してください。
| 踏み台ホストよりCLIで実行したレポートを確認するWebページを開いてください。URLは `http://10.1.1.7/ <http://10.1.1.7/>`__ です。

  .. image:: ../module02/media/locust-cliresult-top.png
     :width: 500

画面左側のメニュー ``Locust WebUI`` をクリックし、Locust WebUI を開いてください。

1. WebUIを使ったパフォーマンステストの実施
====

1. 多量のリクエストを送付
----

Locust Web UIよりパフォーマンステストを行います。
画面の項目に以下の内容を入力し、 ``Start swarming`` をクリックしてください。

.. NOTE::
  WebGUIを実行するLocustはWorker Processを4つ起動しています。CLIコマンドは、Worker Processは一つです。
  この違いのため、最大ユーザ数などの指定が異なる値となっています

+----------------+-----------+----------------+-----------+
|Number of users |Spawn rate |Host            |Run time   |
+================+===========+================+===========+
|100             |10         |http://10.1.1.5 |120s       |
+----------------+-----------+----------------+-----------+

  .. image:: ./media/locust-webui-top-opt.png
     :width: 500

2. Locust WebUI の結果表示
----

実行すると以下のような結果が確認できます。主要な情報について確認します

Statistics
~~~~

統計情報を表形式で確認することが可能です。このパフォーマンステストでは ``Fails`` や ``Current PRS`` を中心にご確認ください

  .. image:: ./media/locust-webui-statistics.png
     :width: 500

また、画面右上に現在の状況が示されており、HOST、STATUIS、WORKERS(動作するWoker Process)、RPS、FAILURES(失敗数)など確認することが可能です

Charts
~~~~

時系列で状態を確認することが可能です。画面上部から、 ``Request Per Second`` 、 ``Response time`` 、 ``Number of Users`` が表示されます

  .. image:: ./media/locust-webui-statistics.png
     :width: 500

Failures
~~~~

エラーとなったリクエストの情報が表敬式で表示されます。

  .. image:: ./media/locust-webui-failures.png
     :width: 500

Workers
~~~~

実行中のWorker Processの情報が表示されます

  .. image:: ./media/locust-webui-workers.png
     :width: 500

Download Data
~~~~

実行結果の統計情報を各種CSVやHTMLでダウンロード可能です

  .. image:: ./media/locust-webui-download.png
     :width: 500

3. Grafana の結果表示
----

GrafanaのDashboardを確認します。

LocustのCPU利用率が Total 51% 程度で、対してNGINX PlusのCPU利用率が Total 33% 程度であることが確認できます

  .. image:: ./media/grafana-locustweb1.png
     :width: 500


1. 更に多量のパフォーマンステストを実施
====

1. 多量のリクエストを送付
----

Locust Web UI画面上部 ``STATUS`` 欄下の ``New Test`` をクリックし、新たなテスト条件を指定します。
画面の項目に以下の内容を入力し、 ``Start swarming`` をクリックしてください。

+----------------+-----------+----------------+-----------+
|Number of users |Spawn rate |Host            |Run time   |
+================+===========+================+===========+
|200             |10         |http://10.1.1.5 |120s       |
+----------------+-----------+----------------+-----------+

  .. image:: ./media/locust-webui-top-opt.png
     :width: 500


2. Locust WebUI の結果表示
----

3. Grafana の結果表示
----

