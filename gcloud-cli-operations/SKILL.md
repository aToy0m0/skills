---
name: gcloud-cli-operations
description: gcloud CLIを使ってGoogle Cloudリソースの現状調査、認証主体とprojectの確認、Service Account Impersonation、設定変更、反映確認を安全に行う。Google Cloud CLIのログイン、AWS AssumeRole相当の権限借用、named configuration管理、API・IAM・Service Account・プロジェクト設定の確認や更新を依頼されたときに使用する。Google Auth Platformの一般的なOAuth Client作成だけを行う場合はCloud Consoleを使う。
---

# Google Cloud CLI Operations

人のログイン、アプリ用ADC、ワークロード資格情報を混同せず、調査主体と変更主体を分離して作業する。

## 認証方式を選ぶ

| 用途 | 方式 |
|---|---|
| 人がgcloudを操作 | `gcloud auth login` |
| 一時的に別権限を借用 | Service Account Impersonation |
| ローカルアプリがClient Libraryを使用 | `gcloud auth application-default login`またはImpersonation付きADC |
| Google Cloud上のワークロード | 実行リソースへService Accountをattach |
| AWSなど外部環境のワークロード | Workload Identity Federation |

- Service Account ImpersonationをAWS STS `AssumeRole`相当として扱う。
- Service Account Key JSONは長期秘密鍵である。ユーザーが明示し、Impersonation、attach、Workload Identity Federationを使えない場合を除いて作成しない。
- Google OAuth Client ID／SecretはアプリのOAuth資格情報であり、gcloudの管理者ログインには使わない。
- `gcloud auth login`の資格情報とApplication Default Credentials（ADC）は別管理である。gcloudの認証失敗を`gcloud auth application-default login`で直そうとしない。

## 1. 環境、主体、projectを確認する

Google Cloud APIを呼ぶ前に、認証を上書きする環境変数が存在するかを値を表示せず確認する。

```powershell
$taskGcloudOverrideNames = @(
  'CLOUDSDK_ACTIVE_CONFIG_NAME',
  'CLOUDSDK_AUTH_ACCESS_TOKEN',
  'CLOUDSDK_AUTH_ACCESS_TOKEN_FILE',
  'CLOUDSDK_AUTH_CREDENTIAL_FILE_OVERRIDE',
  'CLOUDSDK_AUTH_IMPERSONATE_SERVICE_ACCOUNT',
  'GOOGLE_APPLICATION_CREDENTIALS'
)

$taskGcloudOverrideNames | ForEach-Object {
  [pscustomobject]@{
    Name = $_
    IsSet = -not [string]::IsNullOrWhiteSpace([Environment]::GetEnvironmentVariable($_))
  }
}
```

次を取得する。

```powershell
gcloud --version
gcloud auth list --format=json
gcloud config configurations list --format=json
gcloud config list --all --format=json
```

- active account、active configuration、project、region、zone、Impersonation先を確認する。
- 環境変数、credential file、access token fileによる上書きがある場合、active accountの表示だけで実行主体を判断しない。
- project IDをユーザーの依頼または実データから解決する。project名とproject IDを混同しない。
- organization、folder、billing accountを扱う場合は、それぞれのIDも取得して対象範囲を確定する。

## 2. 読み取り専用で調査する

依頼に必要な`list`、`describe`、`get`系操作は追加確認なしで実行できる。projectと必要なlocationを明示する。

```powershell
$taskProjectId = 'PROJECT_ID'

gcloud projects describe $taskProjectId --format=json
gcloud services list --enabled --project=$taskProjectId --format=json
```

- JSONを基本とし、必要な項目は`--format`で絞る。
- 同名リソース、リージョン、依存先、IAM binding、API有効状態を調べる。
- 読み取り権限不足を別アカウントや別projectで暗黙に補完しない。
- active projectに依存せず、リソースコマンドへ`--project`を明示する。
- gcloudがproject flagを持たないリソースでは、完全修飾リソース名または対象専用named configurationを使う。

## 3. gcloudのヘルプで実在する操作を確認する

```powershell
gcloud help
gcloud services --help
gcloud services enable --help
gcloud topic configurations
gcloud topic formats
```

- コマンド、flag、列挙値を推測しない。
- Alpha／Betaコマンドは安定版との差と変更リスクを明示する。
- `--quiet`は確認対象と影響が確定した自動実行だけに使う。探索中に付けない。
- PowerShellではバッククォート改行を使い、Bashのバックスラッシュ継続をそのまま貼らない。

## 4. named configurationで作業境界を作る

AWS named profileに近い単位としてnamed configurationを使える。

