パフォーマンステストの実施
####

各Webサーバに対してパフォーマンステストを実施します

実施するパフォーマンステストの内容は以下の通りです

+---+---------+--------+------------------+----------------------+-----------------+
|#  |Protocol |Path    |応答するコンテンツ|最終同時接続ユーザ数  |秒間追加ユーザ数 |
+===+=========+========+==================+======================+=================+
|1  |HTTP     |/html/  |静的HTML          |900                   |100              |
+---+---------+--------+------------------+----------------------+-----------------+
|2  |HTTPS    |/html/  |静的HTML          |900                   |100              |
+---+---------+--------+------------------+----------------------+-----------------+
|3  |HTTP     |/       |Wordpress(PHP)    |10                    |2                |
+---+---------+--------+------------------+----------------------+-----------------+


0. パフォーマンステスト準備
====

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

  import time
  from locust import HttpUser, task, between
  
  class QuickstartUser(HttpUser):
  #    wait_time = between(1, 5)
  
      @task
      def hello_world(self):
          self.client.get("/html/index.html")


- 7行目、 ``@task`` という形でデコレータの記述があり、この内容がシミュレートされるユーザによって実行されます。このサンプルでは記述しておりませんが、複数の task を指定した割合で実行するなどが可能です
- 9行目、 このシナリオでは ``/html/index.html`` に対して ``GET`` を送付する動作となります


1. HTTP - 静的HTML
====

以下テストを実施します。

+---+---------+--------+------------------+----------------------+-----------------+
|#  |Protocol |Path    |応答するコンテンツ|最終同時接続ユーザ数  |秒間追加ユーザ数 |
+===+=========+========+==================+======================+=================+
|1  |HTTP     |/html/  |静的HTML          |1100                  |100              |
+---+---------+--------+------------------+----------------------+-----------------+


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
- Apacheは、 17.5% の使用率となり、メモリは通信処理時に 約100MB 増加し、435MB となりました

Locustサーバ Webページ を確認します

  .. image:: ./media/locust-web-http-html.png
     :width: 500

画面を更新すると、以下の表にLocustの実行結果が表示されます。各行が各ホストに対する実行結果を示しています。
詳細なレポートを確認する場合、対象の行をクリックしてください。以下にサンプルを示します。

  .. image:: ./media/locust-web-http-html-report.png
     :width: 500

新たなWindowに結果が表示されます。Request / Response の情報が確認できます。また、Requestの ``#Fails`` からエラーなく通信の処理ができていることが確認できます

2. HTTPS - 静的HTML
====

以下テストを実施します。

+---+---------+--------+------------------+----------------------+-----------------+
|#  |Protocol |Path    |応答するコンテンツ|最終同時接続ユーザ数  |秒間追加ユーザ数 |
+===+=========+========+==================+======================+=================+
|2  |HTTPS    |/html/  |静的HTML          |900                   |100              |
+---+---------+--------+------------------+----------------------+-----------------+

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
- Apacheは、 17.5% の使用率となり、メモリは通信処理時に 約100MB 増加し、435MB となりました


Locustサーバ Webページ を更新し結果が表示されることを確認します。詳細なレポートを確認する場合、対象の行をクリックしてください。

  .. image:: ./media/locust-web-https-html-report.png
     :width: 500


3. HTTP - Wordpress(PHP)
====


以下テストを実施します。

+---+---------+--------+------------------+----------------------+-----------------+
|#  |Protocol |Path    |応答するコンテンツ|最終同時接続ユーザ数  |秒間追加ユーザ数 |
+===+=========+========+==================+======================+=================+
|3  |HTTP     |/       |Wordpress(PHP)    |10                    |2                |
+---+---------+--------+------------------+----------------------+-----------------+

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

  .. image:: ./media/grafana-http-wp.png
     :width: 500

- 上から ``Locust`` 、 ``NGINX Unit`` 、 ``NGINX Plus`` 、 ``Apache`` の結果となります
- CPU、Memoryグラフの最大値は記載の通りです
- NGINX Plus、Apache は CPU利用率が 100% となっています。NGINX Unitでは、NGINX UnitがPHPを実行しています。CPU利用率は 51% となその他Webサーバと比較して利用率が低くなっております
- NGINX Unit、NGINX Plusでは、メモリは大きな増加はありません。Apacheは 約364MB 増加し、765MBとなりました

Locustサーバ Webページ を更新し結果が表示されることを確認します。詳細なレポートを確認する場合、対象の行をクリックしてください。

  .. image:: ./media/locust-web-http-wp-report.png
     :width: 500
