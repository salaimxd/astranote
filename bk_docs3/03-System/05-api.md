---
title: API仕様
description: REST / GraphQL / WebSocket API
sidebar:
  badge: Template
---

:::danger[このドキュメントについて]
このドキュメントはAPI仕様を定義するためのテンプレートです。
エンドポイント、リクエスト・レスポンス形式、エラーハンドリングを明確にしてください。
:::

ベースURL、認証方式、REST APIエンドポイント(リクエスト・レスポンス形式)、WebSocket API、エラーレスポンス、レート制限、ページネーションを定義します。フロントエンドとバックエンドのインターフェース仕様として活用します。

## API概要

### ベースURL

- **開発環境**: `http://localhost:3000/api/v1`
- **ステージング環境**: `https://staging-api.example.com/v1`
- **本番環境**: `https://api.example.com/v1`

### 認証

すべてのAPIリクエストには認証トークンが必要です。

```http
Authorization: Bearer {access_token}
```

---

## REST API

### ユーザー管理

#### ユーザー情報取得

```http
GET /users/{userId}
```

**レスポンス**:
```json
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "ユーザー名",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

#### ユーザー作成

```http
POST /users
```

**リクエストボディ**:
```json
{
  "email": "user@example.com",
  "password": "secure_password",
  "name": "ユーザー名"
}
```

**レスポンス**:
```json
{
  "id": "user_123",
  "email": "user@example.com",
  "name": "ユーザー名",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

#### ユーザー更新

```http
PATCH /users/{userId}
```

**リクエストボディ**:
```json
{
  "name": "新しい名前"
}
```

#### ユーザー削除

```http
DELETE /users/{userId}
```

**レスポンス**:
```json
{
  "message": "ユーザーを削除しました"
}
```

---

## WebSocket API

### 接続

```javascript
const ws = new WebSocket('wss://api.example.com/ws?token={access_token}');
```

### イベント

#### メッセージ送信

```json
{
  "type": "message.send",
  "payload": {
    "content": "メッセージ内容"
  }
}
```

#### メッセージ受信

```json
{
  "type": "message.received",
  "payload": {
    "id": "msg_123",
    "content": "メッセージ内容",
    "senderId": "user_123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

---

## エラーレスポンス

### 形式

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "エラーメッセージ",
    "details": []
  }
}
```

### エラーコード一覧

| コード | 説明 | HTTPステータス |
|--------|------|---------------|
| `VALIDATION_ERROR` | バリデーションエラー | 400 |
| `UNAUTHORIZED` | 認証エラー | 401 |
| `FORBIDDEN` | 権限エラー | 403 |
| `NOT_FOUND` | リソースが見つからない | 404 |
| `INTERNAL_ERROR` | サーバーエラー | 500 |

---

## レート制限

- **制限**: 1000リクエスト/時間/ユーザー
- **ヘッダー**:
  ```http
  X-RateLimit-Limit: 1000
  X-RateLimit-Remaining: 999
  X-RateLimit-Reset: 1704067200
  ```

---

## ページネーション

### クエリパラメータ

```http
GET /items?page=1&limit=20
```

### レスポンス形式

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```
