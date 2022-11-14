F5 Labs - Index
####

ようこそ！
====

| ようこそ！NGINX Performance Labへ！
| 実際にNGINXを触って試していただくための手順をまとめています。
| ラボ環境にアクセス可能な方は手順に従って操作をしてください。

このラボでは、各種ツールを用いてNGINX等Webサーバの応答状況を確認致します。
各種ツールやコンポーネントに関する情報を以下に示します。
詳細について確認されたい方は各種サイトをご覧ください

- `NGINX Plus <https://docs.nginx.com/nginx/>`__
  - NGINX が提供する高性能・エンタープライズグレードのWebアプリケーションサーバです
  - 静的HTMLコンテンツ、 `PHP FPM(FastCGI Process Manager) <https://www.php.net/manual/ja/install.fpm.php>`__ を用いPHPを応答を返します

- `NGINX Unit <https://unit.nginx.org/>`__
  - NGINX が提供するオープンソースの動的アプリケーションサーバです
  - 静的HTMLコンテンツ、PHPを用いた応答を返します

- `Apache Web Server <https://httpd.apache.org/>`__
  - クロスプラットフォームのオープンソースWebサーバです

- `Locust <https://locust.io/>`__
  - オープンソースの負荷計測ツールです
  - CLIやWebUIを備えており、Pythonにより負荷計測シナリオを記述することが可能です

- `Grafana <https://grafana.com/oss/grafana/>`__
  - 様々なメトリクスを視覚的に分析ができる、マルチプラットフォームで動作するオープンソースのWebアプリケーションです
  - 多くのデータに対応し、チャートやグラフ等自由にダッシュボードを作成することにより、素早く状態を把握することが可能となります

- `Prometheus <https://prometheus.io/>`__
  - オープンソースのシステム監視およびアラートツールキットです
  - メトリック名とキー/値のペアによって識別される時系列データを用いステータスを管理します
  - 各種メトリクスの値、グラフ化が可能です
  - Grafanaと簡単に連携ができ、容易にダッシュボードでステータスを把握できます
  
- `Node Exporter <https://prometheus.io/docs/guides/node-exporter/>`__
  - Hostの各種ステータスをメトリクスで取得、Prometheusで閲覧可能とする機能です
  - このラボではDocker Imageで実行することにより各種ホストのステータスを取得し、Prometheusにて管理します

.. toctree::
   :maxdepth: 3
   :caption: Lab Contents:
   :glob:

   class*/module*/module*

