# リサーチ結果: ACC-005

## 1. ACC Reviews（レビュー）機能の概要

### ACCにおけるレビュー機能とは

ACCのReviews機能は、設計図書やBIMモデルの承認ワークフローを管理するための機能。ドキュメント管理モジュール内で利用可能。

**主な用途**:
- 図面・モデルの承認プロセス管理
- レビュアーへの依頼と回答の追跡
- 承認・差戻し・コメントの一元管理

### UI上の操作フロー

1. レビューを作成（タイトル、説明、期限を設定）
2. レビュー対象のドキュメント/ファイルを添付（Review Items）
3. レビュアーを指名（Reviewers）
4. レビュアーが各アイテムに対してステータスを返す（Approved / Rejected / Reviewed with comments）
5. すべてのレビュアーが回答するとレビューが完了

---

## 2. Reviews API エンドポイント一覧

### ベースURL
```
https://developer.api.autodesk.com/construction/reviews/v1/projects/{projectId}
```

### エンドポイント

| 目的 | メソッド | パス | 備考 |
|---|---|---|---|
| レビュー一覧取得 | GET | `/reviews` | ページネーション対応、フィルタ可能 |
| レビュー詳細取得 | GET | `/reviews/{reviewId}` | 単一レビューの全情報 |
| レビュー作成 | POST | `/reviews` | 新規レビューの作成 |
| レビュー更新 | PATCH | `/reviews/{reviewId}` | タイトル・説明・期限の変更 |
| レビュアー一覧取得 | GET | `/reviews/{reviewId}/reviewers` | レビュアーとその回答ステータス |
| レビュアー追加 | POST | `/reviews/{reviewId}/reviewers` | 新規レビュアーの追加 |
| レビューアイテム一覧 | GET | `/reviews/{reviewId}/review-items` | レビュー対象ドキュメント一覧 |

### 認証

- 2-legged OAuth2（Client Credentials）または 3-legged OAuth2
- 必要スコープ: `data:read`（読み取り）、`data:write`（書き込み）

### レスポンス構造

**レビュー一覧レスポンス（GET /reviews）**
```json
{
  "pagination": {
    "limit": 20,
    "offset": 0,
    "totalResults": 42
  },
  "results": [
    {
      "id": "review-uuid",
      "name": "構造図面レビュー第1回",
      "description": "S棟構造図面の中間レビュー",
      "status": "open",
      "createdBy": "user-uuid",
      "createdAt": "2025-03-01T09:00:00Z",
      "updatedAt": "2025-03-10T14:30:00Z",
      "dueDate": "2025-03-15T23:59:59Z",
      "reviewersCount": 3,
      "itemsCount": 5
    }
  ]
}
```

**レビュー詳細レスポンス（GET /reviews/{reviewId}）**
```json
{
  "id": "review-uuid",
  "name": "構造図面レビュー第1回",
  "description": "S棟構造図面の中間レビュー",
  "status": "open",
  "createdBy": "user-uuid",
  "createdAt": "2025-03-01T09:00:00Z",
  "updatedAt": "2025-03-10T14:30:00Z",
  "dueDate": "2025-03-15T23:59:59Z",
  "workflowType": "parallel"
}
```

**レビュアー一覧レスポンス（GET /reviews/{reviewId}/reviewers）**
```json
{
  "results": [
    {
      "id": "reviewer-uuid",
      "reviewId": "review-uuid",
      "userId": "user-uuid",
      "status": "approved",
      "step": 1,
      "respondedAt": "2025-03-05T10:00:00Z",
      "comment": "問題なし。承認します。"
    }
  ]
}
```

---

## 3. レビューのステータス遷移

### レビュー全体のステータス
- `draft` → 下書き。レビュアーにはまだ通知されない
- `open` → 進行中。レビュアーが回答できる状態
- `closed` → 完了。すべてのレビュアーが回答済み、またはオーナーが手動で閉じた

### レビュアー個別のステータス
- `pending` → 未回答
- `approved` → 承認
- `rejected` → 差戻し
- `reviewed` → コメント付きレビュー済み（承認でも差戻しでもない）

### ワークフロータイプ
- `parallel` → 全レビュアーが同時にレビュー可能
- `sequential` → ステップ順にレビュー（前のステップが完了するまで次のステップのレビュアーは回答不可）

---

## 4. データ構造の関係

```
Review (レビュー)
├── id, name, description, status, dueDate
├── createdBy (作成者のユーザーID)
├── workflowType (parallel / sequential)
│
├── Review Items (レビュー対象ドキュメント)
│   ├── id, reviewId
│   ├── itemId (Data Management上のファイルID)
│   ├── versionId (ファイルのバージョンID)
│   └── name (ファイル名)
│
└── Reviewers (レビュアー)
    ├── id, reviewId, userId
    ├── status (pending / approved / rejected / reviewed)
    ├── step (sequential時のステップ番号)
    ├── respondedAt (回答日時)
    └── comment (コメント)
```

---

## 5. APIの制約・注意点

### できること
- レビュー一覧・詳細の取得
- レビューの作成・更新
- レビュアーの追加・一覧取得
- レビューアイテムの取得
- ステータスでのフィルタリング

### できないこと / 制約
- レビュアーの回答（承認・差戻し）はAPIからは不可 → UI操作が必要
- レビューの削除はサポートされていない（closedにするのみ）
- 添付ファイルへのマークアップ（図面への書き込み）はAPIで取得不可
- レート制限: 約300リクエスト/分（APS共通）

### 必要な前提条件
- ACCアカウントにDocument Management モジュールが有効であること
- カスタムインテグレーションがACCアカウントに登録されていること
- プロジェクトIDに `b.` プレフィックスが必要

---

## 6. 自動化ユースケース

### ユースケース1: 進捗ダッシュボード
- 全プロジェクトのレビュー状況を一覧で可視化
- オープン/クローズの割合、期限超過レビューの検出
- スプレッドシートやBI ツールへの自動エクスポート

### ユースケース2: リマインダー通知
- 未回答のレビュアーへの自動リマインダー
- 期限が近いレビューの通知
- Slack/Teamsへのウェブフック連携

### ユースケース3: 承認レポート
- 完了したレビューの承認結果レポート自動生成
- 監査証跡としての承認履歴の記録
- CSV/PDFレポートの自動作成・メール送信

---

## 7. 参考リンク

- [ACC Reviews API ドキュメント](https://aps.autodesk.com/en/docs/acc/v1/reference/http/reviews-GET/)
- [ACC Reviews API 概要](https://aps.autodesk.com/en/docs/acc/v1/overview/field-guide/reviews/)
- [APS 認証 v2 リファレンス](https://aps.autodesk.com/en/docs/oauth/v2/reference/http/gettoken-POST)
- [ACC API フィールドガイド](https://aps.autodesk.com/en/docs/acc/v1/overview/field-guide/)
- [Google Apps Script UrlFetchApp](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app)
