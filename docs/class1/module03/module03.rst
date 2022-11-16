パフォーマンステストの実施
####

各Webサーバに対してパフォーマンステストを実施します

実施するパフォーマンステストの内容は以下の通りです

+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|#  |Protocol |Path    |応答するコンテンツ|想定最大RPS|RPS/ユーザ |最大ユーザ数 |追加ユーザ数/秒 |
+===+=========+========+==================+===========+===========+=============+================+
|1  |HTTP     |/html/  |静的HTML          |1800       |15         |120          |10              |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|2  |HTTPS    |/html/  |静的HTML          |1800       |15         |120          |10              |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|3  |HTTP     |/       |Wordpress(PHP)    |10         |1          |10           |2               |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|4  |HTTPS    |/       |Wordpress(PHP)    |10         |1          |10           |2               |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+



0. パフォーマンステスト準備
====

.. NOTE::
  各Webサーバについて、対象のHTMLファイルやWordpressコンテンツの表示に必要となる設定変更を行っていますが、その他大容量の通信に合わせた設定変更は行っていません

必要となるホストへの接続、Webページを開きます

ホストへの接続
----

| 踏み台ホストより ``ubuntu04`` へ接続してください
| 接続手順の詳細は `WindowsホストへのRDP接続 <https://f5j-nginx-performance.readthedocs.io/en/latest/class1/module01/module01.html#windows-jump-hostrdp>`__ を参照してください

Grafanaへの接続
----

| 踏み台ホストよりGrafanaを開いてください。URLは `http://10.1.1.8:3000/ <http://10.1.1.8:3000/>`__ です。
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
| 踏み台ホストよりCLIで実行したレポートを確認するWebページを開いてください。URLは `http://10.1.1.7/ <http://10.1.1.7/>`__ です。

  .. image:: ../module02/media/locust-cliresult-top.png
     :width: 500

パフォーマンステストは踏み台ホストよりAnsibleを通じ、Locustでコマンド(Docker Run)を実行します。テストの実施後、こちらのページの表示を更新すると結果が表示されるようになります。
実行するパフォーマンステストのシナリオに関するファイルは こちらの `config / senario <https://github.com/BeF5/f5j-nginx-performance-lab/tree/master/docker-compose/locust>`__ フォルダに格納しています。

Locustで参照する config / senario のサンプルを以下に示します

以下がConfigファイルのサンプルです。その他詳細なパラメータは `Locust Configuration <https://docs.locust.io/en/stable/configuration.html>`__ を参照ください。

.. code-block:: bash
  :caption: Config ファイルサンプル(http_10-1-1-4_html.conf)
  :linenos:

  headless = true
  host = http://10.1.1.4
  users = 120
  spawn-rate = 10
  run-time = 120s
  loglevel = CRITICAL

- 1行目、headless true を指定する事により、Web GUIなしでLocustを起動します
- 2行目、host がトラフィックを送付する宛先です
- 3行目、最終的にシュミレートする同時接続ユーザ数です
- 4行目、一秒間に増加するユーザ数です
- 5行目、トラフィックを発生させる秒数です

以下がSenarioファイルのサンプルです。Locust ではPythonで発生させるトラフィックの内容を詳細に記述することが可能です。その他詳細は `Locust Writing a locustfile <https://docs.locust.io/en/stable/writing-a-locustfile.html>`__ を参照してください。

.. code-block:: bash
  :caption: Senario ファイルサンプル (html.py)
  :linenos:

  import time
  from locust import HttpUser, task, between, constant_throughput, constant
  
  class QuickstartUser(HttpUser):
      wait_time = constant_throughput(15)
  
      @task
      def hello_world(self):
          self.client.get("/html/index.html")


- 5行目、 ``constant_throughput(15)`` という表記で、1ユーザが1秒間あたりに task を実施する数を指定しています。つまりこのサンプルでは1ユーザ15rpsのリクエストを送付します
- 7行目、 ``@task`` という形でデコレータの記述があり、この内容がシミュレートされるユーザによって実行されます。このサンプルでは記述しておりませんが、複数の task を指定した割合で実行するなどが可能です
- 9行目、 このシナリオでは ``/html/index.html`` に対して ``GET`` を送付する動作となります


