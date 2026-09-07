---
name: aws-cli-operations
description: AWS CLI を使って AWS リソースの現状調査、設定変更、反映確認、認証プロファイル管理を安全に行う。Route 53、IAM、CloudFront、S3、SES などの AWS 設定を CLI で確認・更新する依頼、AWS_PROFILE の影響確認、期限切れの admin セッションを使う変更作業、変更前後の検証が必要な作業で使用する。
---

# AWS CLI Operations

読み取り専用の恒常認証と、変更時だけ有効にする一時的な管理者認証を分離して作業する。

## 認証モデル

- `default` は IAM アクセスキーによる恒常認証で、AI の調査専用の読み取り権限として扱う。
- `admin` はブラウザー認証後に一時利用できる、CDK以外の変更権限として扱う。
- `cdkdep` はAWS CDK専用のデプロイ権限として扱う。`cdk diff`、`cdk deploy`、`cdk destroy`、`cdk bootstrap`とCDKによるasset発行には必ずこのプロファイルを使う。
- 調査には必ず `--profile default`、変更には必ず `--profile admin` を明示する。
- AWS CDKコマンドだけは上記の変更原則の例外とし、必ず `--profile cdkdep` を明示する。`cdkdep`をCDK以外のAWS CLI変更操作へ流用しない。
- `AWS_PROFILE` が設定されていても暗黙プロファイルを使わない。
- アクセスキー、シークレット、セッショントークン、ログイン URL を保存・転載しない。

## 1. 環境と実行主体を確認する

AWS コマンドの前に、必ず現在の環境変数を確認する。

```powershell
$env:AWS_PROFILE
aws configure list-profiles
```

`AWS_PROFILE` が `admin` その他の値でも、調査コマンドには明示的に `--profile default` を付ける。値が設定されている状態で、プロファイル指定のない AWS API コマンドを実行しない。

```powershell
aws sts get-caller-identity --profile default
```

実行結果が読み取り専用ユーザーであることを確認する。アカウント ID や ARN を固定値として信用せず、毎回実データで判断する。

## 2. 読み取り専用で調査する

ユーザーへの追加確認なしで、依頼に必要な取得系コマンドを `--profile default` 付きで実行してよい。

- 対象の ID、ARN、名前、リージョンを AWS から解決する。
- JSON 出力を基本とし、必要な項目は `--query` で抽出する。
- ページネーション、グローバルサービス、公開・非公開の別を確認する。
- 同名設定、依存先、競合設定、既存ポリシーを調べる。
- 読み取り権限不足は明確なエラーとして示し、推測や別資格情報で補完しない。

プロファイル指定なしの `aws sts get-caller-identity` や取得系 API を安全確認の代用にしない。`AWS_PROFILE=admin` の場合に変更権限の主体を使うためである。

### PowerShellでヘルプを検索する

AWS CLI v2のクライアント側ページャーを無効化してから、PowerShellの`Select-String`（別名`sls`）でヘルプを絞り込む。PowerShellでは`grep`を前提にしない。

プロファイルへ恒久設定する場合、PowerShellが通常の`""`を引数から除去するため、stop-parsing記号`--%`を使用する。

```powershell
aws --% configure set cli_pager "" --profile default
aws --% configure set cli_pager "" --profile admin
```

`xx`は実在する対象プロファイル名へ置き換える。

```powershell
aws --% configure set cli_pager "" --profile xx
```

現在のPowerShellセッションだけ無効化する場合は環境変数を使用する。

```powershell
$env:AWS_PAGER = ""
```

設定後は、次のように検索する。

```powershell
aws help | sls bedrock
aws bedrock-agentcore-control help | sls "list-"
aws bedrock-agentcore-control list-agent-runtimes help | sls "profile|region|query|output"
```

ヘルプ検索では、調べたい階層の末尾へ必ず`help`を付ける。`aws <service>`だけを実行するとoperation不足のエラーになり、コマンド一覧は表示されない。

```powershell
# サービスを探す
aws help | sls bedrock

# サービス内のoperationを探す
aws bedrock-agentcore-control help | sls list

# operationの引数を探す
aws bedrock-agentcore-control list-agent-runtimes help | sls "profile|region|query"
```

次の形式を使用しない。

```powershell
# 誤り: helpがないためoperation不足になる
aws bedrock-agentcore-control | sls list
```

注意事項:

- `aws configure set cli_pager help`はヘルプ表示ではなく、ページャー値へ`help`を設定する誤操作である。ヘルプは`aws configure set help`で表示する。
- `--%`以降ではPowerShell変数展開とバッククォート改行を使用しない。空文字をネイティブコマンドへ渡す今回の用途に限定する。
- 単一コマンドだけ無効化する場合は`--no-cli-pager`を使用できる。
- `--no-cli-pager`は画面表示用ページャーの無効化、`--no-paginate`はAWS APIの自動ページ取得停止であり、意味が異なる。ヘルプ検索目的で`--no-paginate`を使わない。
- `aws configure get cli_pager --profile <name>`が何も表示せず、`~/.aws/config`が`cli_pager =`なら無効化できている。

