

原文： Using Iceberg Table Format in CDP Public Cloud to Ingest, Process and Analyze Stock Intraday Data
https://community.cloudera.com/t5/Community-Articles/Using-Iceberg-Table-Format-in-CDP-Public-Cloud-to-Ingest/ta-p/337329


# 概要

データ分析のユースケースにおいて、株式市場はよく使われる例です。Cloudera Data Platformを使用して、迅速な洞察から可視化までをどのように簡単に作成できるでしょうか？

CDPでIcebergテーブルフォーマットを使用して、統合され安全な新機能を利用するのはどうでしょうか？【[Cloudera Data Platform における Apache Iceberg の導入](https://blog.cloudera.jp/jp-introducing-apache-iceberg-in-cloudera-data-platform/)】参照。

本日の記事では、Cloudera Public Cloudを使用してREST APIからデータを取得し、クリックやパラメータ入力のみで、データのクエリ、ダッシュボード、および高度な分析を迅速に開始する方法を紹介します。

この例では、下記の手順を行います：

1) Cloudera Dataflowを使用して、定期的に（D-1データとして）オブジェクトストレージにイントラデイ株価データをロードします。
2) Sparkを使用してデータを処理し、Cloudera Data Engineeringで利用可能にし、Airflowでスケジュールします。
   このプロセスでは、テーブルが新しいかどうかをチェックし、その後新しいICEBERG株価テーブルにMERGE INTOします。コードはこちらです。
3) Cloudera Data Warehouse/Cloudera Data Visualizationでデータを分析およびクエリします。
4) テーブルでTIME TRAVEL/SNAPSHOTS（Icebergの機能）を実行します。

また、Icebergを使用することで、 **Schema Evolution** を実行し、Parquetなどのオープンフォーマットを使用してオープンソースの最適化やエンジン間の相互運用性を利用することができます。

今回利用する株価情報はD-1ですが、新しい株価ティッカーがパラメータに存在するかどうかを特定するために、10分ごとに実行するスケジュールを設定します。

さらに、**すべての操作をコーディングせず**に、保存されたテンプレートのみを使用します！

以下が今回のアーキテクチャです：


![アーキテクチャ](https://community.cloudera.com/t5/image/serverpage/image-id/33708i6FFF51F5D273FD92/image-size/medium?v=v2&px=400)

# 前提条件

株価情報をダウンロードするために、この記事を書いている時点で無料のAPIである[Alpha Vantage](https://www.alphavantage.co/)を使用します。

まず、使用するAPIキーを取得するために登録が必要です。そのAPIキーを保存してください。

また、データを保存するバケットのパス名も必要です。必要な情報（AWS APIキー、Bucket）が揃ったので、始めましょう！


以下は、入力が必要なパラメータのリストです：

*    Alpha Vantage API Key;
*    Root Bucket used by CDP;
*   Workload username;
*  Workload password;

今回利用するCloudera Data Platformのコンポーネントは以下の通りです：

*   Cloudera Data Warehouse;
*   Cloudera Dataflow;
*   Cloudera Data Engineering(Spark 3);


## REST APIで株価データをオブジェクトストレージに取り込むデータフローを作成する



1) まず、ここにある[テンプレート](https://github.com/carrossoni/clouderacommunity/tree/main/cdp_stock_iceberg/1_cdf_flow_stock_template)ファイルをダウンロードして、Cloudera DataFlowにアクセスします。そこでテンプレートをアップロードし、オブジェクトストレージにデータのロードするデータフローをインポートします。
2) Cloudera Dataflow UIで、「Catalog」→「Import Flow Definition」をクリックします。

![CDF](https://community.cloudera.com/t5/image/serverpage/image-id/33709iC89EFED49E4B3902/image-size/medium?v=v2&px=600)
3) Put a name for your flow, description and select the file CDF template that you've downloaded
   ![CDF2](https://community.cloudera.com/t5/image/serverpage/image-id/33710i4E236B9BD6812824/image-size/medium?v=v2&px=600)
4) Click Import;
5) After deploying, select your flow, and in the menu, click the blue button (Deploy):
6) Select the CDP Flow Environment that you are using and then continue:
7) Put your deployment name and click Next
8) Do not change any NiFi configuration and click Next;
9) Now it will ask for the parameters that we need to provide, input your

* CDP Password: Workload password in CDP
* CDP User: Workload user in CDP
* S3 Path: Subfolder that will be created under the main <bucket>/user/<youruser>/<s3path>  (only the name without s3 etc)
* S3 Bucket: The main CDP Bucket used by CDP (only the name without s3 etc)
* API Alpha Key: The API key that will be used (demo can only get IBM stock data)
* Stock_list: the list of the stocks that you want to load, put in each line the ticker of the stock
Note: It is possible to change the stock list after deploying to ingest new ticker data. We will do this to demonstrate the Iceberg time travel feature.

Click Next and aelect the size of the NiFi node and max scaling:
