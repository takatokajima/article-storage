# ACC-006 リサーチメモ

## 記事情報
- **タイトル**: ACC Cost Managementの基本的な使い方 ― 予算・契約・変更管理の全体像
- **タイプ**: 解説記事（explanation）
- **対象読者**: ACCを業務で使っているBIM担当者・建設プロジェクトマネージャー

## 調査内容

### ACC Cost Managementとは
- Autodesk Construction Cloud（ACC）に含まれるコスト管理モジュール
- 建設プロジェクトの予算策定から支払いまでを一元管理する機能
- 旧BIM 360 Cost Managementの後継サービス

### 5つの主要モジュール
1. **Budget（予算）** — プロジェクト全体の予算を費目コード別に管理
2. **Contract（契約）** — 元請・下請間の契約金額を管理
3. **Change Order（変更指示）** — 設計変更・追加工事に伴うコスト変更を追跡
4. **Direct Cost（直接費）** — 契約外の直接経費を記録
5. **Payment（支払い）** — 出来高に基づく支払申請・承認を管理

### Cost Management API エンドポイント
- ベースURL: `https://developer.api.autodesk.com/cost/v1/containers/{containerId}`
- 主要エンドポイント:
  - `GET /budgets` — 予算一覧
  - `GET /contracts` — 契約一覧
  - `GET /change-orders` — 変更指示一覧
  - `GET /payments/applications` — 支払申請一覧
  - `GET /expense-items` — 直接費一覧
- containerIdはプロジェクトIDから取得（`GET /cost/v1/projects/{projectId}`）

### 認証方式
- 2-legged OAuth2（Client Credentials）
- 必要スコープ: `data:read`, `account:read`（読み取りのみの場合）
- ACCカスタムインテグレーション登録が必要

### 活用ユースケース
1. **ERP連携** — 予算・契約データをSAP/Oracle等に自動同期
2. **レポート自動化** — 定期的にデータを取得しダッシュボード化
3. **予算超過アラート** — 予算消化率を監視し閾値超過時に通知
4. **変更指示の追跡** — CO発行状況をリアルタイムに把握

### 制約・注意点
- Cost Management APIは読み取り専用のエンドポイントが多い
- ページネーション対応が必要（limit/offset パラメータ）
- レート制限あり（429エラー対応が必要）
- containerIdの取得が必要（projectIdとは別のID体系）
- Cost Managementモジュールがプロジェクトで有効化されている必要がある

### 関連記事との連携
- ACC-002: GASでACCのフォルダ構成を一括作成する（認証・GASの基本）
- ACC-013（予定）: Cost Management API連携の詳細実装

## 参考URL
- https://aps.autodesk.com/en/docs/acc/v1/overview/field-guide/cost-management/
- https://aps.autodesk.com/en/docs/acc/v1/reference/http/cost-budgets-GET/
- https://aps.autodesk.com/en/docs/acc/v1/reference/http/cost-contracts-GET/
- https://aps.autodesk.com/en/docs/acc/v1/tutorials/cost-management/
