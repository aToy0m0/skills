---
name: prepare-public-release-docs
description: Prepare and audit repository-facing documentation before a public or external release. Use when creating or revising README files, publishing architecture diagrams, copying a local project into a Git repository, changing a repository from private to public, defining docs-related .gitignore exceptions, or checking staged files for credentials, internal documents, local paths, machine-specific names, project sequence numbers, migration notes, and other context that repository readers do not need.
---

# 公開前ドキュメント準備

公開する成果物を、初めてリポジトリを見る読者の視点で整える。README、公開用画像、`.gitignore`、Gitの実追跡対象を一体として検査する。リポジトリが現在Privateでも、将来Publicになる前提で同じ基準を適用する。

## 基本原則

- 公開対象は「製品・システムを理解し、利用・開発・運用するために必要な情報」に限定する。
- 作業履歴、会話履歴、コピー元、ローカル整理番号、個人PCの事情を成果物へ持ち込まない。
- READMEの記述だけでなく、画像内文字、ファイル名、Git追跡状態、ステージ済み差分を確認する。
- `.gitignore`があることを安全性の証明にしない。Gitが実際に追跡・追加するファイルを確認する。
- 検出した秘密値を応答やログへ再掲しない。種類とファイル位置だけを報告する。
- 不明なファイルを「たぶん安全」と判断しない。公開目的を説明できなければ除外候補とする。
- コミットやpushはユーザーが明示的に依頼した場合だけ行う。

## ワークフロー

### 1. 公開境界を確定する

1. リポジトリルート、remote、現在branch、公開予定範囲を確認する。
2. README、公開ドキュメント、画像、example設定、ライセンスなど、必要な成果物を列挙する。
3. 内部設計資料、調査メモ、作業記録、AI向け指示、認証情報、生成物、中間ファイルを除外対象として列挙する。
4. `git status -sb`、`git ls-files`、`git ls-files --others --exclude-standard`で意図と実状態を照合する。

作業ツリーに依頼外の変更がある場合は、対象ファイルを明示指定して扱う。`git add -A`を既定にしない。

LICENSEの要否は配布方針による。不在だけで自動的に失格にせず、公開条件として必要か未決定なら確認事項として報告する。

### 2. 読者に不要なローカル文脈を除去する

README、Markdown、設定例、図、画像内文字、ファイル名について次を確認する。

- [ ] ユーザー名、端末名、ホームディレクトリ、絶対パスがない
- [ ] ローカルのドライブ文字、WSLパス、作業用一時ディレクトリがない
- [ ] ローカル整理用の連番、日付接頭辞、コピー番号、章番号が製品名として残っていない
- [ ] 「別フォルダーからコピーした」「Git管理用に作った」などの移行経緯がない
- [ ] チャットでの依頼、判断経緯、修正会話、個人的メモがない
- [ ] 元プロジェクト名、社内呼称、担当者名など、読者に不要な識別子がない
- [ ] 内部URL、ローカルURL、IPアドレス、個人メールアドレスが不要に公開されていない
- [ ] READMEが現在の成果物を直接説明し、作成者のPC環境を前提にしていない

ローカル番号がディレクトリ名に必要でも、README本文、画像タイトル、公開ファイル名からは除去する。公開ファイル名は内容を表す英語のkebab-caseを優先する。

### 3. 秘密情報と環境固有情報を検査する

次のファイル・値を重点確認する。

- [ ] `.env`、`.env.*`、デプロイ用env、資格情報ファイルが除外されている
- [ ] AWS access key、secret key、session token、GitHub token、API keyがない
- [ ] private key、証明書、Cookie、JWT、OAuth client secretがない
- [ ] AWSアカウントID、実ARN、実S3 bucket名、実endpointを不要に固定記述していない
- [ ] ログ、テスト結果、障害ダンプにtokenやrequest headerが含まれていない
- [ ] exampleファイルはダミー値または明確なplaceholderだけを含む
- [ ] `.env*`のような広い除外ルールが、公開する`.env.example`まで除外していない
- [ ] 認証情報を含む可能性がある画像、スクリーンショット、PDFがない

論理リソース名や一般的なサービス名は、構成理解に必要なら公開してよい。アカウント固有値や運用上の秘密値とは区別する。

秘密がGit履歴へ入った可能性がある場合、作業ツリーが安全でも合格にしない。pushを止め、履歴調査と認証情報の失効・再発行が必要であることを報告する。

### 4. 開発中の中間ファイルを除外する

技術スタックに応じて次を確認する。

