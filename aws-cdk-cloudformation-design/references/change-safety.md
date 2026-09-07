# 変更と削除の安全性

## 変更前

1. 対象Stack、アカウント、リージョン、デプロイconfigを確定する。
2. Statefulリソース、認証、公開URL、外部参照、明示物理名を列挙する。
3. synth結果を前回と比較し、Logical ID、`DeletionPolicy`、`UpdateReplacePolicy`、IAM、Outputを確認する。
4. diffで作成、インプレース更新、置換、削除を区別する。

CDKのデプロイはDockerのマルチステージビルドのような段階選択ではない。CloudFormationが依存グラフと差分に基づいて必要なリソースを更新する。小さなコード変更でも、immutable propertyやLogical IDの変更があれば置換になり得る。

## Statefulリソース

- `RETAIN`はバックアップではない。Stack削除後はCloudFormation管理外の孤立リソースになる。
- 同じCDKを再デプロイしても、保持した既存リソースへ自動で再接続するとは限らない。再利用するなら明示的な参照、import、移行手順が必要である。
- `DESTROY`と自動削除は、再生成可能でデータ消失を受容できる対象に限定する。
- 置換時の保持とStack削除時の保持を区別し、合成テンプレートの両方のPolicyを確認する。
- バケットやロググループなど、サービスやカスタムリソースが残すものも削除後監査へ含める。

## ドリフトと手動変更

CDK管理リソースをコンソールやCLIで直接直すと、次回デプロイで戻る、競合する、またはテンプレートとの差が残る。緊急変更を行った場合は、変更理由を記録し、CDKへ反映してdiffとドリフトを解消する。

CDK管理外の契約、利用者、既存DNS、Secret値などは、所有者と手順を別に明記する。管理外だから削除してよい、またはStack削除の影響を受けない、と推測しない。

## 削除と再デプロイ

- 削除対象のStack ARNまたはStack名、アカウント、リージョンを読み取りで確認する。
- 削除前にRemovalPolicy、termination protection、共有リソース、外部参照を確認する。
- 削除完了後、保持リソース、ロググループ、IAM Role、カスタムリソース生成物、DNS、証明書、Secretsを棚卸しする。
- 残存物は所有Stack、最終利用、復旧・監査要否を確認してから削除する。
- 再デプロイ前に、グローバル一意名の競合とCDK管理外の事前作業を確認する。

## 反映後

- CloudFormation EventsとStack Outputsを確認する。
- 認証、主要API、ログ、メトリクス、アラームを実動作で確認する。
- Statefulリソースの参照先と暗号化、保持期間、タグ、IAMの実値を確認する。
- 失敗時は原因を隠すフォールバックを追加せず、CloudFormation Eventsと対象サービスのエラーから切り分ける。

## 公式資料

- [AWS CDKの識別子](https://docs.aws.amazon.com/cdk/v2/guide/identifiers.html)
- [AWS CDKのベストプラクティス](https://docs.aws.amazon.com/cdk/v2/guide/best-practices.html)
- [CDK RemovalPolicy](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.RemovalPolicy.html)
