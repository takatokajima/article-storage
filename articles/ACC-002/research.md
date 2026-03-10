# リサーチ結果: ACC-002

## 1. ACC Data Management API

### エンドポイント仕様

**ベースURL**
```
https://developer.api.autodesk.com
```

**フォルダ作成エンドポイント**

ACC/BIM360 のフォルダ作成は `POST /data/v1/projects/:project_id/commands`（Commandsエンドポイント）を使用する。

**project_id の注意点**
- ACCのプロジェクトIDには `b.` プレフィックスが付く（例: `b.a4f95080-84fe-4281-8d0a-bd8c88xxxxxx`）

**リクエストヘッダー**
```
Authorization: Bearer {access_token}
Content-Type: application/vnd.api+json
x-user-id: {autodesk_user_id}   ← 2-legged認証の場合は必須
```

**リクエストボディ（JSON形式）**
```json
{
  "jsonapi": { "version": "1.0" },
  "data": {
    "type": "commands",
    "attributes": {
      "extension": {
        "type": "commands:autodesk.core:CreateFolder",
        "version": "1.0.0",
        "data": { "requiredAction": "create" }
      }
    },
    "relationships": {
      "resources": {
        "data": [{ "type": "folders", "id": "1" }]
      }
    }
  },
  "included": [{
    "type": "folders",
    "id": "1",
    "attributes": {
      "name": "作成するフォルダ名",
      "extension": {
        "type": "folders:autodesk.bim360:Folder",
        "version": "1.0"
      }
    },
    "relationships": {
      "parent": {
        "data": {
          "type": "folders",
          "id": "親フォルダのID（URN）"
        }
      }
    }
  }]
}
```

**ネスト上限**: 最大25階層まで

**レスポンス**
- 成功: HTTP 200 + 作成されたフォルダのJSON
- 失敗例: 403（権限不足）、409（同名フォルダが既に存在）、429（レート制限超過）

### 関連エンドポイント一覧

| 目的 | メソッド | エンドポイント |
|---|---|---|
| ハブ一覧取得 | GET | `/project/v1/hubs` |
| プロジェクト一覧取得 | GET | `/project/v1/hubs/{hub_id}/projects` |
| トップフォルダ取得 | GET | `/project/v1/hubs/{hub_id}/projects/{project_id}/topFolders` |
| フォルダ内容取得 | GET | `/data/v1/projects/{project_id}/folders/{folder_id}/contents` |
| フォルダ作成 | POST | `/data/v1/projects/{project_id}/commands` |

### 認証方法

#### 2-legged（Client Credentials）

バックエンド間通信・自動化スクリプトに適した方式。ユーザーログイン不要。

**トークン取得エンドポイント（v2）**
```
POST https://developer.api.autodesk.com/authentication/v2/token
```

**ヘッダー**
```
Authorization: Basic {Base64(client_id:client_secret)}
Content-Type: application/x-www-form-urlencoded
```

**ボディ**
```
grant_type=client_credentials&scope=data:read data:write data:create account:read
```

**重要**: v2では client_id と client_secret をボディではなく `Authorization: Basic` ヘッダーに含める。

### 必要なスコープ

| スコープ | 用途 |
|---|---|
| `data:read` | フォルダ・ファイルの読み取り |
| `data:write` | フォルダ・ファイルの書き込み |
| `data:create` | 新規フォルダ・ファイルの作成 |
| `account:read` | ハブ・プロジェクト情報の読み取り |

### レート制限
- 上限: 約300リクエスト/分
- 超過時: HTTP 429 + `Retry-After` ヘッダー
- 対策: ループ内で `Utilities.sleep(200)` を挟む

---

## 2. GAS実装

### 基本的なAPI呼び出し

```javascript
function getAccessToken(clientId, clientSecret) {
  const credentials = Utilities.base64Encode(clientId + ':' + clientSecret);
  const options = {
    method: 'post',
    headers: {
      'Authorization': 'Basic ' + credentials,
      'Content-Type': 'application/x-www-form-urlencoded'
    },
    payload: 'grant_type=client_credentials&scope=data:read%20data:write%20data:create%20account:read',
    muteHttpExceptions: true
  };
  const response = UrlFetchApp.fetch(
    'https://developer.api.autodesk.com/authentication/v2/token',
    options
  );
  return JSON.parse(response.getContentText()).access_token;
}
```

### GASの実行制限

| 制限項目 | 無料アカウント | Google Workspace |
|---|---|---|
| スクリプト実行時間 | 6分/回 | 30分/回 |
| UrlFetchApp リクエスト数 | 20,000件/日 | 100,000件/日 |

---

## 3. スプレッドシート設計

### アプローチA：インデント列方式（推奨）

| A列（レベル1） | B列（レベル2） | C列（レベル3） | D列（フォルダID） | E列（ステータス） |
|---|---|---|---|---|
| 01_WIP | | | urn:adsk... | 作成済 |
| 01_WIP | 建築 | | urn:adsk... | 作成済 |
| 01_WIP | 建築 | 図面 | | 未作成 |

---

## 4. つまずきポイントまとめ

1. Commandsエンドポイントを使う（直感に反する）
2. x-user-id ヘッダーが必須（2-leggedの場合）
3. ACCカスタムインテグレーション登録を忘れると403
4. `b.` プレフィックス付きのプロジェクトIDを使う
5. 409エラーは「既存として扱う」冪等設計が重要

---

## 5. 参考リンク

- [APS Data Management API 概要](https://aps.autodesk.com/developer/overview/data-management-api)
- [POST /projects/:project_id/commands](https://aps.autodesk.com/en/docs/data/v2/reference/http/projects-project_id-commands-POST)
- [APS 認証 v2](https://aps.autodesk.com/en/docs/oauth/v2/reference/http/gettoken-POST)
- [GAS UrlFetchApp 公式リファレンス](https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app)
- [GAS ベストプラクティス](https://developers.google.com/apps-script/guides/support/best-practices)
