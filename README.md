# locust
本リポジトリは負荷試験ツール「Locust」を用いてインスタントに負荷試験を実施することを目的としている。

## 構成要素
* builds
  * CloudBuildによる実行を宣言するYAML
* docker/locust
  * 負荷試験ツール「Locust」の実行コードおよびDockerfile
* k8s/manifests
  * 後述するTerraformで構築されたGKEにLocustをコンテナとして実行させるマニフェストファイル
* terraform
  * GCPプロジェクト「*****」にVPCやGKEなどの必要なリソースを構築

## ローカル環境からデプロイする際の手順

1. ユーザ認証

```bash
$ gcloud auth application-default login
```

2. LocustのDocker Imageを最新にする
 * Locustのタスクに修正があった場合のみ実行

```bash
$ gcloud builds submit --config builds/cloudbuild-docker.yaml --project *****
```

3. GKEの構築およびLocustコンテナの作成
 
```bash
$ gcloud builds submit --config builds/cloudbuild-create.yaml --project *****
```

```bash
$ gcloud builds submit --config builds/cloudbuild-destroy.yaml --project *****
```

## 負荷試験をかけるターゲットを変更する場合

`k8s/manifests/configmap.yaml` に定義している `TARGET_HOST` の値を変更する

## 負荷試験をかけるターゲットのエンドポイントが限定公開されている場合

* `builds/cloudbuild-create.yaml` `builds/cloudbuild-destroy.yaml` の変数にある `_TARGET_PROJECT` にターゲットとなるGCPプロジェクトを明示する
  * この状態で環境構築を行うことでターゲットとなるGCPプロジェクトのファイアウォールルールを操作し、負荷をかけられるようになる