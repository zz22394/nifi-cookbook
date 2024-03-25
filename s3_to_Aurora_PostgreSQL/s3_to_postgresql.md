# 目的：

Apache NiFi で、S3ファイルの更新を検知して、更新されたファイルをAurora PostgreSQLにInsert


# 完成状態のデータフロー

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/07459b76-7dec-3cd8-a0a8-baa91107346a.png)

# 手順

1.) JSON をダウンロード
JSONファイル
* [Import_S3_To_Aurora_PostgreSQL.json](https://raw.githubusercontent.com/zz22394/nifi-cookbook/main/s3_to_Aurora_PostgreSQL/Import_S3_To_Aurora_PostgreSQL.json)

2.) 新しいプロセッサーグループを作成。作成時、保存したJSONをアップロード

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/a918592f-051c-041a-26d8-aeecf7f46570.png)

作成：
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/85f86ebc-3585-6f9e-8d55-6c81cf798732.png)

これでImport完了：
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/4d4f738a-8a9e-2c38-263a-bc8572e231ee.png)

3.) JDBCドライバーを入れる

```bash
wget https://jdbc.postgresql.org/download/postgresql-42.7.3.jar
mkdir /tmp/nifi
mv postgresql-42.7.3.jar /tmp/nifi/
```

4.) パラメータ設定

4.1) ListS3 のパラメータ設定

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/46af0d0e-2dba-2087-8821-0d6a36b985ee.png)


S3のAccess Key設定：

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/cc06033a-1a90-ef5e-bde5-cb3b19044bda.png)


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/8b308257-aa88-7828-bcb6-3fa51e99bd3f.png)




![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/41678288-9923-3a7a-4c26-d639950f38d6.png)

入力した値は保護されているため、表示されない。
「Sensitive Value set」のみ表示。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/3ec3630f-857f-adda-131d-2f0dab9e2e3d.png)


4.2) AWSキーを保存しているAWSCredentialsProviderControllerServiceを起動
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/8d367fcf-1b90-db7d-c249-936f2aa90270.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/86f9d33b-9bbf-9521-728a-2233fdb80b86.png)

4.3) CSVReaderを起動

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/b5a60474-ded2-4a14-1129-b4f07a904a04.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/cdf449c4-0041-6162-ee25-833c54817c50.png)

4.4) JDBC Connection pool（DBCPConnectionPool-postgreSQL）を起動
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/f5de090a-735e-9f0c-c467-56ac5a2555a3.png)


5.1) S3ファイルを保存しているBucket、Prefixを編集

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/19cb4228-2d5c-5ee8-0071-211d208d94b7.png)


![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/9811c43d-05eb-a42d-9f97-5baf59ce3303.png)

5.2) INSERTしたいPostgreSQLのテーブル名を編集
PutDatabaseRecordプロセッサーの設定：
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/3c4b6f85-6d87-43f0-c9d7-3211d57ca497.png)


6.) 起動

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/b9fd6e9c-6cd6-b2fd-32c6-77e68c2a00ec.png)


7.) 履歴を確認：
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/ff38668d-88ad-60b1-039f-7ab9db4a985b.png)

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/3d6e9d58-46f9-30be-8212-1d5b0d66d5bf.png)
