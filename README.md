# ServiceCatalogFactoryProductTemplate

## GitHub Actions: Upload templates to S3

このリポジトリには、`templates/` 配下のテンプレートファイル（`.yaml` or `.yml`）を `zip` 化し、変更があったファイルのみ S3 にアップロードする GitHub Actions ワークフローを含みます。

### ワークフローの場所

` .github/workflows/upload-product.yml `

### セットアップ (Secrets)

いずれかの認証方法を設定してください。

1) OIDCでロール引受け
- `AWS_ROLE_TO_ASSUME`: 引受けるIAMロールARN
- `AWS_REGION`: AWSリージョン (例: ap-northeast-1)

2) アクセスキー方式
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`

S3アップロード先の指定（どちらか/両方）
- `S3_BUCKET`: デフォルトのアップロード先バケット名
- `S3_KEY`: デフォルトのオブジェクトキー（省略時は `product.zip`）

### 手動実行 (workflow_dispatch)

Actions タブから `Upload Product Template to S3` を選び、以下の入力を必要に応じて指定します。
- `s3_bucket`: アップロード先バケット（未指定時は `S3_BUCKET` Secret を使用）
- `s3_key`: オブジェクトキー（未指定時は `templates/` 相対のディレクトリパスに `product.zip` を付与。例: `templates/path/to/template/...` -> `path/to/template/product.zip`）
- `aws_region`: リージョン（未指定時は `AWS_REGION` Secret を使用）

### 自動実行 (push)

`main` ブランチに `templates/**` の変更が push された場合に自動で実行されます。差分検出ジョブが `git diff` で変更されたテンプレートのみを検出し、マトリクスで並列アップロードします。

### 動作概要

1. リポジトリをチェックアウト
2. 差分検出で `templates/**` の変更ファイル一覧を生成
3. 変更ファイルごとに並列ジョブを生成（マトリクス）
4. 各ジョブで対象テンプレートを `zip` 化
5. AWS 認証を設定（OIDC もしくは アクセスキー）
6. `aws s3 cp` で S3 にアップロード（キーは相対パスを `.zip` に置換）

### 事前要件

- `templates/path/to/template/*.yaml(yml)` が存在すること
- ワークフローを動かす環境に Secrets が設定済みであること
  - `S3_KEY` は不要になりました