1. HTTP - 静的HTML
====

以下テストを実施します。

+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|#  |Protocol |Path    |応答するコンテンツ|想定最大RPS|RPS/ユーザ |最大ユーザ数 |追加ユーザ数/秒 |
+===+=========+========+==================+===========+===========+=============+================+
|1  |HTTP     |/html/  |静的HTML          |1800       |15         |120          |10              |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+


パフォーマンステストの実施
----

作業用ホストで以下コマンドを実行し、トラフィックを発生させます

.. code-block:: cmdin

  # cd ~/f5j-nginx-performance-lab/ansible
  ansible-playbook -i inventory/hosts -l locust load-generate/load-http-html-allservers.yaml

.. code-block:: bash
  :caption: 実行結果サンプル
  :linenos:

  PLAY [all] *********************************************************************
  
  TASK [Gathering Facts] *********************************************************
  ok: [10.1.1.7]
  
  TASK [Locust http to html] *****************************************************
  changed: [10.1.1.7]
  
  PLAY RECAP *********************************************************************
  10.1.1.7                   : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


結果の確認
----

Grafanaのダッシュボードを確認してください。サンプルの結果を以下に示します。

  .. image:: ./media/grafana-http-html.png
     :width: 500

- 上から ``Locust`` 、 ``NGINX Unit`` 、 ``NGINX Plus`` 、 ``Apache`` の結果となります
- CPU、Memoryグラフの最大値は記載の通りです
- NGINX Unit、NGINX Plusでは、それぞれの記載のCPU利用率となります。メモリは通信処理時に大きな変化はありませんでした
- Apacheは、12.8% の使用率となり、メモリは通信処理時に 約62MB 増加し、380MB となりました

Locustサーバ Webページ を確認します

  .. image:: ./media/locust-web-http-html.png
     :width: 500

画面を更新すると、以下の表にLocustの実行結果が表示されます。各行が各ホストに対する実行結果を示しています。
詳細なレポートを確認する場合、対象の行をクリックしてください。以下にサンプルを示します。

  .. image:: ./media/locust-web-http-html-report.png
     :width: 500

新たなWindowに結果が表示されます。

- 画面左上部に ``Target Host`` 、 そして送付するリクエストの内容を示す ``Script`` が表示されていることが確認できます
- Requestの ``Fails`` からエラーなく通信の処理ができていることが確認できます
- Requestの ``RPS`` から秒間処理リクエスト数が 1517.2 であることが確認できます

2. HTTPS - 静的HTML
====

以下テストを実施します。

+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|#  |Protocol |Path    |応答するコンテンツ|想定最大RPS|RPS/ユーザ |最大ユーザ数 |追加ユーザ数/秒 |
+===+=========+========+==================+===========+===========+=============+================+
|2  |HTTPS    |/html/  |静的HTML          |1800       |15         |120          |10              |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+

パフォーマンステストの実施
----

作業用ホストで以下コマンドを実行し、トラフィックを発生させます。コマンドの出力結果は ``HTTP-静的HTML`` と同様のため省略します。

.. code-block:: cmdin

  # cd ~/f5j-nginx-performance-lab/ansible
  ansible-playbook -i inventory/hosts -l locust load-generate/load-https-html-allservers.yaml


結果の確認
----

Grafanaのダッシュボードを確認してください。サンプルの結果を以下に示します。

  .. image:: ./media/grafana-https-html.png
     :width: 500


- 上から ``Locust`` 、 ``NGINX Unit`` 、 ``NGINX Plus`` 、 ``Apache`` の結果となります
- CPU、Memoryグラフの最大値は記載の通りです
- NGINX Unit、NGINX Plusでは、それぞれの記載のCPU利用率となります。メモリは通信処理時に大きな変化はありませんでした
- Apacheは、 12.2% の使用率となり、メモリは通信処理時に 約59MB 増加し、395MB となりました


