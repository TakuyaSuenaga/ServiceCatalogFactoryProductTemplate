# ServiceCatalogFactoryProductTemplate

## GitHub Actions: Upload templates to S3

このリポジトリには、`templates/` 配下の `product.template.yaml` ファイルを `zip` 化し、変更があったファイルのみ S3 にアップロードする GitHub Actions ワークフローを含みます。

### ワークフローの場所

` .github/workflows/upload-product.yml `

### セットアップ (Secrets)

OIDCでロール引受けによる認証を使用します。

**必須Secrets:**
- `AWS_ROLE_TO_ASSUME`: 引受けるIAMロールARN
- `AWS_REGION`: AWSリージョン (例: ap-northeast-1)
- `S3_BUCKET`: デフォルトのアップロード先バケット名

**IAMロールの信頼ポリシー例:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::ACCOUNT-ID:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:OWNER/REPO:*"
        }
      }
    }
  ]
}
```

### 手動実行 (workflow_dispatch)

Actions タブから `Upload Product Template to S3` を選び、以下の入力を必要に応じて指定します。
- `s3_bucket`: アップロード先バケット（未指定時は `S3_BUCKET` Secret を使用）
- `s3_key`: オブジェクトキー（未指定時は `templates/` 相対のディレクトリパスに `product.zip` を付与。例: `templates/path/to/template/...` -> `path/to/template/product.zip`）
- `aws_region`: リージョン（未指定時は `AWS_REGION` Secret を使用）

### 自動実行 (push)

`main` ブランチに `templates/**/product.template.yaml` の変更が push された場合に自動で実行されます。差分検出ジョブが `git diff` で変更された `product.template.yaml` のみを検出し、マトリクスで並列アップロードします。

### 動作概要

1. リポジトリをチェックアウト
2. 差分検出で `templates/**/product.template.yaml` の変更ファイル一覧を生成
3. 変更ファイルごとに並列ジョブを生成（マトリクス）
4. 各ジョブで対象テンプレートを `zip` 化
5. AWS 認証を設定（OIDC ロール引受け）
6. `aws s3 cp` で S3 にアップロード（キーは相対パスを `.zip` に置換）

### 事前要件

- `templates/path/to/template/product.template.yaml` が存在すること
- ワークフローを動かす環境に Secrets が設定済みであること
- GitHub OIDC プロバイダーが AWS アカウントに設定済みであること