## 3. 変更内容を確定する

変更前に対象、現在値、変更後の値、影響範囲、検証方法を整理する。

- ユーザーが明示的に依頼した範囲だけ変更する。
- 削除、置換、公開範囲の拡大、権限付与は高影響操作として対象を再確認する。
- `UPSERT` は意図しない上書きになり得るため、既存値を取得してから使う。
- 認証失敗、対象不明、競合ありの状態で変更へ進まない。フォールバック実装は禁止する。

## 4. 変更用プロファイルを確認する

### AWS CDKではcdkdepを使う

AWS CDKによる変更では、`admin`ではなく`cdkdep`を使う。実行前に、対象スタック、CloudFormation変更セット、S3・ECRへのasset発行、スタック配下の更新対象、実行後の検証方法をユーザーへ説明する。

```powershell
aws sts get-caller-identity --profile cdkdep
npx cdk diff <stack-name> --profile cdkdep
npx cdk deploy <stack-name> --profile cdkdep
```

デプロイスクリプトがconfigのプロファイルを読む場合も、実効プロファイルが`cdkdep`であることを確認する。`AWS_PROFILE`の暗黙値に依存しない。認証失敗時はエラーを示し、`admin`へ切り替えて回避しない。

### CDK以外の変更ではadminを一時認証する

CDK以外の変更依頼がある場合だけ、まず admin の状態を明示的に確認する。

### adminを使う理由を実行前に毎回説明する

`--profile admin`を付けたコマンドまたは`aws login --profile admin`を実行する前に、次をユーザーへ具体的に説明する。過去のメッセージで説明済みでも、adminを再試行する前には省略しない。

- 今から行う変更操作と対象リソース
- その操作に必要な主な書き込み権限
- 読み取り専用の`default`では実行できない理由
- 想定する影響範囲と実行後の検証方法

「変更権限が必要」だけの抽象的な説明は禁止する。変更するAWS APIと対象リソースを具体的に示す。AWS CDKの説明と認証は前項の`cdkdep`手順に従う。

```powershell
aws sts get-caller-identity --profile admin
```

期限切れなら、ブラウザー認証が開くことをユーザーに伝えてから実行する。

```powershell
aws login --profile admin
aws sts get-caller-identity --profile admin
```

期待するアカウントの管理者ロールであることを確認できるまで変更しない。

## 5. 変更用プロファイルを明示する

CDK以外の変更コマンドには例外なく `--profile admin` を付ける。AWS CDKコマンドには例外なく `--profile cdkdep`を付ける。`AWS_PROFILE` に依存しない。

```powershell
aws <service> <mutating-command> ... --profile admin
npx cdk deploy <stack-name> --profile cdkdep
```

失敗時は終了コードと AWS エラーを示す。別リージョン、別アカウント、別資格情報へ暗黙に切り替えない。

## 6. 反映を検証する

変更 API の成功だけで完了としない。同じリソースを取得し直し、要求値と一致することを確認する。CDK以外の変更結果確認は一貫性のため `--profile admin` を使ってよい。CDKデプロイ結果は`--profile cdkdep`でCloudFormationと出力を確認する。

- 非同期変更は waiter またはステータス API で完了を確認する。
- DNS は Route 53 の値に加え、必要なら権威 DNS または公開 DNS の応答も確認する。
- 不一致や伝播中を完了と報告しない。

## 7. 一時認証を終了する

この作業で `aws login --profile admin` を実行した場合は、ユーザーが継続利用を明示しない限り終了時にログアウトする。

```powershell
aws logout --profile admin
```

原則として `AWS_PROFILE` は設定せず、すべてのコマンドで `--profile` を明示する。AI 自身が一時設定した場合だけ終了時に解除する。作業開始前からユーザーが設定していた値は勝手に変更せず、最終報告で残存値を注意喚起する。

```powershell
$env:AWS_PROFILE = $null
```

## Route 53 の追加規則

- ホストゾーンを名前から解決し、公開・プライベートの別を確認する。
- 同一名の既存レコードと、CNAME と他タイプの競合を確認する。
- TXT の引用符と完全修飾ドメイン名の末尾ドットを AWS CLI 用 JSON で正しく扱う。
- 変更後は `list-resource-record-sets --profile admin` で対象名を再取得する。
- 伝播待ちは `route53 wait resource-record-sets-changed --profile admin` に変更 ID を渡す。

## 完了報告

実行主体、変更前後、検証結果、未完了事項を簡潔に報告する。資格情報、認証 URL、不要なアカウント情報は含めない。権限不足や期限切れを成功として扱わない。
