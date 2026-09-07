# 固定ベースライン

この資料はAWS CDK / CloudFormation設計で毎回確認する共通基準である。プロジェクト固有値を固定する一覧ではない。例外が必要な場合は、理由、影響、確認方法を設計書へ残す。

## 固定する方針

| 対象 | 基準 |
|---|---|
| Cognito自己サインアップ | 既定は無効。ユーザーが公開登録を明示的に要求した場合だけ有効化し、不正登録対策と運用責任を併記する。外部IdPの有効化は自己サインアップ許可を意味しない |
| 公開アクセス | S3、データストア、管理APIは非公開を既定にし、公開が要件の入口だけを明示する |
| Secret | 本文をconfig、CDK context、テンプレート、Output、静的配信物へ保存しない。Secret参照だけをIaCで扱う |
| IAM | 利用するActionとResourceへ限定する。ワイルドカードがサービス仕様上必要なら、理由と境界を記録する |
| 暗号化 | 対応するStatefulリソースは保存時暗号化を有効にする。AWS管理キーか顧客管理キーかは回復・監査要件で決める |
| ログ保持 | 日数の唯一の規約は`system-design`スキルが管理する。本スキルへ数値を複製せず、その指定値をIaCとデプロイ後確認へ適用する |
| 削除方針 | Statefulリソースごとに削除時と置換時を決める。CDKのRemovalPolicyを使う場合も、合成後の`DeletionPolicy`と`UpdateReplacePolicy`を確認する |
| CDK依存関係 | CDKライブラリとCLIの互換性を確認し、プロジェクトのlockfileで再現可能にする |
| config検証 | 未知キーを拒否し、必須値、相互依存、リージョン、文字種、長さをデプロイ前に検証する |
| Stack Output | 接続に必要な非秘密値だけを出す。Secret、パスワード、トークンは出力しない |

Amazon Cognitoで自己登録を無効にすると、利用者は管理APIまたはフェデレーション経由で作成する。自己登録を有効にする場合は、単にUIへ登録リンクを出すだけでなく、検証、レート制御、不正利用、通知費用を設計対象にする。

## 条件付きで固定する値

| 条件 | 基準 |
|---|---|
| CloudFrontのViewer証明書にACMを使う | 証明書は`us-east-1`で発行またはインポートする |
| カスタムドメインを使う | FQDN、Hosted Zone、証明書は所有関係を確認できる個別設定にする。共通プレフィックスから推測しない |
| 複数リージョンへ配置する | 配置リージョンをconfigで明示し、リージョン固定サービスと外部参照の所在を個別に検証する |
| JavaScript/TypeScriptを公開する | ソースマップの生成・配置・公開範囲は`system-deploy-check`スキルを正本として確認する |

## 固定しないもの

- AWS CLIプロファイル名、アカウントID、リージョン、ドメイン、証明書ARN、既存リソースID
- ログ保持日数以外の環境別容量、タイムアウト、予算、スケーリング値
- すべての物理リソース名
- Statefulリソースに対する一律の`RETAIN`または`DESTROY`

これらは要件と環境に依存する。サンプル値を本番既定値へ昇格させない。

## 公式資料

- [AWS CDKのベストプラクティス](https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html)
- [Amazon Cognitoのユーザー作成ポリシー](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-admin-create-user-policy.html)
- [CloudFrontで使用する証明書の要件](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html)
- [CDK RemovalPolicy](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.RemovalPolicy.html)
