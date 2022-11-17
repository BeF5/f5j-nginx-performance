補足情報
####

Tips1. 各種アプリケーションのデプロイ
====

本ラボ環境ではAnsibleを用いて各種環境をデプロイしています。
Ansible Playbookはこのラボで利用したGitHubレポジトリ上に管理されています。

同様の環境をデプロイする場合には以下手順を参考にセットアップを行ってください

1. Ansible Install
----

以下手順を参考に、Ansibleをインストールしてください

- https://f5j-nginx-ansible.readthedocs.io/en/latest/class1/module2/module2.html

2. GitHubに記載の内容を参考に各種アプリケーションを実行してください
----

- https://github.com/BeF5/f5j-nginx-performance-lab

Tips2. Locustに指定するパラメータ
====

Locustで期待したテストを実施するために、locustfileや、同時接続ユーザ、秒間追加ユーザ数、Locustサーバについてまとめます。

Locust の構成
----

  .. image:: ./media/locust-structure.png
     :width: 500

- 起動時

  - Locust は locustファイルにより定義されたテストシナリオを参照しプロセスを起動します
  - locustファイルは、Pythonで自由な記述が可能です。このラボではシンプルなHTTP/HTTPSとなりますが、その他POSTによるデータの送付や、ログイン処理などの実装が可能です
  - locustファイルは、1ユーザが行う内容を定義します。以下のサンプルを見ると5行目に ``15RPS`` という形でリクエストを送付する　``頻度`` を指定しています
  
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

- 起動後のパラメータ指定

  - Locustでは、最大ユーザ数、秒間増加ユーザ数、宛先ホスト、実行時間(オプション) を指定することでテストを実行できます
  - CLIで実行する際には、コマンド実行時点で起動と同時にこれらのパラメータを指定しテストを実行します

- テスト実行

  - Locustは指定された 最大ユーザ数に到達するまで、指定されたユーザを追加して行きます
  - 複数のWorker Processが動作する場合には、順にユーザをシミュレートしていきます
  - 最大ユーザ数に到達したあとは、残った指定時間の間テストを継続します
  - 実行時間が指定された場合には、その時間経過後テストを終了します

本ラボでのLocustの状態
----

本ラボで以下のような値を指定しテストを実行しています。その場合のLocustの動作について考えます

+----------------+-----------+----------------+-----------+
|Number of users |Spawn rate |Host            |Run time   |
+================+===========+================+===========+
|6000            |300        |http://10.1.1.5 |120s       |
+----------------+-----------+----------------+-----------+

このパラメータの場合以下のような動作となります。

  .. image:: ./media/locust-structure-thislab.png
     :width: 500

これらのパラメータを指定したときに以下のようなテストが期待されます

- 最大ユーザ数に約20秒で到達します
- 20秒後に6000ユーザとなり、その後locustファイルで指定した通信が1RPSで実行されます
- 最大ユーザに到達後は、約6000RPSの通信が発生すると想定されます

Locustの動作を考慮したシナリオ設計に関するコメント
----

- 1Userで実行したい通信の内容を定義します

  - 宛先 PATH、Protocol、Method、実行間隔(定期的、ランダムで指定した秒数待つ など)
  - 複数のシナリオを用意し、割合を指定した実行

- 最大到達するユーザ数を確認します

- 秒間増加するユーザ数を確認します

  - 複雑な処理を行うアプリケーションの場合、この増加数を抑えて状態を確認する必要があるかもしれません

- 指定のシナリオが最大ユーザ数に到達した際に想定されるRPSを確認します

- Locustファイル作成後、最大ユーザ数を低くした上で、どのような通信状況となるか確認します

  - Webサーバはエラーを出力していないか、想定外の動作が発生しないか
  - LocustサーバのCPU利用率はどの様になっているか、Worker Processは必要十分か、エラーが発生していないか


これらを考慮し、適切なテストシナリオを設計することでより効果的な負荷計測が行なえます