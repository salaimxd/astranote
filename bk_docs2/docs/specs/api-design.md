---
title: API設計書
description: Star Novel の RESTful API 設計仕様
---

## 概要

Star Novel の API は RESTful 設計原則に従い、Cloudflare Workers + Hono フレームワークで実装します。

### 設計原則

| 原則 | 説明 |
|------|------|
| RESTful | リソース指向の URL 設計 |
| JSON | リクエスト/レスポンスは JSON 形式 |
| バージョニング | URL パスに `/v1/` を含める |
| 認証 | Bearer トークン（JWT）による認証 |
| エラーハンドリング | RFC 7807 準拠の Problem Details |

### ベース URL

```
Production: https://api.star-novel.com/v1
Staging:    https://api-staging.star-novel.com/v1
```

---

## 認証

### 認証フロー

```
┌─────────────────────────────────────────────────────────────┐
│                      認証フロー                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  【メール/パスワード認証】                                   │
│  POST /v1/auth/register    → アカウント作成                 │
│  POST /v1/auth/login       → ログイン（トークン取得）       │
│  POST /v1/auth/refresh     → トークンリフレッシュ           │
│  POST /v1/auth/logout      → ログアウト                     │
│                                                             │
│  【OAuth認証】                                               │
│  GET  /v1/auth/oauth/:provider          → OAuth開始         │
│  GET  /v1/auth/oauth/:provider/callback → コールバック      │
│                                                             │
│  【パスワードリセット】                                      │
│  POST /v1/auth/forgot-password  → リセットメール送信        │
│  POST /v1/auth/reset-password   → パスワード変更            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 認証ヘッダー

```http
Authorization: Bearer <access_token>
```

### トークン仕様

| 種類 | 有効期限 | 用途 |
|------|---------|------|
| Access Token | 15分 | API アクセス |
| Refresh Token | 7日 | Access Token の更新 |

---

## エンドポイント一覧

### 認証 API

#### POST /v1/auth/register

新規ユーザー登録

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "username": "novelist123",
  "locale": "ja"
}
```

**Response: 201 Created**
```json
{
  "user": {
    "id": "usr_abc123",
    "email": "user@example.com",
    "username": "novelist123",
    "role": "reader",
    "createdAt": "2025-01-15T10:00:00Z"
  },
  "tokens": {
    "accessToken": "eyJ...",
    "refreshToken": "eyJ...",
    "expiresIn": 900
  }
}
```

#### POST /v1/auth/login

ログイン

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response: 200 OK**
```json
{
  "user": {
    "id": "usr_abc123",
    "email": "user@example.com",
    "username": "novelist123",
    "role": "writer"
  },
  "tokens": {
    "accessToken": "eyJ...",
    "refreshToken": "eyJ...",
    "expiresIn": 900
  }
}
```

---

### ユーザー API

#### GET /v1/users/me

現在のユーザー情報取得

