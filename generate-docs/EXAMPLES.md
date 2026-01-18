# ドキュメント生成の実行例

このファイルには、スキルの実行例と期待される出力を示します。

---

## 基本的な使用例

### 例1: デフォルト出力先

```
/generate-docs
```

`docs/` ディレクトリに以下のファイルが生成されます:
- `docs/index.md`
- `docs/overview.md`
- `docs/api-reference.md`
- `docs/setup-guide.md`

### 例2: カスタム出力先

```
/generate-docs documentation/
```

`documentation/` ディレクトリに生成されます。

---

## 分析プロセスの例

### Python FastAPIプロジェクトの場合

#### ステップ1: 構造把握

```
プロジェクト構造を確認中...

ファイル構成:
├── app/
│   ├── main.py          <- エントリーポイント検出
│   ├── config.py        <- 設定ファイル検出
│   ├── models.py        <- ORMモデル検出
│   ├── db.py            <- DB接続検出
│   └── ...
├── docker-compose.yml   <- Docker構成検出
├── requirements.txt     <- 依存関係検出
└── README.md            <- 既存ドキュメント検出
```

#### ステップ2: 主要要素の抽出

```
検出した要素:

フレームワーク: FastAPI
データベース: PostgreSQL (SQLAlchemy)
認証: セッションベース
外部API連携:
  - Dify (LLM)
  - Pleasanter (業務システム)

エンドポイント数: 15
モデル数: 4
環境変数数: 20
```

#### ステップ3: ドキュメント生成

```
生成中...

✓ docs/index.md (2.1 KB)
✓ docs/overview.md (15.3 KB)
✓ docs/api-reference.md (8.7 KB)
✓ docs/setup-guide.md (6.2 KB)

完了: 4ファイル生成 (32.3 KB)
```

---

## 出力サンプル

### overview.md の冒頭部分

```markdown
# Pleasanter-Dify Connector 解説書

## 1. プロジェクト概要

### 1.1 目的
本プロジェクトは、**Pleasanter**（業務管理システム）と**Dify**（LLMプラットフォーム）を連携し、
業務メールの自動要約と帳票作成を支援するWebアプリケーションです。

### 1.2 主な機能
1. **メール取得**: Pleasanterから案件に紐づくメールを取得
2. **クレンジング**: メール本文から不要部分を除去
3. **AI要約**: Dify経由でLLMがメールを要約
4. **フォーム編集**: 要約結果をユーザーが確認・編集
5. **保存**: 編集結果をPleasanterに反映
```

### api-reference.md のエンドポイント例

```markdown
### メール要約

```
POST /api/summarize_email
Content-Type: application/json
```

**説明**: メール本文を送信してAI要約を実行

**パラメータ**:
| 名前 | 型 | 必須 | 説明 |
|------|-----|------|------|
| email_text | string | 必須 | 要約対象のメール本文 |
| conversation_id | string | - | 既存会話ID（空なら新規作成） |

**リクエスト例**:
```json
{
  "email_text": "お世話になっております。先日の件について...",
  "conversation_id": ""
}
```

**レスポンス例**:
```json
{
  "conversation_id": "abc123",
  "answer": "{\"llm_comment\":\"要約しました\",...}",
  "parsed": {
    "llm_comment": "要約しました",
    "DescriptionA": "問い合わせ対応の概要"
  }
}
```
```

---

## プロジェクトタイプ別の検出パターン

### Webアプリケーション

検出キーワード:
- `FastAPI`, `Flask`, `Django`, `Express`, `NestJS`
- `routes/`, `controllers/`, `views/`
- `templates/`, `static/`

生成する図:
- クライアント-サーバー構成図
- リクエスト処理シーケンス図

### CLI ツール

検出キーワード:
- `argparse`, `click`, `typer`, `commander`
- `bin/`, `cli/`, `commands/`

生成する図:
- コマンド実行フロー図
- サブコマンド構造図

### ライブラリ/SDK

検出キーワード:
- `setup.py`, `pyproject.toml` with `[project]`
- `src/`, `lib/`
- `__init__.py` with exports

生成する図:
- モジュール依存関係図
- クラス階層図

---

## エラーハンドリング

### ドキュメント生成失敗時

```
エラー: エントリーポイントが見つかりません

確認事項:
1. main.py, app.py, index.js などが存在するか
2. package.json, requirements.txt などが存在するか

解決策:
- プロジェクトのルートディレクトリで実行してください
- 手動でエントリーポイントを指定: /generate-docs --entry src/main.py
```

### 部分的な生成

```
警告: 一部の情報が取得できませんでした

- APIエンドポイント: 検出済み (15件)
- データモデル: 検出済み (4件)
- 環境変数: 一部のみ検出 (README.mdに記載なし)

続行しますか？環境変数は後で手動追加できます。
```
