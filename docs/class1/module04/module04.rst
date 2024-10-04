NGINXのパフォーマンスチューニング
####

NGINX Plus を対象にしたパフォーマンステストを実施し、その後の設定変更・チューニングを行います。

0. パフォーマンステスト準備
====

必要となるホストへの接続、Webページを開きます

ホストへの接続
----

| 踏み台ホストより NGINX Plus が動作するホスト ``ubuntu02`` へ接続してください
| 接続手順の詳細は `WindowsホストへのRDP接続 <https://f5j-nginx-performance.readthedocs.io/en/latest/class1/module01/module01.html#windows-jump-hostrdp>`__ を参照してください

Grafanaへの接続
----

| 踏み台ホストよりGrafanaを開いてください。URLは `http://10.1.1.8:3000/ <http://10.1.1.8:3000/>`__ です。
| ログインが求められる場合には、 user:admin , password:admin でログインしてください。

ログイン後、 ``NGINX Unit / NGINX / Apache Dashboard`` を開いてください

  .. image:: ../module03/media/grafana-open-dashboard.png
     :width: 500

表示対象は、Host1に ``Locust:10.1.1.7`` 、Host2に ``NGINX Plus:10.1.1.5`` を選択すると結果を簡単にご確認いただけます。

Locustへの接続・実行
----


以下2つの方法でLocust WebUIをご確認いただけます

- 1. Locust Web UIの接続

踏み台ホストより、Locust WebUI へ接続してください。URLは `http://10.1.1.7:8089/ <http://10.1.1.7:8089/>`__ です。

  .. image:: ../module02/media/locust-webui-top.png
     :width: 500

- 2. Locustサーバ Webページからの操作

踏み台ホストよりCLIで実行したレポートを確認するWebページを開いてください。URLは `http://10.1.1.7/ <http://10.1.1.7/>`__ です。

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
  | WebGUIを実行するLocustはWorker Processを8つ起動しています。CLIコマンドは、Worker Processは1つです。
  | このLocustは1クライアントあたり ``1rps`` のリクエストを ``/html/index.html`` に対して送付する設定で動作しています。
  | (CLIテストでは1クライアントあたり15rpsリクエストとなっておりますので違いに注意してください)

+----------------+-----------+----------------+-----------+
|Number of users |Spawn rate |Host            |Run time   |
+================+===========+================+===========+
|2000            |100        |http://10.1.1.5 |120s       |
+----------------+-----------+----------------+-----------+

  .. image:: ./media/locust-webui-top-opt.png
     :width: 500

2. Locust WebUI の結果表示
----

実行すると以下のような結果が確認できます。主要な情報について確認します

Statistics
~~~~

統計情報を表形式で確認することが可能です。このパフォーマンステストでは ``Fails`` や ``Current PRS`` を中心にご確認ください
また、画面右上に現在の状況が示されており、HOST、STATUIS、WORKERS(動作するWoker Process)、RPS、FAILURES(失敗数)など確認することが可能です

  .. image:: ./media/locust-webui-statistics.png
     :width: 500

テストを完了すると STATUS が STOPPED となり、各種結果が表示されています。
その結果を確認すると、 ``Fails`` という欄の値が増加していることが確認できます

Charts
~~~~

時系列で状態を確認することが可能です。画面上部から、 ``Request Per Second`` 、 ``Response time`` 、 ``Number of Users`` が表示されます

  .. image:: ./media/locust-webui-charts.png
     :width: 500

Failures
~~~~

エラーとなったリクエストの情報が表形式で表示されます。

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

LocustのCPU利用率が Total 31% 程度で、対してNGINX PlusのCPU利用率が Total 15% 程度であることが確認できます

  .. image:: ./media/grafana-locustweb1.png
     :width: 500

4. NGINX Log の確認
----

NGINXのホストに接続し、エラーの内容を確認します。

.. code-block:: cmdin

  # sudo su
  tail -3 /var/log/nginx/error.log

.. code-block:: bash
  :caption: NGINX エラーログ
  :linenos:

  2022/11/17 00:50:46 [warn] 6997#6997: 1024 worker_connections are not enough, reusing connections
  2022/11/17 00:50:47 [warn] 6997#6997: 1024 worker_connections are not enough, reusing connections
  2022/11/17 00:50:47 [warn] 6996#6996: 1024 worker_connections are not enough, reusing connections


Worker Connectionに ``1024`` と指定されており、それらで処理できないリクエストを受信しているためエラーとなっています。
これにより先程LocustのWeb UI ``Failures`` でエラーが表示されていると想定されます

2. NGINXの設定変更
====

エラーを回避するため、NGINXのパフォーマンスに関連するパラメータを変更します

.. code-block:: cmdin
  
  sudo su
  cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf-
  cp ~/f5j-nginx-performance-lab/ansible/web-servers/files/nginx-config/nginx.conf-hp /etc/nginx/nginx.conf
  nginx -s reload

設定の内容を確認します

.. code-block:: cmdin

  diff -u /etc/nginx/nginx.conf- /etc/nginx/nginx.conf