**Response: 200 OK**
```json
{
  "id": "usr_abc123",
  "email": "user@example.com",
  "username": "novelist123",
  "displayName": "小説家太郎",
  "avatar": "https://cdn.star-novel.com/avatars/usr_abc123.webp",
  "bio": "異世界ファンタジーを書いています",
  "role": "writer",
  "locale": "ja",
  "stats": {
    "novelsCount": 5,
    "followersCount": 1234,
    "followingCount": 56
  },
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### PATCH /v1/users/me

ユーザー情報更新

**Request:**
```json
{
  "displayName": "小説家次郎",
  "bio": "ラブコメも始めました",
  "locale": "en"
}
```

**Response: 200 OK**
```json
{
  "id": "usr_abc123",
  "displayName": "小説家次郎",
  "bio": "ラブコメも始めました",
  "locale": "en",
  "updatedAt": "2025-01-15T12:00:00Z"
}
```

#### GET /v1/users/:userId

ユーザー公開情報取得

**Response: 200 OK**
```json
{
  "id": "usr_abc123",
  "username": "novelist123",
  "displayName": "小説家太郎",
  "avatar": "https://cdn.star-novel.com/avatars/usr_abc123.webp",
  "bio": "異世界ファンタジーを書いています",
  "role": "writer",
  "stats": {
    "novelsCount": 5,
    "followersCount": 1234
  }
}
```

---

### 小説 API

#### GET /v1/novels

小説一覧取得

**Query Parameters:**

| パラメータ | 型 | 説明 | デフォルト |
|-----------|-----|------|-----------|
| page | number | ページ番号 | 1 |
| limit | number | 取得件数（最大100） | 20 |
| sort | string | ソート順 | `updatedAt` |
| order | string | 昇順/降順 | `desc` |
| genre | string | ジャンルフィルタ | - |
| status | string | 連載状態フィルタ | - |
| language | string | 言語フィルタ | - |

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "nvl_xyz789",
      "title": "異世界転生したら最強だった件",
      "synopsis": "交通事故で死んだ主人公が...",
      "cover": "https://cdn.star-novel.com/covers/nvl_xyz789.webp",
      "author": {
        "id": "usr_abc123",
        "username": "novelist123",
        "displayName": "小説家太郎"
      },
      "genre": "fantasy",
      "tags": ["異世界", "転生", "チート"],
      "status": "ongoing",
      "originalLanguage": "ja",
      "availableLanguages": ["ja", "en"],
      "stats": {
        "chaptersCount": 150,
        "totalWords": 450000,
        "viewsCount": 1234567,
        "bookmarksCount": 5678,
        "rating": 4.5,
        "ratingsCount": 890
      },
      "createdAt": "2024-06-01T00:00:00Z",
      "updatedAt": "2025-01-14T18:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1234,
    "totalPages": 62
  }
}
```

#### POST /v1/novels

小説作成（作家のみ）

**Request:**
```json
{
  "title": "新しい物語",
  "synopsis": "これは新しい冒険の始まり...",
  "genre": "fantasy",
  "tags": ["冒険", "ファンタジー"],
  "isR18": false,
  "autoTranslate": true,
  "targetLanguages": ["en"]
}
```

**Response: 201 Created**
```json
{
  "id": "nvl_new123",
  "title": "新しい物語",
  "synopsis": "これは新しい冒険の始まり...",
  "genre": "fantasy",
  "status": "draft",
  "originalLanguage": "ja",
  "autoTranslate": true,
  "targetLanguages": ["en"],
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### GET /v1/novels/:novelId

小説詳細取得

**Query Parameters:**

| パラメータ | 型 | 説明 |
|-----------|-----|------|
| language | string | 取得する言語（翻訳版） |

**Response: 200 OK**
```json
{
  "id": "nvl_xyz789",
  "title": "異世界転生したら最強だった件",
  "synopsis": "交通事故で死んだ主人公が異世界に転生し...",
  "cover": "https://cdn.star-novel.com/covers/nvl_xyz789.webp",
  "author": {
    "id": "usr_abc123",
    "username": "novelist123",
    "displayName": "小説家太郎",
    "avatar": "https://cdn.star-novel.com/avatars/usr_abc123.webp"
  },
  "genre": "fantasy",
  "tags": ["異世界", "転生", "チート", "ハーレム"],
  "status": "ongoing",
  "isR18": false,
  "originalLanguage": "ja",
  "currentLanguage": "ja",
  "availableLanguages": ["ja", "en"],
  "autoTranslate": true,
  "stats": {
    "chaptersCount": 150,
    "totalWords": 450000,
    "viewsCount": 1234567,
    "bookmarksCount": 5678,
    "rating": 4.5,
    "ratingsCount": 890
  },
  "chapters": [
    {
      "id": "chp_001",
      "number": 1,
      "title": "第1章：転生",
      "publishedAt": "2024-06-01T00:00:00Z"
    }
  ],
  "createdAt": "2024-06-01T00:00:00Z",
  "updatedAt": "2025-01-14T18:00:00Z"
}
```

#### PATCH /v1/novels/:novelId

小説更新（作者のみ）

**Request:**
```json
{
  "title": "更新されたタイトル",
  "synopsis": "更新された概要",
  "status": "completed"
}
```

#### DELETE /v1/novels/:novelId

小説削除（作者のみ）

**Response: 204 No Content**

---

### 章 API

#### GET /v1/novels/:novelId/chapters

章一覧取得

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "chp_001",
      "number": 1,
      "title": "第1章：転生",
      "wordCount": 3500,
      "status": "published",
      "publishedAt": "2024-06-01T00:00:00Z"
    },
    {
      "id": "chp_002",
      "number": 2,
      "title": "第2章：新しい力",
      "wordCount": 4200,
      "status": "published",
      "publishedAt": "2024-06-03T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 150,
    "totalPages": 3
  }
}
```

