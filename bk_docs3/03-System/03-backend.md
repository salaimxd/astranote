---
title: バックエンド設計
description: サーバーアーキテクチャ
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントはバックエンドアーキテクチャを定義するためのテンプレートです。
レイヤー構成、技術スタック、API設計などを明確にしてください。
:::

レイヤードアーキテクチャ(Presentation/Application/Domain/Infrastructure)、言語・フレームワーク・データベース・ORMの技術選定、RESTful API設計、認証・認可、エラーハンドリング、ロギング、テスト戦略を定義します。

## アーキテクチャ

### レイヤードアーキテクチャ

```
┌─ Presentation Layer ─────────────┐
│  API Endpoints (REST/GraphQL)    │
└──────────────────────────────────┘
              ↓
┌─ Application Layer ──────────────┐
│  Business Logic / Use Cases      │
└──────────────────────────────────┘
              ↓
┌─ Domain Layer ───────────────────┐
│  Domain Models / Entities        │
└──────────────────────────────────┘
              ↓
┌─ Infrastructure Layer ───────────┐
│  Database / External APIs        │
└──────────────────────────────────┘
```

## 技術選定

### 言語・フレームワーク

- **選定**: Node.js / Python / Go / Java
- **フレームワーク**: Express / FastAPI / Gin / Spring Boot
- **理由**: （選定理由）

### データベース

- **選定**: PostgreSQL / MySQL / MongoDB
- **理由**: （選定理由）

### ORM

- **選定**: Prisma / TypeORM / SQLAlchemy / GORM
- **理由**: （選定理由）

## API設計

### RESTful API

- **ベースURL**: `https://api.example.com/v1`
- **認証**: Bearer Token（JWT）
- **レート制限**: 1000 req/hour per user

### エンドポイント命名規則

- **リソース**: 複数形（`/users`, `/posts`）
- **HTTPメソッド**:
  - GET: 取得
  - POST: 作成
  - PUT/PATCH: 更新
  - DELETE: 削除

## データベース設計

### スキーマ設計原則

- 正規化の適用
- インデックスの適切な配置
- 外部キー制約の設定

### マイグレーション

- **ツール**: Prisma Migrate / Alembic / golang-migrate
- **命名規則**: `YYYYMMDDHHMMSS_description`
- **ロールバック**: 各マイグレーションにダウンマイグレーションを用意

## 認証・認可

### 認証方式

- **JWT**: ステートレスな認証
- **有効期限**: アクセストークン 15分、リフレッシュトークン 7日
- **トークン保存**: HTTPOnly Cookie / LocalStorage

### 認可

- **RBAC**: Role-Based Access Control
- **ロール**: Admin, User, Guest
- **権限チェック**: ミドルウェアで実装

## エラーハンドリング

### エラーレスポンス形式

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "入力値が不正です",
    "details": [
      {
        "field": "email",
        "message": "メールアドレスの形式が正しくありません"
      }
    ]
  }
}
```

### HTTPステータスコード

| コード | 用途 |
|--------|------|
| 200 | 成功 |
| 201 | 作成成功 |
| 400 | バリデーションエラー |
| 401 | 認証エラー |
| 403 | 認可エラー |
| 404 | リソースが見つからない |
| 500 | サーバーエラー |

## ロギング

### ログレベル

- **ERROR**: エラー発生時
- **WARN**: 警告レベルの事象
- **INFO**: 通常の動作ログ
- **DEBUG**: デバッグ情報

### ログ出力先

- 開発環境: コンソール
- 本番環境: CloudWatch Logs / Stackdriver

## テスト戦略

### ユニットテスト

- ビジネスロジックのテスト
- カバレッジ目標: 80%以上

### 統合テスト

- APIエンドポイントのテスト
- データベースとの連携テスト

### E2Eテスト

- 主要なユーザーフローのテスト
