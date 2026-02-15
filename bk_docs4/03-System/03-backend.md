---
title: バックエンド設計
description: Cloudflare Workers + Hono によるAPI設計
---

Astranoteのバックエンドは、**Cloudflare Workers** 上で動作する **Hono** フレームワークを用いて構築します。サーバーレスで、エッジでの低遅延処理を実現します。

## 技術スタック (Backend)

- **Runtime**: Cloudflare Workers
- **Framework**: Hono (Ultra-fast, standard Web Standards)
- **ORM**: Drizzle ORM
- **Database**: Cloudflare D1 (SQLite)
- **Validation**: Zod
- **Auth**: Lucia Auth

## サービス構成

### API サーバー (Workers)

Hono を使用し、モノリスとしてデプロイしますが、内部は機能ごとのモジュール構成とします。

```text
src/
├── routes/
│   ├── auth.ts          # 認証関連
│   ├── novels.ts        # 小説、章CRUD
│   ├── users.ts         # プロフィール
│   └── translate.ts     # 翻訳キュー処理
├── db/
│   └── schema.ts        # Drizzle スキーマ
├── services/
│   └── translation.ts   # AI翻訳ロジック
└── middleware/
    └── auth.ts          # JWT検証
```

## 翻訳処理フロー

1. 作家が日本語で章を投稿 (POST /novels/:id/chapters)
2. APIが章をDBに保存 (`status: processing`)
3. 非同期でAI翻訳サービス (Gemini/Claude) を呼び出し
4. 翻訳結果を保存し、ステータスを更新 (`status: published`)

## エラーハンドリング

統一されたエラーレスポンス形式（RFC 7807 Problem Details for HTTP APIs 準拠）を使用します。

```json
{
  "type": "about:blank",
  "title": "Validation Error",
  "status": 400,
  "detail": "Title is required",
  "instance": "/v1/novels"
}
```
