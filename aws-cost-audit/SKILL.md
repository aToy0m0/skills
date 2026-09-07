---
name: aws-cost-audit
description: AWS CLIのReadOnlyプロファイルでAWS費用を安全に調査し、直近1か月のスパイク、過去半年の推移、サービス・使用タイプ・オペレーション別の増加原因、AgentCore・Bedrockモデル・Lambda等の実利用量を分析する。AWS料金、請求増加、Cost Explorer、コスト異常、AgentCore Runtime、Nova、Claude、サーバーレス構成の費用確認やグラフ化を依頼されたときに使用する。
---

# AWS料金確認

AWSの費用と原因を、ReadOnly認証だけで調査する。

## 安全規則

- すべてのAWS CLIコマンドに必ず `--profile default` を直接指定する。省略しない。
- `AWS_PROFILE`、`AWS_DEFAULT_PROFILE`、シェルの既定値を認証根拠にしない。
- 調査開始時に環境変数の状態と `aws sts get-caller-identity --profile default` を確認する。
- 呼び出し元が期待するReadOnlyユーザーでない場合は調査を停止し、ユーザーへ報告する。
- `admin` プロファイル、変更系API、ログイン、設定変更を使用しない。
- 認証情報、アクセスキー、セッショントークンを表示しない。
- 権限不足を推測や別資格情報で補完しない。AWSエラーを明示する。
- AWS Cost Explorer APIには利用料があるため、必要なディメンションをまとめ、呼び出し回数を最小化する。最終報告で呼び出し回数と追加計上の可能性を伝える。

## 調査手順

### 1. 認証を固定する

PowerShellでは機密値を除外し、次を個別確認する。

```powershell
$names = 'AWS_PROFILE','AWS_DEFAULT_PROFILE','AWS_REGION','AWS_DEFAULT_REGION'
foreach ($name in $names) {
  [PSCustomObject]@{ Name = $name; Value = [Environment]::GetEnvironmentVariable($name) }
}
aws configure list --profile default
aws sts get-caller-identity --profile default --output json --no-cli-pager
```

以後のすべての `aws` コマンドにも `--profile default` を付ける。

### 2. 最少回数で費用を取得する

- Cost Explorerの終了日は排他的として扱い、報告に実データの最終日を明記する。
- `UnblendedCost` を基本指標にする。必要な場合だけ `UsageQuantity` を同じ呼び出しへ追加する。
- 直近31日と直前31日を同じ日数で比較する。
- 過去半年は月次合計とサービス別上位項目を示す。
- 直近期間は日次で `SERVICE` と `USAGE_TYPE` を同時にグループ化し、スパイク日と原因を一度に取得する。
- オペレーション名が必要な場合だけ `SERVICE` と `OPERATION` の追加照会を行う。
- Cost Anomaly Detectionが設定済みなら `ce get-anomalies` の結果と独自判定を照合する。
- 税、月初固定費、返金、クレジットを実利用スパイクと分離する。

基本形:

```powershell
aws ce get-cost-and-usage `
  --time-period Start=<start>,End=<exclusive-end> `
  --granularity DAILY `
  --metrics UnblendedCost UsageQuantity `
  --group-by Type=DIMENSION,Key=SERVICE Type=DIMENSION,Key=USAGE_TYPE `
  --profile default --output json --no-cli-pager
```

### 3. 原因をリソースまで追跡する

費用だけで断定せず、該当する読み取りAPIで裏付ける。

- Bedrock/Nova/Claude: 入出力トークン、モデル別費用、呼び出し日。
- AgentCore: Runtimeのメモリ時間・vCPU時間、Gateway呼び出し、Knowledge Base保存量。
- Lambda: メモリ設定、実行回数、Duration合計・平均・最大、エラー、スロットル。レスポンスストリーミング中も実行時間課金になる点を確認する。
- CloudWatch: Logsと `CW:OTEL:Bytes` を区別し、OpenTelemetryのinstrumentation/operationを特定する。
- KMS: `CurrentKeys` の開始日を見つけ、`kms describe-key`、タグ、CloudTrailの `CreateKey` で作成日時と主体を確認する。
- EC2 Other: EBS、IPv4、NAT等の使用タイプを確認し、停止中インスタンスに残るボリュームを調べる。
- DynamoDB: 課金モード、項目数、サイズ、PITR、読み書き利用量。
- CloudFront: リクエスト数、転送量、PriceClass。
- S3/ECR: 保存量とリクエスト。CDK bootstrap資産も対象に含める。
- Cognito: User Pool数ではなくMAUと機能ティアで判断する。

CloudFormation/IaCが調査範囲にある場合は、ローカル定義と `cloudformation list-stack-resources` の実リソースを突き合わせる。

### 4. 請求反映待ちを分離する

Cost Explorerにサービス行がなくても、デプロイ直後は「費用ゼロ」と断定しない。

- `請求反映済み`: Cost ExplorerのUnblendedCost。
- `実利用確認済み・反映待ち`: CloudWatchメトリクスや各サービスAPIの数量。
- `概算`: 公開単価×実利用量。無料枠適用前後を混同せず、前提を明記する。

サーバーレス構成では、Lambda、AgentCore、モデル、CloudFront、DynamoDB、CloudWatch、S3/ECRを別々に評価する。

### 5. 報告する

最初に結論を示し、次を簡潔にまとめる。

1. 直近期間の合計、比較期間との差額・増減率。
2. 過去半年の月次推移。
3. スパイク日、金額、原因、AWS異常検知との一致。
4. 継続課金と一時的な検証費用の区別。
5. AgentCore、モデル、サーバーレス層の利用量と費用。
6. Cost Explorerの推定フラグ、反映遅延、API照会による追加費用。

グラフを依頼された場合、または既存グラフを更新する場合は、プロジェクト直下の `output/aws-cost-analysis.html` に配置する。月次費用、日次スパイク、主要サービス内訳、反映待ちの実利用量を分けて表示する。既存ファイルがある場合は上書き更新し、JavaScript構文と主要値を検証する。