#### POST /v1/novels/:novelId/chapters

章作成（作者のみ）

**Request:**
```json
{
  "title": "第151章：決戦",
  "content": "本文のコンテンツ...",
  "authorNote": "作者コメント",
  "status": "draft"
}
```

**Response: 201 Created**
```json
{
  "id": "chp_151",
  "number": 151,
  "title": "第151章：決戦",
  "wordCount": 5000,
  "status": "draft",
  "translationStatus": {
    "en": "pending"
  },
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### GET /v1/novels/:novelId/chapters/:chapterId

章本文取得

**Query Parameters:**

| パラメータ | 型 | 説明 |
|-----------|-----|------|
| language | string | 取得する言語 |

**Response: 200 OK**
```json
{
  "id": "chp_001",
  "novelId": "nvl_xyz789",
  "number": 1,
  "title": "第1章：転生",
  "content": "本文のコンテンツ...",
  "authorNote": "初投稿です。よろしくお願いします。",
  "wordCount": 3500,
  "status": "published",
  "originalLanguage": "ja",
  "currentLanguage": "ja",
  "availableLanguages": ["ja", "en"],
  "translationStatus": {
    "en": "completed"
  },
  "navigation": {
    "previous": null,
    "next": {
      "id": "chp_002",
      "number": 2,
      "title": "第2章：新しい力"
    }
  },
  "publishedAt": "2024-06-01T00:00:00Z",
  "updatedAt": "2024-06-01T12:00:00Z"
}
```

#### PATCH /v1/novels/:novelId/chapters/:chapterId

章更新（作者のみ）

**Request:**
```json
{
  "title": "第1章：運命の転生",
  "content": "更新された本文...",
  "status": "published"
}
```

#### DELETE /v1/novels/:novelId/chapters/:chapterId

章削除（作者のみ）

**Response: 204 No Content**

---

### 翻訳 API

#### POST /v1/novels/:novelId/chapters/:chapterId/translate

手動翻訳リクエスト（作者のみ）

**Request:**
```json
{
  "targetLanguage": "en",
  "priority": "normal"
}
```

**Response: 202 Accepted**
```json
{
  "jobId": "job_tr_abc123",
  "status": "queued",
  "targetLanguage": "en",
  "estimatedCompletionTime": "2025-01-15T10:05:00Z"
}
```

#### GET /v1/translations/:jobId

翻訳ジョブ状態確認

**Response: 200 OK**
```json
{
  "jobId": "job_tr_abc123",
  "status": "completed",
  "chapterId": "chp_001",
  "sourceLanguage": "ja",
  "targetLanguage": "en",
  "wordCount": 3500,
  "cost": 0.035,
  "startedAt": "2025-01-15T10:00:30Z",
  "completedAt": "2025-01-15T10:02:15Z"
}
```

---

### 用語集 API

#### GET /v1/novels/:novelId/glossary

用語集取得

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "gloss_001",
      "term": "魔王城",
      "reading": "まおうじょう",
      "translations": {
        "en": "Demon Lord's Castle",
        "ko": "마왕성",
        "zh-TW": "魔王城"
      },
      "description": "物語の舞台となる城",
      "category": "location",
      "createdAt": "2024-06-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 100,
    "total": 45
  }
}
```

#### POST /v1/novels/:novelId/glossary

用語追加（作者のみ）

**Request:**
```json
{
  "term": "聖剣エクスカリバー",
  "reading": "せいけんエクスカリバー",
  "translations": {
    "en": "Holy Sword Excalibur"
  },
  "description": "主人公が手にする伝説の剣",
  "category": "item"
}
```