- [ ] dependency directoryとpackage-manager cache
- [ ] build、bundle、framework、IaC synthの出力
- [ ] coverage、E2E report、screenshot、trace、test result
- [ ] runtime artifact、ZIP、container build contextの一時生成物
- [ ] editor、OS、AI agent、local toolの設定ディレクトリ
- [ ] ログ、PID、lock、temporary、swapファイル
- [ ] 内部docs、作業メモ、未確定設計、調査記録

lockfile、migration、生成コードなどは一律除外しない。再現可能性とプロジェクト規約から追跡要否を判断する。

### 5. 公開用構成図を検査する

draw.ioなどの編集可能な正本とPNGがある場合は次を実施する。

- [ ] 最新の正本を一意に特定する
- [ ] 正本の更新日時がPNGより新しければ再生成する
- [ ] 可能なら最新版の同一レンダラーでPNGを再生成する
- [ ] 複数ページは各ページを正しいページ番号で個別に出力する
- [ ] PNGの英語ファイル名が内容を説明している
- [ ] 図タイトルからローカル連番、コピー名、作業用接頭辞を除去する
- [ ] アカウント、リージョン、信頼境界、所有境界が正しい
- [ ] 通信、データ、管理操作、IAM許可を混同していない
- [ ] 矢印方向、認証主体、保存先、実行主体が実装・IaCと一致する
- [ ] 公式アイコン、ラベル、余白、重なり、切れを確認する
- [ ] 画像のfile signature、形式、寸法を確認し、拡張子だけ画像のplaceholderや破損ファイルを拒否する
- [ ] PNGに編集用XML、ローカルパス、不要なtext metadataが埋め込まれていない
- [ ] READMEの相対リンクが実在するPNGを指す

AWS構成図で対応スキルが利用可能なら、それを使って公式アイコン検証、レンダリング、実装照合、独立レビューまで行う。目視だけで合格にしない。

### 6. `.gitignore`例外を安全に設計する

内部文書ディレクトリ全体を除外し、その中の公開画像だけを追跡する場合、親ディレクトリを再包含してから対象ファイルを再包含する。

```gitignore
/docs/*
!/docs/architecture/
/docs/architecture/*
!/docs/architecture/system-architecture.png
```

単に `!/docs/architecture/system-architecture.png` を追加するだけでは、除外された親ディレクトリをGitが探索できない場合がある。

次を必ず実行して挙動を確認する。

```bash
git status --short --untracked-files=all
git check-ignore -v -- docs/internal-note.md
git check-ignore -v -- docs/architecture/system-architecture.png
git ls-files --others --exclude-standard
git add --dry-run -- README.md .gitignore docs/architecture/system-architecture.png
```

`git check-ignore -v`の出力だけで判断せず、`git status`と`git ls-files --others --exclude-standard`で対象PNGが追加候補になり、内部文書が候補にならないことを確認する。

### 7. READMEを公開物として仕上げる

- [ ] 冒頭でシステムの目的と主要構成を説明する
- [ ] 初見の読者に必要な前提条件、設定、検証、実行・デプロイ方法がある
- [ ] ローカルでのみ成立する説明を一般化するか削除する
- [ ] 実値を記述せず、exampleとplaceholderから設定できる
- [ ] 構成図セクションの画像参照はリポジトリ相対パスである
- [ ] リンク切れ、存在しないコマンド、古い名称がない
- [ ] コピー元や公開準備作業を説明するメタコメントがない
- [ ] 内部文書を参照しない

READMEから公開対象外ファイルへリンクしない。GitHub上で追跡されるファイルだけでREADMEが成立することを確認する。

### 8. ステージ後の最終ゲートを通す

push前に対象ファイルだけを明示してステージし、次を確認する。

```bash
git status -sb
git diff --cached --name-status
git diff --cached --stat
git diff --cached --check
git ls-files docs
```

さらに、ステージ済み差分と新規バイナリを対象に、秘密値、絶対パス、ローカル識別子を検索する。画像は目視に加え、文字列とmetadataを検査する。

次のいずれかがあればpushしない。

- 対象外ファイルがステージされている
- 内部文書が追跡されている
- 秘密または実環境固有値の疑いがある
- READMEの画像リンク先が未追跡または存在しない
- 図と実装・IaCの不一致がある
- ローカル文脈がREADME、画像、ファイル名に残る
- 検証結果を説明できない

## 完了報告

結果を次の形式で簡潔に報告する。

1. 公開対象ファイル
2. 除外を維持した内部ファイル群
3. READMEと構成図の修正内容
4. 秘密情報・ローカル情報の検査結果
5. Git追跡・ignore・ステージ確認結果
6. 実施した検証と未実施事項
7. commit／pushを行った場合はbranchとcommit ID

秘密値そのものは報告へ含めない。合格項目だけでなく、残存リスクや未確認範囲があれば明示する。