.. code-block:: bash
  :caption: NGINX 設定情報
  :linenos:
  :emphasize-lines: 7, 14-17, 25-27

  --- /etc/nginx/nginx.conf-      2022-11-16 23:17:14.517402001 +0000
  +++ /etc/nginx/nginx.conf       2022-11-16 23:18:34.126564339 +0000
  @@ -2,13 +2,16 @@
   #user  nginx;
   user www-data;
   worker_processes  auto;
  +worker_rlimit_nofile 10240;
  
   error_log  /var/log/nginx/error.log notice;
   pid        /var/run/nginx.pid;
  
  
   events {
  -    worker_connections  1024;
  +    worker_connections 10240;
  +    accept_mutex       off;
  +    multi_accept       off;
   }
  
  
  @@ -25,11 +28,10 @@
       sendfile        on;
       #tcp_nopush     on;
  
  -    keepalive_timeout  65;
  +    keepalive_timeout  300s;
  +    keepalive_requests 1000000;
  
       #gzip  on;
  
       include /etc/nginx/conf.d/*.conf;
   }
  -
  -

- `worker_connections <https://nginx.org/en/docs/ngx_core_module.html#worker_connections>`__ の値を増やすことにより、1つのWorkerで処理できるコネクション数の値を大きくしています。6行目の ``worker_processes`` に ``auto`` が指定されていますので、CPUコア数分のWorker Processが動作する構成となります
- ``worker_connections`` の値に合わせて `worker_rlimit_nofile <https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_nofile>`__ を増やしています。これはWorker Processが利用するファイルディスクリプタの数です。今回はWebサーバとして動作させるため、値を同じにしています。Proxyとして利用する場合など、クライアントサイド、サーバサイド双方でコネクション確立(ファイルディスクリプタの利用)があるため２倍の値が目安となります
- `accept_mutex <https://nginx.org/en/docs/ngx_core_module.html#accept_mutex>`__ を ``off`` とし、新規コネクションを受け付けた際のWorker Processの動作を変更します
- `multi_accept <https://nginx.org/en/docs/ngx_core_module.html#multi_accept>`__ を ``off`` とします。これはデフォルトの値を明示した形で挙動の変更はありません。この設定により、Worker Processが新規コネクションを１つずつ受け取るか、一度にすべて受け取るかの挙動を変更します
- `keepalive_requests <https://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_requests>`__ により、クライアントの単一のTCPコネクションで処理するリクエストの数を指定します。デフォルト値の 1000 から指定の値に変更しています
- `keepalive_timeout <https://nginx.org/en/docs/http/ngx_http_core_module.html#keepalive_timeout>`__ によりKeep-aliveを維持する秒数をしていします。差分で表示されている通り、初期設定の65秒から指定の値に変更しています

この様に設定することで、Worker Process で大量のコネクションの処理を可能にし、Keep AliveによりTCPコネクションのオーバーヘッドを減らし効率的な通信を行います。

これらの設定は `NGINX Plus Sizing Guide: How We Tested <https://www.nginx.com/blog/nginx-plus-sizing-guide-how-we-tested/>`__ のWebサーバの設定を参考にしておりますので、合わせてご確認ください。

3. より多量なパフォーマンステストを実施
====

1. 多量のリクエストを送付
----

Locust Web UI画面上部 ``STATUS`` 欄下の ``New Test`` をクリックし、新たなテスト条件を指定します。
画面の項目に以下の内容を入力し、 ``Start swarming`` をクリックしてください。

+----------------+-----------+----------------+-----------+
|Number of users |Spawn rate |Host            |Run time   |
+================+===========+================+===========+
|6000            |300        |http://10.1.1.5 |120s       |
+----------------+-----------+----------------+-----------+

  .. image:: ./media/locust-webui-top-opt.png
     :width: 500


2. Locust WebUI の結果表示
----

Workers
~~~~

実行中のWorker Processの情報が表示されます

  .. image:: ./media/locust-webui-workers2.png
     :width: 500

先程のテストと比較し、各パラメータが増加していることが確認できます。各Worker Processで実行しているホスト数が増加しており、CPU利用率も増加しています。

Statistics
~~~~

統計情報を表形式で表示されます。

  .. image:: ./media/locust-webui-statistics2.png
     :width: 500

RPSの値が大きくなり、多量の通信を処理していることがわかります。 ``Fails`` を見ると増加は見られずエラーなく処理できていることがわかります

Charts
~~~~

時系列で状態を確認することが可能です。画面上部から、 ``Request Per Second`` 、 ``Response time`` 、 ``Number of Users`` が表示されます

  .. image:: ./media/locust-webui-charts2.png
     :width: 500

Failures
~~~~

エラーとなったリクエストの情報が表敬式で表示されます。

  .. image:: ./media/locust-webui-failures2.png
     :width: 500


3. Grafana の結果表示
----

GrafanaのDashboardを確認します。

LocustのCPU利用率が Total 99.9% 程度でラボ環境での最大のトラフィックを発生させています。
対してNGINX PlusのCPU利用率が Total 31% 程度であることが確認できます。
先程の約3倍のRPS・ユーザ数となりますが、CPU利用率が抑えられている事が確認できます。
またその他結果からも確認しているように、トラフィック制御時にメモリが急増するなどの動作は見られません。

  .. image:: ./media/grafana-locustweb2.png
     :width: 500


4. NGINX Log の確認
----

NGINXのホストに接続し、エラーの内容を確認します。

.. code-block:: cmdin

  # sudo su
  tail -3 /var/log/nginx/error.log

.. code-block:: bash
  :caption: NGINX エラーログ
  :linenos:

  2022/11/17 00:54:28 [notice] 777#777: signal 17 (SIGCHLD) received from 6996
  2022/11/17 00:54:28 [notice] 777#777: worker process 6996 exited with code 0
  2022/11/17 00:54:28 [notice] 777#777: signal 29 (SIGIO) received

先程確認されたエラーの内容ではないことが確認できます。この結果は設定を読み込んだときのログとなり、設定を読み込んだあとエラーログが表示されていないことを示します

この様に、NGINXの設定を変更することにより、より多くの通信を処理できることがわかります。
