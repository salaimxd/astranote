---
title: インフラ構成
description: Cloudflare エコシステムによるインフラ設計
---

Astranote は **Cloudflare エコシステムに完全に統一** することで、運用コストを削減し、グローバル規模のスケーラビリティを確保します。

## 構成要素

| サービス | 用途 | プラン (Phase 1) |
| -------- | ---- | ---------------- |
| **Cloudflare Pages** | フロントエンドのホスティング、ビルド、デプロイ | Free |
| **Cloudflare Workers** | APIサーバー、エッジコンピューティング | Free (100k req/day) |
| **Cloudflare D1** | リレーショナルデータベース (SQLite) | Free (5GB) |
| **Cloudflare KV** | 高速なキーバリューストア（キャッシュ、セッション） | Free |
| **Cloudflare R2** | オブジェクトストレージ（画像、アセット） | Free (10GB) |
| **Cloudflare DNS** | DNS管理、DDoS保護、SSL | Free |

## コストメリット

- **Egress (データ転送) 無料**: R2からの画像配信コストがゼロ。
- **Workers 無料枠**: 1日10万リクエストまで無料で、初期フェーズのトラフィックをカバー。
- **D1/KV 無料枠**: データベース周りの固定費が不要。

## デプロイパイプライン

Github Actions または Cloudflare Pages の自動デプロイを使用します。

1. **Frontend**: `main` ブランチへのプッシュで Cloudflare Pages が自動ビルド・デプロイ。
2. **Backend**: `wrangler deploy` コマンドで Workers へデプロイ。D1のマイグレーションも同時に実行。

## モニタリング

- **Cloudflare Analytics**: Webサイトのトラフィック解析。
- **Workers Analytics**: APIのリクエスト数、CPU時間、エラレート。
- **Sentry**: アプリケーションのエラー追跡（フロントエンド/バックエンド）。
