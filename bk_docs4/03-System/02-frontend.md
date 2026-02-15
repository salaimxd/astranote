---
title: フロントエンド設計
description: Astro + React によるアイランドアーキテクチャ
---

Astranoteは、パフォーマンスとコスト効率を最大化するために **Astro Islands アーキテクチャ** を採用します。

## 技術スタック (Frontend)

- **Framework**: Astro (Static Site Generation / Server Side Rendering)
- **Interactive UI**: React (必要な箇所のみ `client:load` で使用)
- **Language**: TypeScript
- **Styling**: Tailwind CSS, shadcn/ui
- **State Management**: nanostores (軽量、フレームワーク非依存)
- **Hosting**: Cloudflare Pages

## アーキテクチャ設計

### Astro Islands

Webサイトの大部分（ランディングページ、小説本文表示など）は静的なHTMLとして生成し、JSを送信しません。
インタラクティブ性が必要な以下の部分のみ、Reactコンポーネントとしてロード（ハイドレーション）します。

- **Reader Menu**: フォント設定、背景色変更
- **Comment Form**: コメント投稿フォーム
- **Writer Dashboard**: 完全にインタラクティブなSPAライクな動作
- **Auth Status**: ログイン状態の表示

### ディレクトリ構造

```text
src/
├── components/          # UIコンポーネント
│   ├── static/          # 静的コンポーネント (.astro)
│   └── interactive/     # 動的コンポーネント (.tsx)
├── layouts/             # 共通レイアウト
├── pages/               # ファイルベースルーティング
│   ├── index.astro
│   ├── novel/[id].astro # SSR/SSG
│   └── dashboard/       # ライターダッシュボード
├── stores/              # nanostores 状態定義
└── styles/              # グローバルスタイル
```

## パフォーマンス目標

- **初期ロード JS**: < 50KB（ダッシュボードを除く）
- **LCP (Largest Contentful Paint)**: < 1.0s
- **CLS (Cumulative Layout Shift)**: 0
- **SEO**: 完全なSSR/SSGにより検索エンジン最適化
