目的：
CLIを使用してCDF-PCでデータフローを作成／インポートし、それを環境にデプロイする


# 1) CDP Cli インストール

MacOSを使用しているので、この方法でcdpcliをインストールします。
https://docs.cloudera.com/cdp-public-cloud/cloud/cli/topics/mc-install-cdp-client-on-macos.html#mc-install-cdp-client-on-macos


```
pip3 install cdpcli
```

インストール結果を確認：

```
[zzeng@zeng-mbp ~]$ ll ~/Library/Python/3.9/bin/ | grep cdp
-rwxr-xr-x@ 1 zzeng  staff   250 Mar  4 12:22 cdp
-rwxr-xr-x@ 1 zzeng  staff   250 Mar  4 12:22 cdp_completer
[zzeng@zeng-mbp ~]$ export PATH="$HOME/Library/Python/3.9/bin:$PATH"
[zzeng@zeng-mbp ~]$ cdp --version
0.9.107
[zzeng@zeng-mbp ~]$
```


# 2) CDP cli設定
公式ドキュメント参照URL:
https://docs.cloudera.com/cdp-public-cloud/cloud/cli/topics/mc-configuring-cdp-client-with-the-api-access-key.html

[Management Console] ->[User] -> [Profile]

![jpg01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/3caaf936-f8e2-069a-2089-78e37a74c25f.png)

![jpg02.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/fe2c7a0d-a527-d2b0-ecfc-aee5ca523417.png)

上記メニューでAPI keyを確認（もしくは作成。）

Cloudera Data PlatformのAPI Keyをメモった後、CLI側で設定する。
ドキュメント：
https://docs.cloudera.com/cdp-public-cloud/cloud/cli/topics/mc-configuring-cdp-client-with-the-api-access-key.html

```
cdp configure
```


設定後、下記コマンドで状態確認。

```
[zzeng@zeng-mbp ~]$ cdp iam get-user
{
    "user": {
        "userId": *****
        "status": "ACTIVE",
        "workloadPasswordDetails": {
            "isPasswordSet": true
        }
    }
}
```


# 3) CDP CLIでDataFlowを作成（Import）


コマンドフォーマット：
```
cdp df import-flow-definition \
  --name "zzeng2-fetch_from_S3_folder" \
  --file "/<<PATH_TO_UPDATE>>/fetch_from_S3_folder.json" \
  --comments "Initial Version"
```

例：

```
$ cdp df import-flow-definition   --name "zzeng-fetch_from_S3_folder"   --description "Description for this flow"   --file "/Users/zzeng/Library/CloudStorage/OneDrive-Personal/38_CLDR_Docs/50_demo/FetchFromS3Folder/fetch_from_S3_folder.json"   --comments "Initial Version"
{
    "crn": "crn:cdp:df:us-west-1:******:flow:zzeng-fetch_from_S3_folder",
    "name": "zzeng-fetch_from_S3_folder",
    "versionCount": 1,
    "createdTimestamp": 1709632435790,
    "description": "Description for this flow",
    "modifiedTimestamp": 1709632435790,
    "versions": [
        {
            "crn": "crn:cdp:df:us-west-1:******:flow:zzeng-fetch_from_S3_folder/v.1",
            "bucketIdentifier": "https://s3.us-west-2.amazonaws.com/*****.cloudera.com/******",
            "author": "Zhen Zeng",
            "version": 1,
            "timestamp": 1709632435792,
            "deploymentCount": 0,
            "comments": "Initial Version",
            "draftCount": 0,
            "tags": []
        }
    ]
}
```


# 4) 該当DataFlowをDeployする

CDF-PCの画面でWizard式入力して作るか、CLIで作るか、どちらでもできます。
CLIを作る一つ楽な方法は、CDF-PCの画面でWizard式入力して、最後のDeploy画面まで進んだら、「CLIを表示メニュー」があります。
それをクリックしたら、DataFlowをDeployするCLIが自動的に生成されている。

![img4-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/8ec842ce-dd7d-eef2-356e-f900092b02bf.png)


```
cdp df create-deployment \
  --service-crn crn:cdp:df:us-west-1:558bc1d2-8867-4357-8524-311d51259233:service:08280633-a615-41b4-953d-87b74d5f7fd9 \
  --flow-version-crn "crn:cdp:df:us-west-1:558bc1d2-8867-4357-8524-311d51259233:flow:zzeng-fetch_from_S3_folder/v.1" \
  --deployment-name "zzeng-deploy-01" \
  --project-crn "crn:cdp:df:us-west-1:558bc1d2-8867-4357-8524-311d51259233:project:fd081b85-736a-4648-956e-e80b8848c8db" \
  --cfm-nifi-version 1.24.0.2.3.13.0-9 \
  --auto-start-flow \
  --cluster-size-name EXTRA_SMALL \
  --static-node-count 1 \
  --no-auto-scaling-enabled
```

例：

```
$ cdp df create-deployment \
>   --service-crn crn:cdp:df:us-west-1:******:service:***** \
>   --flow-version-crn "crn:cdp:df:us-west-1:*****:flow:zzeng-fetch_from_S3_folder/v.1" \
>   --deployment-name "zzeng-deploy-01" \
>   --project-crn "crn:cdp:df:us-west-1:****:project:*****" \
>   --cfm-nifi-version 1.24.0.2.3.13.0-9 \
>   --auto-start-flow \
>   --cluster-size-name EXTRA_SMALL \
>   --static-node-count 1 \
>   --no-auto-scaling-enabled
{
    "deploymentCrn": "crn:cdp:df:us-west-1:******:deployment:*****/*****"
}

```

![img5-1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/133516/1ba92a06-7e5c-69d8-c09d-e343fa3f1268.png)


参考記事：

* https://cloudera.github.io/cdp-dev-docs/cli-docs/dfworkload/update-deployment.html?highlight=update%20deployment
* https://docs.cloudera.com/cdp-public-cloud/cloud/cli/topics/mc-cli-client-setup.html
* https://docs.cloudera.com/dataflow/cloud/deploy-flows/topics/cdf-deploy-flow-cli.html 

