---
title: 用語集
description: Star Novel で使用する用語の定義（SSOT）
---

:::caution[SSOT]
このドキュメントは Star Novel で使用する用語の **Single Source of Truth（唯一の信頼できる情報源）** です。
他のドキュメントと定義が異なる場合は、このドキュメントを正としてください。
:::

---

## プロダクト用語

### コア概念

| 用語 | 定義 |
|------|------|
| **Star Novel** | 多言語AI翻訳を特徴とするグローバルライトノベルプラットフォーム |
| **Write Once, Read Anywhere** | 母国語で書くだけで世界中の読者に届くというコンセプト |
| **作家（Writer）** | Star Novel で作品を投稿するユーザー |
| **読者（Reader）** | Star Novel で作品を閲覧するユーザー |
| **作品（Novel）** | 作家が投稿する小説コンテンツ |
| **章（Chapter）** | 作品を構成する各話 |

### 翻訳機能

| 用語 | 定義 |
|------|------|
| **自動翻訳** | AI による自動的な多言語翻訳機能 |
| **用語集（Glossary）** | 作家が登録する固有名詞・造語の翻訳ルール集 |
| **原文言語** | 作家が執筆に使用する言語（主に日本語） |
| **翻訳言語** | 読者が閲覧する翻訳先の言語 |

### 収益化

| 用語 | 定義 |
|------|------|
| **プレミアム（作家）** | 追加言語翻訳、詳細統計などが使える有料プラン |
| **プレミアム（読者）** | 広告非表示、オフライン閲覧などが使える有料プラン |
| **投げ銭（Tip）** | 読者から作家への直接的な金銭支援 |
| **先読み** | 有料で最新話を先に読める機能 |

---

## ユーザーロール

| ロール | 説明 | 権限 |
|--------|------|------|
| **Guest** | 未登録ユーザー | 閲覧のみ |
| **User** | 登録済みユーザー | 閲覧、ブックマーク、コメント |
| **Writer** | 作家権限を持つユーザー | User権限 + 作品投稿 |
| **Premium** | 有料会員 | 広告非表示、追加機能 |
| **Admin** | 管理者 | 全権限 |

---

## 技術用語

### フロントエンド

| 用語 | 定義 |
|------|------|
| **WYSIWYG** | What You See Is What You Get。見たままを編集できるエディタ形式 |
| **Tiptap** | WYSIWYGエディタのフレームワーク |
| **ルビ** | 漢字の上に表示する読み仮名（`<ruby>漢字<rt>かんじ</rt></ruby>`） |

### バックエンド

| 用語 | 定義 |
|------|------|
| **Cloudflare Pages** | フロントエンドのホスティングサービス |
| **Cloudflare D1** | SQLite互換のサーバーレスデータベース |
| **Cloudflare R2** | S3互換のオブジェクトストレージ |
| **Cloudflare KV** | キーバリューストア（キャッシュ、セッション管理） |
| **Drizzle ORM** | TypeScript向けのORMライブラリ |
| **Lucia Auth** | セッションベースの認証ライブラリ |

### API

| 用語 | 定義 |
|------|------|
| **REST API** | Representational State Transfer。HTTPベースのAPI設計スタイル |
| **JWT** | JSON Web Token。認証トークン形式 |
| **Rate Limiting** | API呼び出し回数の制限 |

---

## ビジネス用語

### 指標

| 用語 | 定義 |
|------|------|
| **MAU** | Monthly Active Users。月間アクティブユーザー数 |
| **DAU** | Daily Active Users。日間アクティブユーザー数 |
| **MRR** | Monthly Recurring Revenue。月間経常収益 |
| **NPS** | Net Promoter Score。顧客推奨度（-100〜100） |
| **ARPU** | Average Revenue Per User。ユーザーあたり平均収益 |
| **CVR** | Conversion Rate。転換率（無料→有料など） |
| **LTV** | Lifetime Value。顧客生涯価値 |
| **CAC** | Customer Acquisition Cost。顧客獲得コスト |

### フェーズ

| 用語 | 定義 |
|------|------|
| **PMF** | Product-Market Fit。製品が市場に受け入れられた状態 |
| **MVP** | Minimum Viable Product。実用最小限の製品 |
| **クローズドβ** | 招待制の限定公開テスト |
| **オープンβ** | 誰でも参加できる公開テスト |
| **GA** | General Availability。正式公開 |

---

## 開発用語

### プロセス

| 用語 | 定義 |
|------|------|
| **SSOT** | Single Source of Truth。唯一の信頼できる情報源 |
| **Docs-as-Code** | ドキュメントをコードと同様に管理する手法 |
| **仕様駆動開発** | 仕様書を先に作成し、それに基づいて実装する開発手法 |

### バージョン管理

| 用語 | 定義 |
|------|------|
| **main** | 安定版ブランチ（本番デプロイ） |
| **develop** | 開発統合ブランチ |
| **feature/#N-*** | 機能開発ブランチ（Issue番号付き） |
| **fix/#N-*** | バグ修正ブランチ（Issue番号付き） |

---

## 略語一覧

| 略語 | 正式名称 | 説明 |
|------|---------|------|
| AI | Artificial Intelligence | 人工知能 |
| API | Application Programming Interface | アプリケーション間のインターフェース |
| ARPU | Average Revenue Per User | ユーザーあたり平均収益 |
| CAC | Customer Acquisition Cost | 顧客獲得コスト |
| CVR | Conversion Rate | 転換率 |
| DAU | Daily Active Users | 日間アクティブユーザー数 |
| GA | General Availability | 正式公開 |
| GTM | Go-to-Market | 市場投入戦略 |
| JWT | JSON Web Token | 認証トークン形式 |
| KPI | Key Performance Indicator | 重要業績評価指標 |
| LTV | Lifetime Value | 顧客生涯価値 |
| MAU | Monthly Active Users | 月間アクティブユーザー数 |
| MRR | Monthly Recurring Revenue | 月間経常収益 |
| MVP | Minimum Viable Product | 実用最小限の製品 |
| NPS | Net Promoter Score | 顧客推奨度 |
| ORM | Object-Relational Mapping | オブジェクト関係マッピング |
| PMF | Product-Market Fit | 製品と市場の適合 |
| PMM | Product Marketing Manager | プロダクトマーケティングマネージャー |
| REST | Representational State Transfer | HTTPベースのAPI設計スタイル |
| SEO | Search Engine Optimization | 検索エンジン最適化 |
| SSOT | Single Source of Truth | 唯一の信頼できる情報源 |
| UI | User Interface | ユーザーインターフェース |
| UX | User Experience | ユーザー体験 |
| WYSIWYG | What You See Is What You Get | 見たまま編集 |
