---
name: aws-cdk-cloudformation-design
description: AWS CDKまたはCloudFormationによるシステム設計・実装・レビューで、設定境界、命名、固定セキュリティ基準、更新・削除時の安全性を整理する。AWS以外の設計や単純なAWS CLI操作には使用しない。
---

# AWS CDK / CloudFormation Design

AWS CDKとCloudFormationを、単なるリソース作成コードではなく、設定・識別子・ライフサイクルを管理する契約として扱う。

## 参照する資料

- AWS CDKまたはCloudFormationを扱うときは、必ず[固定ベースライン](references/fixed-baseline.md)を読む。
- リソース名、名前空間、Construct ID、Stack名、configからの名前生成を設計・変更するときは、[命名と識別子](references/naming-and-identities.md)を読む。
- 既存Stackの更新、リファクタリング、置換、削除、再デプロイ、ドリフト対応を扱うときは、[変更と削除の安全性](references/change-safety.md)を読む。

必要な参照だけを読み、同じ規約をプロジェクト文書へ複製しない。プロジェクト固有の値はconfigまたはそのプロジェクトの設計書で管理する。

## 設計の進め方

1. 現在のCDKコード、合成テンプレート、デプロイconfig、既存Stackを確認する。
2. 値を「利用者が設定する値」「CDKが導出する値」「CloudFormation/CDKが生成する値」「CDK管理外の事前作業」に分ける。
3. 固定ベースラインとの差を洗い出し、差が必要なら理由、影響、確認方法を記録する。
4. Statefulリソース、公開URL、認証、外部参照について、更新・置換・削除時の挙動を明示する。
5. synthとテストで構造を確認し、既存環境への変更ではdiffで置換・削除・IAM拡張を確認する。
6. デプロイが依頼範囲に含まれる場合だけ反映し、Stack Outputと実リソースを確認する。

## 設定境界

- 同じ意味の値をconfig、CDK、Runtime、フロントエンドへ重複して持たせない。変換は一か所に集約し、生成結果を参照またはOutputで渡す。
- 未知のconfigキー、無効値、回復不能な競合は、原因が分かる形で停止する。固定値への暗黙的な置換や勝手な別名での再試行をしない。
- Secret本文、パスワード、アクセストークンをCDK context、CloudFormation Parametersの既定値、Output、静的配信物へ入れない。Secrets Manager等の参照名またはARNだけを扱う。
- アカウントID、リソースID、実ドメインなどの環境固有値をサンプルや共通スキルへ固定しない。
- モデル契約、外部IdP登録、既存証明書、既存Hosted Zoneなど、CloudFormationが安全に所有できない事前作業を明示する。IaC管理外であることは、未管理のまま放置することを意味しない。

## 成果物

設計またはレビューでは、少なくとも次を判断できる形にする。

- config項目と導出値の対応
- CDK管理内・管理外の境界
- 固定ベースラインとの差分
- 物理名を固定する対象と、CDK生成名へ任せる対象
- 更新で置換される可能性があるリソースとデータ影響
- デプロイ前の手動作業、diff確認点、デプロイ後の検証点
