# ドキュメントテンプレート詳細

このファイルには、各ドキュメントの詳細なテンプレートと記述ガイドラインを含みます。

---

## 1. overview.md 詳細テンプレート

### セクション1: プロジェクト概要

```markdown
# {プロジェクト名} 解説書

## 1. プロジェクト概要

### 1.1 目的
本プロジェクトは、**{主要機能の説明}**を目的としたアプリケーションです。

{2-3文で背景と解決する課題を説明}

### 1.2 主な機能
1. **{機能1}**: {説明}
2. **{機能2}**: {説明}
3. **{機能3}**: {説明}
{必要に応じて追加}

### 1.3 対象ユーザー
- {ユーザータイプ1}: {どのような用途で使うか}
- {ユーザータイプ2}: {どのような用途で使うか}
```

### セクション2: システム構成

アーキテクチャ図は以下のパターンから選択:

#### パターンA: Webアプリケーション（シンプル）
```mermaid
graph TB
    subgraph ユーザー
        Browser[Web ブラウザ]
    end

    subgraph サーバー環境
        App[Webアプリケーション]
        DB[(データベース)]
    end

    Browser --> App
    App --> DB
```

#### パターンB: マイクロサービス / 外部連携あり
```mermaid
graph TB
    subgraph ユーザー
        Browser[Web ブラウザ]
    end

    subgraph Docker環境
        subgraph webコンテナ
            App[アプリケーション]
        end
        subgraph dbコンテナ
            DB[(データベース)]
        end
    end

    subgraph 外部サービス
        ExtAPI1[外部API 1]
        ExtAPI2[外部API 2]
    end

    Browser --> App
    App --> DB
    App --> ExtAPI1
    App --> ExtAPI2
```

#### パターンC: フロントエンド/バックエンド分離
```mermaid
graph TB
    subgraph ユーザー
        Browser[Web ブラウザ]
    end

    subgraph フロントエンド
        FE[SPA / SSR]
    end

    subgraph バックエンド
        API[API サーバー]
        DB[(データベース)]
        Cache[(キャッシュ)]
    end

    Browser --> FE
    FE --> API
    API --> DB
    API --> Cache
```

### セクション3: 処理フロー

シーケンス図のパターン:

#### CRUD操作
```mermaid
sequenceDiagram
    participant U as ユーザー
    participant W as Webサーバー
    participant DB as データベース

    U->>W: リクエスト
    W->>DB: クエリ実行
    DB-->>W: 結果
    W-->>U: レスポンス
```

#### 外部API連携
```mermaid
sequenceDiagram
    participant U as ユーザー
    participant W as Webサーバー
    participant E as 外部API
    participant DB as データベース

    U->>W: リクエスト
    W->>E: API呼び出し
    E-->>W: レスポンス
    W->>DB: 結果保存
    DB-->>W: 確認
    W-->>U: 完了通知
```

### セクション4: データモデル

ER図の記述ルール:
- `||--o{`: 1対多
- `||--||`: 1対1
- `}o--o{`: 多対多

```mermaid
erDiagram
    User ||--o{ Entity : "has"
    User {
        int id PK
        string name
        string email UK
        datetime created_at
    }
    Entity {
        int id PK
        int user_id FK
        string data
    }
```

### セクション5: ファイル構成

ツリー形式で表示:

```
project/
├── app/                  # メインアプリケーション
│   ├── __init__.py
│   ├── main.py           # エントリーポイント
│   ├── config.py         # 設定管理
│   ├── models.py         # データモデル
│   └── ...
├── tests/                # テスト
├── docs/                 # ドキュメント
├── docker-compose.yml    # コンテナ定義
├── Dockerfile            # ビルド定義
├── requirements.txt      # 依存関係
└── README.md             # 簡易説明
```

### セクション6: 主要モジュール解説

各モジュールの説明フォーマット:

```markdown
### 6.1 {モジュール名}（{ファイル名}）

**役割**: {1文で説明}

**主要クラス/関数**:
- `{クラス/関数名}`: {説明}
- `{クラス/関数名}`: {説明}

**依存関係**:
- {依存モジュール1}
- {依存モジュール2}

**特記事項**:
{重要な実装詳細やデザイン決定}
```

### セクション7: 環境変数

表形式:

| 変数名 | 必須 | 説明 | デフォルト値 |
|--------|------|------|-------------|
| `VAR_NAME` | 必須/- | 説明文 | `default` or `-` |

---

## 2. api-reference.md 詳細テンプレート

### エンドポイント記述フォーマット

```markdown
### {機能名}

```
{METHOD} {PATH}
```

**説明**: {1文で機能を説明}

**パラメータ**:
| 名前 | 型 | 必須 | 説明 |
|------|-----|------|------|
| param1 | string | 必須 | 説明 |
| param2 | int | - | 説明（デフォルト: 10） |

**リクエスト例**:
```json
{
  "key": "value"
}
```

**レスポンス例**:
```json
{
  "status": "success",
  "data": {}
}
```

**エラーコード**:
| コード | 説明 |
|--------|------|
| 400 | リクエスト不正 |
| 404 | リソースが見つからない |
```

---

## 3. setup-guide.md 詳細テンプレート

### 環境変数ファイル例

```markdown
### 環境変数ファイル作成

`.env.example` を参考に `.env` を作成:

```env
# アプリケーション設定
APP_ENV=dev
APP_SECRET_KEY=change-this-in-production

# データベース設定
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname

# 外部サービス設定
API_KEY=your-api-key-here
```
```

### Docker運用コマンド

```markdown
### コンテナ操作

```bash
# 起動
docker compose up -d

# 停止
docker compose down

# 再ビルド
docker compose up -d --build

# ログ確認
docker compose logs -f {service}
```
```

### トラブルシューティングフォーマット

```markdown
#### {エラー名/症状}

**症状**:
```
{エラーメッセージ}
```

**原因**: {原因の説明}

**対処**:
1. {ステップ1}
2. {ステップ2}
```

---

## 4. 言語別の考慮事項

### Python プロジェクト

確認するファイル:
- `requirements.txt` / `pyproject.toml` / `setup.py`
- `main.py` / `app.py` / `__main__.py`
- `config.py` / `settings.py`
- `models.py`
- `Dockerfile`

フレームワーク検出:
- FastAPI: `from fastapi import`
- Django: `django.conf.settings`
- Flask: `from flask import`

### Node.js / TypeScript プロジェクト

確認するファイル:
- `package.json`
- `src/index.ts` / `src/main.ts` / `index.js`
- `tsconfig.json`
- `.env.example`

フレームワーク検出:
- Express: `express()`
- Next.js: `next.config.js`
- NestJS: `@nestjs/`

### Go プロジェクト

確認するファイル:
- `go.mod`
- `main.go` / `cmd/`
- `internal/` / `pkg/`

---

## 5. 出力チェックリスト

生成完了前に確認:

- [ ] 全てのセクションが埋まっている
- [ ] Mermaid図が正しく描画される
- [ ] コード例が実行可能
- [ ] 秘密情報がマスキングされている
- [ ] 日本語として自然な文章
- [ ] 表のフォーマットが崩れていない
- [ ] リンクが正しい（相対パス）