```powershell
gcloud config configurations create gcloud-readonly
gcloud config set account reader@example.com
gcloud config set project PROJECT_ID

gcloud config configurations create gcloud-admin
gcloud config set account operator@example.com
gcloud config set project PROJECT_ID

gcloud config configurations list
```

named configurationはIAM権限を制限しない。同じ高権限ユーザーを両方へ設定しても読み取り専用にはならない。権限分離には別principalまたはprivileged Service AccountへのImpersonationを使う。

単一コマンドでは`--configuration`を明示できる。

```powershell
gcloud projects describe PROJECT_ID `
  --configuration=gcloud-readonly `
  --format=json
```

## 5. 変更内容を確定する

変更前に次を整理する。

- 対象のorganization、folder、project、location、リソース完全名
- 現在値と変更後の値
- 必要なIAM permissionまたはrole
- 依存リソースと影響範囲
- 変更後に実行する取得コマンド
- ロールバックまたは無効化方法

API有効化、IAM binding追加、billing連携、公開範囲変更、鍵作成、削除は変更操作として扱う。認証失敗、対象不明、競合ありの状態で進まない。

## 6. 変更時だけ一時的に権限を借りる

privileged Service Accountを使う前にユーザーへ次を具体的に説明する。

- 変更対象と実行する操作
- 必要な主な書き込み権限
- 現在の調査主体では実行できない理由
- 影響範囲と検証方法

操作者は先に人のアカウントでログインする。

```powershell
gcloud auth login
gcloud auth list --format=json
```

Impersonationには対象Service Accountの`iam.serviceAccounts.getAccessToken`が必要であり、通常は`roles/iam.serviceAccountTokenCreator`を付与する。

1コマンドだけ借りる方式を優先する。

```powershell
$taskPrivilegedServiceAccount = 'gcloud-admin@PROJECT_ID.iam.gserviceaccount.com'

gcloud services enable SERVICE.googleapis.com `
  --project=PROJECT_ID `
  --impersonate-service-account=$taskPrivilegedServiceAccount
```

複数コマンドで継続する必要がある場合だけ、専用named configurationへ設定する。

```powershell
gcloud config set auth/impersonate_service_account $taskPrivilegedServiceAccount `
  --configuration=gcloud-admin
```

## 7. project、location、Impersonation先を明示して変更する

```powershell
gcloud <group> <mutating-command> ... `
  --project=PROJECT_ID `
  --impersonate-service-account=$taskPrivilegedServiceAccount
```

- active configurationの暗黙値だけに依存しない。
- 失敗時に別project、別region、別principalへ切り替えない。
- `add-iam-policy-binding`の前に現在のIAM policyを取得する。
- replace系操作では、他主体が管理するbindingや設定を消さない。
- 削除、鍵作成、organization policy、billing変更は対象を再確認する。

## 8. 反映を同じ主体で検証する

変更コマンドの成功だけで完了としない。同じprojectとImpersonation先を指定して取得し直す。

```powershell
gcloud services list `
  --enabled `
  --project=PROJECT_ID `
  --impersonate-service-account=$taskPrivilegedServiceAccount `
  --format=json
```

- 長時間処理はoperationの完了状態を確認する。
- IAM、API有効化、organization policyなど伝播時間がある変更を即時完了と断定しない。
- 要求値、不変であるべき既存値、影響範囲外の代表値を確認する。

## 9. 一時的な権限借用と認証を終了する

この作業でnamed configurationへImpersonationを設定した場合は解除する。

```powershell
gcloud config unset auth/impersonate_service_account `
  --configuration=gcloud-admin
```

この作業で人のアカウントを追加し、継続利用が不要な場合だけ失効させる。

```powershell
gcloud auth revoke operator@example.com
```

ADCも作成した場合は別に失効させる。

```powershell
gcloud auth application-default revoke
```

作業開始前から存在したactive configuration、環境変数、ADCを勝手に変更しない。残存する上書き設定がある場合は最終報告で示す。

## Google Auth Platformの境界

- 一般的なGoogle Sign-In用Web application OAuth ClientはGoogle Auth PlatformのClients画面で作成する。
- `gcloud iam oauth-clients`はWorkforce Identity FederationとIAP向けであり、一般的なGoogle Sign-In OAuth Clientの代替として使わない。
- OAuth Client Secretをログ、Markdown、コマンド履歴へ出力しない。
- Google OAuth Client ID／Secretをgcloud認証やService Account Keyの代わりに使わない。

## 完了報告

次を簡潔に報告する。

- 実行したprincipalまたはImpersonation先
- organization／folder／project／location
- 変更前後
- 検証結果
- 伝播中、権限不足、未完了事項

access token、refresh token、Service Account Key、OAuth Client Secret、認証URLを報告へ含めない。
