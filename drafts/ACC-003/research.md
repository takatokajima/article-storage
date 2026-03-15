# リサーチ結果: ACC-003

## 1. ACC User Management API

### エンドポイント仕様

**ベースURL**
```
https://developer.api.autodesk.com
```

### ユーザー管理エンドポイント一覧

| 目的 | メソッド | エンドポイント |
|---|---|---|
| ユーザー一覧取得 | GET | `/hq/v1/accounts/{account_id}/users` |
| ユーザー招待（アカウントレベル） | POST | `/hq/v1/accounts/{account_id}/users` |
| ユーザー更新 | PATCH | `/hq/v1/accounts/{account_id}/users/{user_id}` |
| プロジェクトユーザー追加 | POST | `/construction/admin/v1/projects/{project_id}/users` |
| プロジェクトユーザー一覧 | GET | `/construction/admin/v1/projects/{project_id}/users` |

### ユーザー招待 API

**POST /hq/v1/accounts/{account_id}/users**

アカウントレベルでユーザーを招待する。招待されたユーザーにはメール通知が届く。

**リクエストヘッダー**
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**リクエストボディ**
```json
{
  "email": "user@example.com",
  "company_id": "company-uuid",
  "role_ids": ["role-uuid-1"],
  "access_level": "account_admin" | "project_admin" | "user"
}
```

**access_level の選択肢**
| 値 | 説明 |
|---|---|
| `account_admin` | アカウント管理者。全プロジェクトにアクセス可能 |
| `project_admin` | プロジェクト管理者。割り当てられたプロジェクトを管理 |
| `user` | 一般ユーザー。割り当てられたプロジェクトのみ |

**レスポンス**
- 成功: HTTP 200 / 201 + 作成されたユーザーのJSON
- 失敗例:
  - 400（バリデーションエラー、メールアドレス形式不正）
  - 403（権限不足）
  - 409（既に招待済みのユーザー）
  - 429（レート制限超過）

### ユーザー一覧取得 API

**GET /hq/v1/accounts/{account_id}/users**

ページネーション対応。`limit` と `offset` パラメータでページ分割。

**クエリパラメータ**
| パラメータ | 型 | デフォルト | 説明 |
|---|---|---|---|
| `limit` | integer | 10 | 1ページあたりの件数（最大100） |
| `offset` | integer | 0 | 開始位置 |
| `email` | string | - | メールアドレスでフィルタ |
| `status` | string | - | ステータスでフィルタ（active/inactive/pending） |

**ユーザーステータス**
| ステータス | 説明 |
|---|---|
| `active` | 招待を受諾し、アクティブなユーザー |
| `pending` | 招待メールを送信済みだが未受諾 |
| `inactive` | 無効化されたユーザー |

### プロジェクトユーザー追加 API

**POST /construction/admin/v1/projects/{project_id}/users**

アカウントレベルで招待済みのユーザーをプロジェクトに追加する。

**リクエストボディ**
```json
{
  "email": "user@example.com",
  "products": [
    {
      "key": "projectAdministration",
      "access": "administrator"
    },
    {
      "key": "docs",
      "access": "user"
    }
  ],
  "industry_roles": ["role-id-1"]
}
```

**products の key 一覧（主要なもの）**
| key | 説明 |
|---|---|
| `projectAdministration` | プロジェクト管理 |
| `docs` | ドキュメント管理 |
| `build` | Build（施工管理） |
| `designCollaboration` | Design Collaboration |
| `insight` | Insight（分析） |
| `modelCoordination` | Model Coordination |

**access の選択肢**
| 値 | 説明 |
|---|---|
| `administrator` | サービスの管理者権限 |
| `user` | サービスの一般ユーザー権限 |
| `none` | アクセスなし |

### 認証方法

ACC-002と同じ2-legged OAuth2（Client Credentials）を使用。

**必要なスコープ**

| スコープ | 用途 |
|---|---|
| `account:read` | ハブ・プロジェクト・ユーザー情報の読み取り |
| `account:write` | ユーザーの招待・更新 |
| `data:read` | プロジェクトデータの読み取り |

### レート制限
- 上限: 約300リクエスト/分（Data Management APIと共通）
- User Management APIは一部エンドポイントで100リクエスト/分の制限あり
- 超過時: HTTP 429 + `Retry-After` ヘッダー
- 対策: ループ内で `Utilities.sleep(500)` を挟む（ユーザー操作はやや長めに待つ）

---

## 2. 冪等設計のポイント

### ユーザー招待の冪等性

1. **事前チェック**: GET APIでメールアドレスによるフィルタで既存ユーザーを確認
2. **409エラーのハンドリング**: 既に招待済みの場合は409が返る → スキップ扱い
3. **ステータス管理**: スプレッドシートにステータス列を設けて処理済みを記録

### プロジェクト追加の冪等性

1. プロジェクトのユーザー一覧を事前取得してキャッシュ
2. 既にプロジェクトに存在するユーザーはスキップ
3. 権限変更が必要な場合のみPATCHで更新

---

## 3. GAS実装の考慮事項

### バッチ処理 vs 個別処理
- User Management APIはバッチ招待をサポートしていない（1件ずつ処理）
- GASの実行制限（6分/30分）を考慮し、大量ユーザーの場合はページ分割が必要

### エラーハンドリング
- 1件のエラーで全体が止まらないように try-catch で囲む
- エラー内容をスプレッドシートに書き戻して可視化

### GASの実行制限

| 制限項目 | 無料アカウント | Google Workspace |
|---|---|---|
| スクリプト実行時間 | 6分/回 | 30分/回 |
| UrlFetchApp リクエスト数 | 20,000件/日 | 100,000件/日 |

---

## 4. account_id の取得方法

- ACCのURLから取得可能: `https://acc.autodesk.com/admin/accounts/{account_id}/...`
- API経由: `GET /project/v1/hubs` のレスポンスから hub_id を取得し、`b.` プレフィックスを除いたものが account_id

---

## 5. つまずきポイントまとめ

1. account_id と hub_id の関係（hub_id = `b.` + account_id）
2. アカウントレベル招待 → プロジェクト追加の2段階が必要
3. 既存ユーザーの409エラーを正常系として扱う
4. products の key 名が直感的でない（docs, build など小文字）
5. industry_roles の ID は事前に GET で取得する必要あり
6. メールアドレスの大文字小文字は区別されない（APIが正規化する）

---

## 6. 参考リンク

- [ACC User Management API 概要](https://aps.autodesk.com/en/docs/acc/v1/overview/field-guide/users/)
- [POST /hq/v1/accounts/:account_id/users](https://aps.autodesk.com/en/docs/acc/v1/reference/http/users-POST/)
- [GET /hq/v1/accounts/:account_id/users](https://aps.autodesk.com/en/docs/acc/v1/reference/http/users-GET/)
- [POST /construction/admin/v1/projects/:project_id/users](https://aps.autodesk.com/en/docs/acc/v1/reference/http/admin-projects-projectId-users-POST/)
- [APS 認証 v2](https://aps.autodesk.com/en/docs/oauth/v2/reference/http/gettoken-POST)
- [GAS UrlFetchApp 公式リファレンス](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app)
