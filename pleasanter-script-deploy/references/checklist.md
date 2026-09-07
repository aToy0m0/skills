# Pleasanter スクリプト反映チェックリスト

## 初回に確認する情報

- 対象ベース URL。`.env` の `PLEASANTER_BASE_URL` から読み込む。
- フォルダー SiteId。
- 実際に変更するテーブル SiteId。
- 参照だけの SiteId と変更対象の SiteId。
- API キー。`.env` の `PLEASANTER_API_KEY` から読み込み、チャットへ貼らない。
- ログインユーザー名とパスワード。必要な場合は `.env` の `PLEASANTER_LOGIN_ID`、`PLEASANTER_LOGIN_PASSWORD` から読み込む。
- テストユーザーのロール、サイト権限、レコード更新権限。
- 作業環境がローカル、開発、検証、本番のどれか。
- 変更してよい範囲。

## API 例

サイト設定取得:

```javascript
import fs from 'node:fs';

function loadDotEnv(file = '.env') {
  if (!fs.existsSync(file)) return;
  for (const line of fs.readFileSync(file, 'utf8').split(/\r?\n/)) {
    const trimmed = line.trim();
    if (!trimmed || trimmed.startsWith('#')) continue;
    const match = trimmed.match(/^([A-Za-z_][A-Za-z0-9_]*)=(.*)$/);
    if (!match || process.env[match[1]]) continue;
    process.env[match[1]] = match[2].replace(/^['"]|['"]$/g, '');
  }
}

function requiredEnv(name) {
  const value = process.env[name];
  if (!value) throw new Error(`環境変数 ${name} が未設定です。`);
  return value;
}

function requiredEnvInt(name) {
  const value = requiredEnv(name);
  const parsed = Number.parseInt(value, 10);
  if (!Number.isInteger(parsed) || String(parsed) !== value.trim()) {
    throw new Error(`環境変数 ${name} は整数の SiteId で指定してください。`);
  }
  return parsed;
}

loadDotEnv();

const BASE = requiredEnv('PLEASANTER_BASE_URL').replace(/\/+$/, '');
const API_KEY = requiredEnv('PLEASANTER_API_KEY');
const TARGET_SITE_ID = requiredEnvInt('PLEASANTER_TARGET_SITE_ID');

async function api(apiPath, body = {}) {
  const res = await fetch(`${BASE}${apiPath}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json; charset=utf-8' },
    body: JSON.stringify({ ApiVersion: 1.1, ApiKey: API_KEY, ...body }),
  });
  const json = await res.json();
  if (json.StatusCode !== 200) {
    throw new Error(`${apiPath}: ${JSON.stringify(json)}`);
  }
  return json;
}

async function getSite(siteId) {
  const got = await api(`/api/items/${siteId}/getsite`);
  return got.Response.Data;
}

async function updateSite(siteId, mutate) {
  const site = await getSite(siteId);
  const settings = site.SiteSettings || {};
  mutate(settings, site);
  return api(`/api/items/${siteId}/updatesite`, {
    Title: site.Title,
    Body: site.Body || '',
    SiteSettings: settings,
  });
}
```

この例はプロジェクトごとに対象 SiteId の変数名を増やしてよい。ベース URL、API キー、SiteId をコードへ固定しない。API キーを成果物や公開ファイルへ混入させない。

## クライアントスクリプト投入の確認

- `SiteSettings.Scripts` が配列か。
- 既存スクリプトをタイトルで探すか、`Gantt: true` で探すか。
- 同名スクリプトが複数ないか。
- 置換後も `Gantt`、`Index`、`New`、`Edit` などの適用画面フラグが正しいか。
- スクリプト本文に環境固有値が残っていないか。

## ブラウザ検証の観点

- ログイン後に対象 URL へ到達できる。
- 対象ページのタイトル、SiteId、パンくずが期待通り。
- 追加 UI が 1 回だけ生成され、再描画で重複しない。
- 標準ボタン、期間変更、前後移動、保存、iframe 編集などで壊れない。
- 保存後に実データまたは画面表示が最新になる。
- エラー時にエラー表示が出る。
- 成功メッセージが積み重ならない。
- 短期表示、長期表示、短いバー、同日バーなど境界条件を見る。

## 完了報告に含める内容

- 変更したファイル。
- 反映した SiteId。
- API 反映結果。
- ブラウザで確認した URL と操作。
- 最新エクスポート、スクリーンショット、検証記録。
- 未検証項目がある場合は明記する。