Locustサーバ Webページ を更新し結果が表示されることを確認します。詳細なレポートを確認する場合、対象の行をクリックしてください。

  .. image:: ./media/locust-web-https-html-report.png
     :width: 500

- 画面左上部に ``Target Host`` 、 そして送付するリクエストの内容を示す ``Script`` が表示されていることが確認できます
- Requestの ``Fails`` からエラーなく通信の処理ができていることが確認できます
- Requestの ``RPS`` から秒間処理リクエスト数が 1233.2 であることが確認できます

3. HTTP - Wordpress(PHP)
====


以下テストを実施します。

+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|#  |Protocol |Path    |応答するコンテンツ|想定最大RPS|RPS/ユーザ |最大ユーザ数 |追加ユーザ数/秒 |
+===+=========+========+==================+===========+===========+=============+================+
|3  |HTTP     |/       |Wordpress(PHP)    |10         |1          |10           |2               |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+

このテストではWebサーバとしての機能だけでなく、PHPの実行、SQLサーバの処理があるため、一つのリクエストに対し様々な機能が動作しています。
その動作がどの様に変化するか確認してください。

パフォーマンステストの実施
~~~~

作業用ホストで以下コマンドを実行し、トラフィックを発生させます。コマンドの出力結果は ``HTTP-静的HTML`` と同様のため省略します。

.. code-block:: cmdin

  # cd ~/f5j-nginx-performance-lab/ansible
  ansible-playbook -i inventory/hosts -l locust load-generate/load-http-wp-allservers.yaml

.. NOTE::
  稀にNGINX UnitがPHPの応答を返さない状態になる場合があります。その際には、 ``ubuntu01`` に接続し、以下コマンドを実行してください。テスト実施時間の 120秒 経過を待ち、再度テストを行ってください

  .. code-block::

    sudo service unit restart

結果の確認
----

Grafanaのダッシュボードを確認してください。サンプルの結果を以下に示します。

  .. image:: ./media/grafana-http-wp.png
     :width: 500

- 上から ``Locust`` 、 ``NGINX Unit`` 、 ``NGINX Plus`` 、 ``Apache`` の結果となります
- CPU、Memoryグラフの最大値は記載の通りです
- CPU利用率は NGINX Plusが 88.2% 、Apacheが 93% となっています。NGINX Unitでは、NGINX UnitがPHPを実行しています。CPU利用率は 50% となその他Webサーバと比較して利用率が低くなっております
- NGINX Unit、NGINX Plusでは、メモリは大きな増加はありません。Apacheは 約20MB 増加し、447MBとなりました

Locustサーバ Webページ を更新し結果が表示されることを確認します。詳細なレポートを確認する場合、対象の行をクリックしてください。

  .. image:: ./media/locust-web-http-wp-report.png
     :width: 500

- 画面左上部に ``Target Host`` 、 そして送付するリクエストの内容を示す ``Script`` が表示されていることが確認できます
- Requestの ``Fails`` からエラーなく通信の処理ができていることが確認できます
- Requestの ``RPS`` から秒間処理リクエスト数が 9.8 であることが確認できます。WordpressへのリクエストではWebサーバの他様々なコンポーネントが動作するため、リクエスト数に対しリソースの消費が大きくなっています

4. HTTPS - Wordpress(PHP)
====


以下テストを実施します。

+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|#  |Protocol |Path    |応答するコンテンツ|想定最大RPS|RPS/ユーザ |最大ユーザ数 |追加ユーザ数/秒 |
+===+=========+========+==================+===========+===========+=============+================+
|4  |HTTPS    |/       |Wordpress(PHP)    |10         |1          |10           |2               |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+

このテストではWebサーバとしての機能だけでなく、PHPの実行、SQLサーバの処理があるため、一つのリクエストに対し様々な機能が動作しています。
その動作がどの様に変化するか確認してください。

