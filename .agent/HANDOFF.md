# 引き継ぎメモ

## 最終更新
2026-03-11

## 完了したこと
- GitHubからリポジトリをプル・初期化
- .agent/ フォルダの作成・初期化（AGENTS.md / MEMORY.md / HANDOFF.md）
- articles/ACC-002/ フォルダ構成の作成
- リサーチエージェントによるACC Data Management API + GAS実装調査
- ACC-002 記事の執筆（index.md）
- 図解ルールをAGENTS.mdに記録

## 残タスク
- [ ] ACC-002 の `published: false` → 著者が最終確認後 `true` にする
- [ ] EDITORIAL_CALENDAR.md の ACC-002 ステータスを「✍️ 執筆中」→「👀 レビュー中」に更新
- [ ] 実際の画像ファイル（スクリーンショット等）が必要な場合は `images/ACC-002/` に格納

## 次回の開始地点
1. `articles/ACC-002/index.md` を著者が確認
2. 実際にGASを動かして動作検証
3. 問題なければ `published: true` に変更してZennに公開

## 注意事項
- ACCカスタムインテグレーション登録を必ず確認（403エラーの主原因）
- 2-legged認証では `x-user-id` ヘッダーが必須
- フォルダ作成はCommandsエンドポイント（POST /commands）を使う