**Response: 201 Created**
```json
{
  "id": "gloss_046",
  "term": "聖剣エクスカリバー",
  "reading": "せいけんエクスカリバー",
  "translations": {
    "en": "Holy Sword Excalibur"
  },
  "description": "主人公が手にする伝説の剣",
  "category": "item",
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### PATCH /v1/novels/:novelId/glossary/:termId

用語更新（作者のみ）

#### DELETE /v1/novels/:novelId/glossary/:termId

用語削除（作者のみ）

---

### ブックマーク API

#### GET /v1/users/me/bookmarks

ブックマーク一覧取得

**Response: 200 OK**
```json
{
  "data": [
    {
      "id": "bkm_001",
      "novel": {
        "id": "nvl_xyz789",
        "title": "異世界転生したら最強だった件",
        "cover": "https://cdn.star-novel.com/covers/nvl_xyz789.webp",
        "author": {
          "id": "usr_abc123",
          "displayName": "小説家太郎"
        }
      },
      "lastReadChapter": {
        "id": "chp_050",
        "number": 50,
        "title": "第50章：中間決戦"
      },
      "latestChapter": {
        "id": "chp_150",
        "number": 150
      },
      "unreadCount": 100,
      "createdAt": "2024-08-01T00:00:00Z",
      "updatedAt": "2025-01-10T12:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 15
  }
}
```

#### POST /v1/novels/:novelId/bookmark

ブックマーク追加

**Response: 201 Created**
```json
{
  "id": "bkm_002",
  "novelId": "nvl_xyz789",
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### DELETE /v1/novels/:novelId/bookmark

ブックマーク解除

**Response: 204 No Content**

---

### 読書進捗 API

#### POST /v1/novels/:novelId/chapters/:chapterId/progress

読書進捗保存

**Request:**
```json
{
  "progress": 0.75,
  "position": "paragraph_42"
}
```

**Response: 200 OK**
```json
{
  "chapterId": "chp_050",
  "progress": 0.75,
  "position": "paragraph_42",
  "updatedAt": "2025-01-15T10:00:00Z"
}
```

#### GET /v1/novels/:novelId/progress

小説の読書進捗取得

**Response: 200 OK**
```json
{
  "novelId": "nvl_xyz789",
  "lastReadChapter": {
    "id": "chp_050",
    "number": 50,
    "progress": 0.75,
    "position": "paragraph_42"
  },
  "completedChapters": [1, 2, 3, 4, 5, 49],
  "updatedAt": "2025-01-15T10:00:00Z"
}
```

---

### フォロー API

#### POST /v1/users/:userId/follow

ユーザーをフォロー

**Response: 201 Created**
```json
{
  "followingId": "usr_abc123",
  "createdAt": "2025-01-15T10:00:00Z"
}
```

#### DELETE /v1/users/:userId/follow

フォロー解除

**Response: 204 No Content**

#### GET /v1/users/me/following

フォロー中のユーザー一覧

#### GET /v1/users/me/followers

フォロワー一覧

---

### コメント API

#### GET /v1/novels/:novelId/comments

小説へのコメント一覧

#### POST /v1/novels/:novelId/comments

小説にコメント投稿

#### GET /v1/novels/:novelId/chapters/:chapterId/comments

章へのコメント一覧

#### POST /v1/novels/:novelId/chapters/:chapterId/comments

章にコメント投稿

**Request:**
```json
{
  "content": "とても面白かったです！続きが楽しみです。",
  "parentId": null
}
```

**Response: 201 Created**
```json
{
  "id": "cmt_001",
  "content": "とても面白かったです！続きが楽しみです。",
  "author": {
    "id": "usr_reader001",
    "displayName": "読者A",
    "avatar": "https://cdn.star-novel.com/avatars/usr_reader001.webp"
  },
  "likes": 0,
  "repliesCount": 0,
  "createdAt": "2025-01-15T10:00:00Z"
}
```

---

### 検索 API

#### GET /v1/search

統合検索

**Query Parameters:**

| パラメータ | 型 | 説明 |
|-----------|-----|------|
| q | string | 検索クエリ（必須） |
| type | string | 検索対象（novels, authors, all） |
| genre | string | ジャンルフィルタ |
| language | string | 言語フィルタ |
| sort | string | ソート順 |
| page | number | ページ番号 |
| limit | number | 取得件数 |

**Response: 200 OK**
```json
{
  "novels": {
    "data": [...],
    "total": 123
  },
  "authors": {
    "data": [...],
    "total": 5
  },
  "query": "異世界",
  "took": 45
}
```

---

### ランキング API

#### GET /v1/rankings

ランキング取得

**Query Parameters:**

| パラメータ | 型 | 説明 | 値 |
|-----------|-----|------|-----|
| type | string | ランキング種類 | daily, weekly, monthly, total |
| genre | string | ジャンル | all, fantasy, romance, etc. |
| language | string | 言語 | ja, en, etc. |

**Response: 200 OK**
```json
{
  "type": "weekly",
  "genre": "fantasy",
  "period": {
    "start": "2025-01-08T00:00:00Z",
    "end": "2025-01-14T23:59:59Z"
  },
  "data": [
    {
      "rank": 1,
      "previousRank": 2,
      "novel": {
        "id": "nvl_xyz789",
        "title": "異世界転生したら最強だった件",
        "cover": "https://cdn.star-novel.com/covers/nvl_xyz789.webp",
        "author": {
          "id": "usr_abc123",
          "displayName": "小説家太郎"
        },
        "stats": {
          "viewsCount": 50000,
          "bookmarksCount": 500
        }
      },
      "score": 12500
    }
  ]
}
```

---

## エラーハンドリング

### エラーレスポンス形式

RFC 7807 Problem Details 準拠

```json
{
  "type": "https://api.star-novel.com/errors/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "リクエストの検証に失敗しました",
  "instance": "/v1/novels",
  "errors": [
    {
      "field": "title",
      "message": "タイトルは必須です"
    },
    {
      "field": "synopsis",
      "message": "概要は10文字以上必要です"
    }
  ],
  "traceId": "trace_abc123"
}
```

### HTTPステータスコード

| コード | 意味 | 使用場面 |
|--------|------|---------|
| 200 | OK | 成功（取得・更新） |
| 201 | Created | リソース作成成功 |
| 202 | Accepted | 非同期処理受付 |
| 204 | No Content | 削除成功 |
| 400 | Bad Request | リクエスト不正 |
| 401 | Unauthorized | 認証エラー |
| 403 | Forbidden | 権限エラー |
| 404 | Not Found | リソース未存在 |
| 409 | Conflict | 競合エラー |
| 422 | Unprocessable Entity | 検証エラー |
| 429 | Too Many Requests | レート制限 |
| 500 | Internal Server Error | サーバーエラー |

### エラーコード一覧

| コード | 説明 |
|--------|------|
| AUTH_INVALID_CREDENTIALS | 認証情報が無効 |
| AUTH_TOKEN_EXPIRED | トークン期限切れ |
| AUTH_INSUFFICIENT_PERMISSIONS | 権限不足 |
| USER_NOT_FOUND | ユーザーが見つからない |
| USER_ALREADY_EXISTS | ユーザーが既に存在 |
| NOVEL_NOT_FOUND | 小説が見つからない |
| CHAPTER_NOT_FOUND | 章が見つからない |
| TRANSLATION_FAILED | 翻訳処理失敗 |
| RATE_LIMIT_EXCEEDED | レート制限超過 |
| VALIDATION_ERROR | 入力検証エラー |

---

## レート制限

### 制限値

| エンドポイント | 制限 | 単位 |
|---------------|------|------|
| 認証 API | 10 | 分 |
| 読み取り API | 1000 | 時間 |
| 書き込み API | 100 | 時間 |
| 翻訳 API | 50 | 時間 |
| 検索 API | 100 | 分 |

### レート制限ヘッダー

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1705312800
```

### 制限超過レスポンス

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 3600
```

```json
{
  "type": "https://api.star-novel.com/errors/rate-limit-exceeded",
  "title": "Rate Limit Exceeded",
  "status": 429,
  "detail": "リクエスト制限を超過しました。1時間後に再試行してください。",
  "retryAfter": 3600
}
```

---

## ページネーション

### リクエストパラメータ

| パラメータ | 型 | デフォルト | 最大値 |
|-----------|-----|-----------|--------|
| page | number | 1 | - |
| limit | number | 20 | 100 |

### レスポンス形式

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 1234,
    "totalPages": 62,
    "hasNext": true,
    "hasPrevious": false
  }
}
```

---

## 関連ドキュメント

| ドキュメント | 内容 |
|-------------|------|
| [アーキテクチャ](/specs/architecture/) | システム全体設計 |
| [DB設計書](/specs/db-design/) | データベース設計 |
| [用語集](/specs/glossary/) | プロダクト用語定義 |
