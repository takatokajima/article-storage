# article-management

Zenn記事の管理リポジトリです。AIエージェントチームを活用した定期的な技術記事の執筆・公開を行います。

## カバートピック

| カテゴリ | 内容 |
|---------|------|
| **Autodesk ACC** | API連携・自動化、運用・活用事例 |
| **点群処理** | 取得・処理技術、可視化・BIM連携活用 |
| **Unity MCP** | 実践的な活用例、プロジェクトでの応用 |
| **AI/LLM活用** | 建設・開発分野でのAI・LLM活用術 |

## ディレクトリ構成

```
.
├── articles/          # Zenn記事（Markdown）
├── books/             # Zenn本（連載形式）
├── images/            # 記事で使う画像
├── docs/              # 運用ドキュメント・戦略
│   ├── PUBLISHING_STRATEGY.md
│   ├── AGENT_TEAM_GUIDE.md
│   ├── EDITORIAL_CALENDAR.md
│   └── ARTICLE_TEMPLATE.md
└── .github/workflows/ # CI/CD（将来用）
```

## 使い方

```bash
# 記事のプレビュー
npm run preview

# 新しい記事を作成
npm run new:article

# 新しい本を作成
npm run new:book
```

## 投稿フロー

1. `docs/EDITORIAL_CALENDAR.md` から次のテーマを確認
2. `docs/ARTICLE_TEMPLATE.md` のテンプレートを元に下書き作成
3. `docs/AGENT_TEAM_GUIDE.md` に沿ってAIエージェントチームでレビュー・推敲
4. `articles/` に配置してプレビュー確認
5. mainブランチへマージで自動公開
