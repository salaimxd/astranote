---
title: 用語集
description: プロジェクト共通用語の定義
---

プロジェクトで使用する用語の定義（SSOT: Single Source of Truth）です。

## コア概念

| 用語 | 定義 |
|------|------|
| **Astranote** | 多言語AI翻訳を特徴とするグローバルライトノベルプラットフォーム |
| **Write Once, Read Anywhere** | 母国語で書くだけで世界中の読者に届くというコンセプト |
| **作家（Writer）** | Astranote で作品を投稿するユーザー |
| **読者（Reader）** | Astranote で作品を閲覧するユーザー |
| **作品（Novel）** | 作家が投稿する小説コンテンツ |
| **章（Chapter）** | 作品を構成する各話 |

## 翻訳機能

| 用語 | 定義 |
|------|------|
| **自動翻訳** | AI による自動的な多言語翻訳機能 |
| **用語集（Glossary）** | 作家が登録する固有名詞・造語の翻訳ルール集 |
| **原文言語** | 作家が執筆に使用する言語（主に日本語） |
| **翻訳言語** | 読者が閲覧する翻訳先の言語 |

## ユーザーロール

| ロール | 説明 | 権限 |
|--------|------|------|
| **Guest** | 未登録ユーザー | 閲覧のみ |
| **User** | 登録済みユーザー | 閲覧、ブックマーク、コメント |
| **Writer** | 作家権限を持つユーザー | User権限 + 作品投稿 |
| **Premium** | 有料会員 | 広告非表示、追加機能 |
| **Admin** | 管理者 | 全権限 |

## 技術用語

| 用語 | 定義 |
|------|------|
| **WYSIWYG** | What You See Is What You Get。見たままを編集できるエディタ形式 |
| **Tiptap** | WYSIWYGエディタのフレームワーク |
| **Cloudflare Pages** | フロントエンドのホスティングサービス |
| **Cloudflare D1** | SQLite互換のサーバーレスデータベース |
| **Cloudflare R2** | S3互換のオブジェクトストレージ |
| **Cloudflare KV** | キーバリューストア（キャッシュ、セッション管理） |
| **Lucia Auth** | セッションベースの認証ライブラリ |

## ビジネス用語・指標

| 用語 | 定義 |
|------|------|
| **MAU** | Monthly Active Users。月間アクティブユーザー数 |
| **MRR** | Monthly Recurring Revenue。月間経常収益 |
| **PMF** | Product-Market Fit。製品が市場に受け入れられた状態 |
| **MVP** | Minimum Viable Product。実用最小限の製品 |
| **LTV** | Lifetime Value。顧客生涯価値 |
| **CAC** | Customer Acquisition Cost。顧客獲得コスト |

### API

**定義**: Application Programming Interface。アプリケーション間でデータをやり取りするためのインターフェース。

---

### JWT

**定義**: JSON Web Token。認証情報を安全に伝送するためのトークン形式。

**使用例**: 「JWTトークンをヘッダーに付与してAPIリクエストを送信する」

---

## ビジネス用語

### KPI

**定義**: Key Performance Indicator（重要業績評価指標）。ビジネスの成功を測定するための指標。

**使用例**: 「DAUをKPIとして設定する」

---

### MVA

**定義**: Minimum Viable Product（最小実用製品）。最小限の機能で市場投入できる製品。

**使用例**: 「Phase 1ではMVPをリリースする」

---

## 略語一覧

| 略語 | 正式名称 | 説明 |
|------|---------|------|
| API | Application Programming Interface | プログラム間のインターフェース |
| UI | User Interface | ユーザーインターフェース |
| UX | User Experience | ユーザー体験 |
| MVP | Minimum Viable Product | 最小実用製品 |
| KPI | Key Performance Indicator | 重要業績評価指標 |
| DAU | Daily Active Users | 日次アクティブユーザー数 |
| MAU | Monthly Active Users | 月次アクティブユーザー数 |
