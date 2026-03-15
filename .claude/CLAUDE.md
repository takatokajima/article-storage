# article-management

## 概要
Zenn記事の管理リポジトリ。AIエージェントチームを活用した技術記事の執筆・公開を行う。

## カバートピック
- Autodesk ACC（API連携・自動化、運用・活用事例）
- 点群処理（取得・処理技術、可視化・BIM連携）
- Unity MCP（実践的な活用例）
- AI/LLM活用（建設・開発分野）

## ディレクトリ構成
```
articles/          # Zenn記事（直下に スラッグ.md を配置。Zennはここだけ認識する）
drafts/            # 記事の作業ファイル（リサーチ・アウトライン等）。ACC-002/ のように記事IDでサブディレクトリを切る
books/             # Zenn本（連載形式）
images/            # 記事で使う画像
docs/              # 運用ドキュメント・戦略
.agent/            # セッション引き継ぎ（MEMORY.md, HANDOFF.md）
```

## Zenn記事のファイル命名規則
- `articles/` 直下にフラットに配置（サブディレクトリNG）
- ファイル名 = Zennのスラッグ（小文字英数字とハイフンのみ、12〜50文字）
- 例: `acc-002-gas-acc-folder-creation.md`
- 作業ファイル（research.md, outline.md等）は `drafts/記事ID/` に格納

## 開発コマンド
- プレビュー: `npm run preview`
- 新規記事: `npm run new:article`
- 新規本: `npm run new:book`
