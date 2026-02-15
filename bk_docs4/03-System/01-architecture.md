---
title: システムアーキテクチャ
description: 技術アーキテクチャ設計（コスト最適化版）
---

Astranoteのシステムアーキテクチャの全体像を定義します。**維持費を最小限に抑えながら**、スケーラビリティと開発速度を両立する設計です。

## 設計方針

- **コストファースト**: 無料枠・低コストサービスを最大活用
- **Cloudflare統一**: 一つのプラットフォームで完結させコスト削減
- **段階的スケール**: 必要になってから有料プランへ移行

## アーキテクチャ構成図 (Cloudflare統一)

```mermaid
graph TD
    User[ユーザー (Web/PWA)] --> CF[Cloudflare CDN]
    CF --> Pages[Cloudflare Pages (Frontend)]
    CF --> Workers[Cloudflare Workers (Backend API)]
    
    subgraph Backend Services
        Workers --> D1[(Cloudflare D1 - DB)]
        Workers --> KV[(Cloudflare KV - Cache)]
        Workers --> R2[(Cloudflare R2 - Storage)]
        Workers --> AI[Translation Service]
    end
    
    AI --> Gemini[Gemini 2.0 Flash (激安)]
    AI --> Claude[Claude 3.5 Haiku (高品質)]
```

## 技術スタック

### フロントエンド
- **Framework**: Astro (SSG/SSR)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI Lib**: shadcn/ui
- **State**: nanostores

### バックエンド
- **Runtime**: Cloudflare Workers
- **Framework**: Hono
- **ORM**: Drizzle ORM
- **Auth**: Lucia Auth
- **Validation**: Zod

### データストア
- **DB**: Cloudflare D1 (SQLite) - 無料5GB/日5M Read
- **Cache**: Cloudflare KV
- **Storage**: Cloudflare R2 (Egress無料)

## コスト最適化戦略

### 翻訳機能
- **Gemini 2.0 Flash**: メインの翻訳エンジンとして使用（$0.10/1M tokens）。
- **DeepL Free**: 補助的に利用（月50万文字まで無料）。
- **Claude 3.5 Sonnet**: プレミアムユーザー向け（高コスト高画質）。

### インフラコスト
- **Phase 1 (MVP)**: すべて無料枠内で収める（月額 ~$4 ※ドメイン代等のみ）。
- **Phase 2 (Growth)**: Cloudflare Workers Paid ($5) + Gemini従量課金。
- **検索**: Meilisearch ($30/mo) の代わりに **D1 FTS5** ($0) を利用。

## キャッシュ戦略

1. **Static Assets**: Cloudflare CDN (1年)
2. **Translation**: Cloudflare KV / D1 (永続)
3. **Session**: Cloudflare KV (24時間)
4. **Browser**: Service Workerによるオフラインキャッシュ
