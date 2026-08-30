# 仕事日記（shigoto-nikki）

日々のタスク管理と仕事日記を記録するPWA（Progressive Web App）。バックエンドやビルドツールは無く、静的ファイルのみで完結する。

## 構成

- `index.html` — アプリ本体。HTML/CSS/JSがすべて1ファイルに収まっている（フレームワーク不使用、ビルド不要）
- `manifest.json` — PWAマニフェスト（アプリ名、アイコン、テーマカラー等）
- `sw.js` — Service Worker。オフラインキャッシュ（`workdiary-v1`）を管理
- `icon-192.png` / `icon-512.png` — ホーム画面用アイコン

## 動かし方

ビルド不要。`index.html` を静的ファイルとして配信すればよい（例: `python3 -m http.server` や任意の静的ホスティング）。ルート直下で配信すること（`manifest.json`・`sw.js`・アイコンを絶対パス `/xxx` で参照しているため）。

## データの持ち方

サーバーやDBは無く、すべて `localStorage` に保存される。

- `tasks_YYYY-MM-DD` — その日の「今日必ずやること」タスク一覧（`{id, text, done}[]`）
- `draft_YYYY-MM-DD` — 保存前の日記フォームの下書き（`goal`, `ach`, `reflection`, `gratitude`）
- `diary_entries` — 保存済みの日記エントリ配列。各エントリは `{id, date, time, goal, ach, reflection, gratitude}`
  - `ach` は達成度インデックス: `0=達成できた, 1=だいたいできた, 2=できなかった`

日記は1日1エントリで、同じ日に再保存すると上書き（`todayIdx` で既存エントリを検索して更新）。

## 画面構成

2タブ構成（`switchTab()` で切替、DOM操作は素のJS）:

1. **記録する** (`pane-record`)
   - セクションA: タスク追加・チェック・削除、進捗バー（完了数/合計、%）
   - セクションB: 日記フォーム（目標／達成度3択／反省・改善点／感謝したこと）→「日記を保存する」
2. **履歴** (`pane-history`)
   - 保存済みエントリを新しい順に5件ずつページネーション表示

## 実装上の注意

- XSS対策として `esc()` でユーザー入力をエスケープしてから `innerHTML` に挿入している。新しくユーザー入力をDOMに描画する箇所を追加する際は必ず `esc()` を通すこと
- Service Workerのキャッシュ対象ファイルを追加・変更したら `sw.js` の `CACHE_URLS` と `CACHE_NAME`（バージョン文字列）を更新すること。更新しないと古いキャッシュが残り、変更が反映されない
- 日付キーはすべてローカルタイムの `YYYY-MM-DD`（`TODAY` 定数）ベース。UTC変換は行っていない
- 外部ライブラリ・CDN依存なし。追加する場合はオフライン動作（Service Worker配下）に影響しないか確認すること
