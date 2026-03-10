# プロジェクト記憶

## 技術的な知識・決定事項

### ACC Data Management API（2025-03-11 確認）
- フォルダ作成は `POST /data/v1/projects/{project_id}/commands` (Commandsエンドポイント)を使う
  - `POST /projects/:id/folders` ではないので注意（よくある誤解）
- リクエストボディはJSON:API形式（`jsonapi`, `data`, `included` のネスト構造）
- 2-legged認証でフォルダ操作する場合は `x-user-id` ヘッダーが必須
- ACCプロジェクトIDは `b.` プレフィックス付き（例: `b.a4f95080-...`）
- ACCカスタムインテグレーション登録を忘れると403エラーになる

### GAS実装のポイント
- 2-legged認証は外部ライブラリ不要（`Utilities.base64Encode` + `UrlFetchApp`）
- 実行時間制限: 無料6分 / Google Workspace 30分
- レート制限対策: ループ内で `Utilities.sleep(200)` を挟む
- 冪等設計: スプレッドシートに作成済みIDを書き戻し、409は「既存」として扱う

## プロジェクト構成
- 記事フォルダ: `articles/{記事ID}/`（index.md, outline.md, research.md）
- 画像フォルダ: `images/{記事ID}/`
- 運用ドキュメント: `docs/`

## 記事作成履歴
| 記事ID | タイトル | ステータス |
|--------|---------|----------|
| ACC-002 | GASでACCのフォルダ構成を一括作成する | 執筆中 |
