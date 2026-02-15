---
title: API仕様
description: REST / GraphQL / WebSocket API
---

Astranote の API は RESTful 設計原則に従い、Cloudflare Workers + Hono フレームワークで実装します。

## 概要

- **Base URL**: `https://api.star-novel.com/v1`
- **Auth**: Bearer Token (JWT)

## 認証

### POST /v1/auth/register

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

### POST /v1/auth/login

ログイン

**Request:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

## ユーザー

### GET /v1/users/me

現在のユーザー情報取得

### GET /v1/users/:userId

公開ユーザー情報取得

## 小説 (Novels)

### GET /v1/novels

小説一覧取得

**Query Params**: `page`, `limit`, `sort`, `genre`, `status`

### POST /v1/novels

小説作成（作家のみ）

### GET /v1/novels/:novelId

小説詳細取得

- `chapters`: 章リストを含む

## 章 (Chapters)

### GET /v1/novels/:novelId/chapters/:chapterId

章本文取得

**Query Params**: `language` (en, ko, etc.)

### POST /v1/novels/:novelId/chapters

章作成

## 翻訳 (Translations)

### POST /v1/novels/:novelId/chapters/:chapterId/translate

手動翻訳リクエスト

## 用語集 (Glossary)

### GET /v1/novels/:novelId/glossary

用語集一覧

### POST /v1/novels/:novelId/glossary

用語追加

## その他

- **Bookmarks**: `GET /v1/users/me/bookmarks`, `POST /v1/novels/:novelId/bookmark`
- **Comments**: `GET`, `POST` (Novel/Chapter comments)
- **Follow**: `POST /v1/users/:userId/follow`
