# ACC-001 リサーチメモ

## 使用するAPIエンドポイント

| エンドポイント | メソッド | 用途 |
|---|---|---|
| `/authentication/v2/token` | POST | 2-legged OAuth2 トークン取得 |
| `/project/v1/hubs` | GET | ハブ（組織）一覧の取得 |
| `/project/v1/hubs/{hub_id}/projects` | GET | プロジェクト一覧の取得 |

## 認証方式

- 2-legged OAuth2（Client Credentials Grant）
- Basic認証ヘッダー: `Base64(client_id:client_secret)`
- スコープ: `data:read account:read`

## レスポンス形式

- JSON:API 仕様（https://jsonapi.org/）
- トップレベルに `data` キー、各オブジェクトに `type`, `id`, `attributes`, `relationships`

## 注意点

- Forge → APS リブランド（2023年）。エンドポイントURLは変わらない
- ハブIDは `b.` プレフィックス（BIM 360 / ACC）
- ACCカスタムインテグレーション登録が必須（2-legged認証の場合）