パフォーマンステストの実施
~~~~

作業用ホストで以下コマンドを実行し、トラフィックを発生させます。コマンドの出力結果は ``HTTP-静的HTML`` と同様のため省略します。

.. code-block:: cmdin

  # cd ~/f5j-nginx-performance-lab/ansible
  ansible-playbook -i inventory/hosts -l locust load-generate/load-http-wp-allservers.yaml

結果の確認
----

Grafanaのダッシュボードを確認してください。サンプルの結果を以下に示します。

  .. image:: ./media/grafana-https-wp.png
     :width: 500

- 上から ``Locust`` 、 ``NGINX Unit`` 、 ``NGINX Plus`` 、 ``Apache`` の結果となります
- CPU、Memoryグラフの最大値は記載の通りです
- CPU利用率は NGINX Plusが 88% 、Apacheが 96% となっています。NGINX Unitでは、NGINX UnitがPHPを実行しています。CPU利用率は 51% となその他Webサーバと比較して利用率が低くなっております
- NGINX Unit、NGINX Plusでは、メモリは大きな増加はありません。Apacheは 約29MB 増加し、453MBとなりました

Locustサーバ Webページ を更新し結果が表示されることを確認します。詳細なレポートを確認する場合、対象の行をクリックしてください。

  .. image:: ./media/locust-web-https-wp-report.png
     :width: 500

- 画面左上部に ``Target Host`` 、 そして送付するリクエストの内容を示す ``Script`` が表示されていることが確認できます
- Requestの ``Fails`` からエラーなく通信の処理ができていることが確認できます
- Requestの ``RPS`` から秒間処理リクエスト数が 8 であることが確認できます

5. 結果の考察
====

全体的なテストの結果
----

+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|#  |Protocol |Path    |応答するコンテンツ|想定最大RPS|RPS/ユーザ |最大ユーザ数 |追加ユーザ数/秒 |
+===+=========+========+==================+===========+===========+=============+================+
|1  |HTTP     |/html/  |静的HTML          |1800       |15         |120          |10              |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|2  |HTTPS    |/html/  |静的HTML          |1800       |15         |120          |10              |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|3  |HTTP     |/       |Wordpress(PHP)    |10         |1          |10           |2               |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+
|4  |HTTPS    |/       |Wordpress(PHP)    |10         |1          |10           |2               |
+---+---------+--------+------------------+-----------+-----------+-------------+----------------+

- html はすべてのWebサーバで程度安定した結果となります
- Apache は通信を処理する際にWorker Processをforkするため、同時接続ユーザ数に応じてメモリ消費量が増加します
- Wordpress の処理はWebサーバの同一ホスト内でPHP・DBが動作するため少ないリクエスト数でCPU利用率が高くなっています
- NGINX Unit は PHPの処理をNGINX UnitのPHPモジュールにて実施しますが、NGINX PlusやApacheの環境に比べてCPU利用率が低くなっています

その他テストから確認した結果
----

ラボ環境のApacheのデフォルト設定では、最大Worker Process数が ``150`` となっています。これは同時に処理可能なクライアント数を指定しています

.. code-block:: bash
  :caption: Apache 同時処理可能なクライアント設定 (mpm_worker.conf)
  :linenos:

  <IfModule mpm_worker_module>
          StartServers             2
          MinSpareThreads          25
          MaxSpareThreads          75
          ThreadLimit              64
          ThreadsPerChild          25
          MaxRequestWorkers        150
          MaxConnectionsPerChild   0
  </IfModule>

このためパフォーマンステストのユーザ数がこの値を超える場合、以下のようなエラーを返します

.. code-block:: bash
  :caption: 同時接続数超過によるエラーログ (error.log)
  :linenos:

  [Wed Nov 16 07:29:52.251435 2022] [mpm_prefork:error] [pid 817] AH00161: server reached MaxRequestWorkers setting, consider raising the MaxRequestWorkers setting
