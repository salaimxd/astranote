---
title: DB設計書
description: Star Novel のデータベース設計仕様
---

## 概要

Star Novel は Turso（libSQL）をメインデータベースとして使用し、Cloudflare KV をキャッシュ層、Cloudflare R2 をファイルストレージとして活用します。

### データベース構成

```
┌─────────────────────────────────────────────────────────────┐
│                    データストア構成                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  【Turso (libSQL)】                                         │
│  ├── ユーザーデータ                                         │
│  ├── 小説・章データ                                         │
│  ├── 翻訳データ                                             │
│  ├── 用語集データ                                           │
│  └── リレーショナルデータ全般                               │
│                                                             │
│  【Cloudflare KV】                                          │
│  ├── セッション情報                                         │
│  ├── レート制限カウンター                                   │
│  ├── ランキングキャッシュ                                   │
│  └── 頻繁アクセスデータのキャッシュ                         │
│                                                             │
│  【Cloudflare R2】                                          │
│  ├── カバー画像                                             │
│  ├── ユーザーアバター                                       │
│  └── 添付ファイル                                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## ER図

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│    users     │       │    novels    │       │   chapters   │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ id (PK)      │──┐    │ id (PK)      │──┐    │ id (PK)      │
│ email        │  │    │ author_id(FK)│◄─┘    │ novel_id(FK) │◄─┐
│ username     │  │    │ title        │       │ number       │  │
│ password_hash│  └───►│ synopsis     │       │ title        │  │
│ display_name │       │ cover_url    │       │ content      │  │
│ avatar_url   │       │ genre        │       │ word_count   │  │
│ bio          │       │ status       │       │ status       │  │
│ role         │       │ is_r18       │       │ author_note  │  │
│ locale       │       │ original_lang│       │ published_at │  │
│ created_at   │       │ created_at   │       │ created_at   │  │
│ updated_at   │       │ updated_at   │       │ updated_at   │  │
└──────────────┘       └──────────────┘       └──────────────┘  │
       │                      │                      │          │
       │                      │                      │          │
       ▼                      ▼                      ▼          │
┌──────────────┐       ┌──────────────┐       ┌──────────────┐  │
│   sessions   │       │ novel_tags   │       │ translations │  │
├──────────────┤       ├──────────────┤       ├──────────────┤  │
│ id (PK)      │       │ novel_id(FK) │       │ id (PK)      │  │
│ user_id (FK) │       │ tag_id (FK)  │       │ chapter_id   │──┘
│ token        │       └──────────────┘       │ language     │
│ expires_at   │              │               │ title        │
│ created_at   │              ▼               │ content      │
└──────────────┘       ┌──────────────┐       │ status       │
                       │    tags      │       │ translated_at│
                       ├──────────────┤       └──────────────┘
                       │ id (PK)      │
                       │ name         │
                       │ slug         │
                       └──────────────┘

┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│  bookmarks   │       │   follows    │       │  glossaries  │
├──────────────┤       ├──────────────┤       ├──────────────┤
│ id (PK)      │       │ id (PK)      │       │ id (PK)      │
│ user_id (FK) │       │ follower_id  │       │ novel_id(FK) │
│ novel_id(FK) │       │ following_id │       │ term         │
│ created_at   │       │ created_at   │       │ reading      │
└──────────────┘       └──────────────┘       │ description  │
                                              │ category     │
┌──────────────┐       ┌──────────────┐       │ created_at   │
│read_progress │       │   comments   │       └──────────────┘
├──────────────┤       ├──────────────┤              │
│ id (PK)      │       │ id (PK)      │              ▼
│ user_id (FK) │       │ user_id (FK) │       ┌──────────────┐
│ chapter_id   │       │ novel_id(FK) │       │glossary_trans│
│ progress     │       │ chapter_id   │       ├──────────────┤
│ position     │       │ parent_id    │       │ id (PK)      │
│ updated_at   │       │ content      │       │ glossary_id  │
└──────────────┘       │ created_at   │       │ language     │
                       └──────────────┘       │ translation  │
                                              └──────────────┘
```

---

## テーブル定義

### users（ユーザー）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー（usr_xxx） |
| email | TEXT | NO | - | メールアドレス（ユニーク） |
| username | TEXT | NO | - | ユーザー名（ユニーク） |
| password_hash | TEXT | YES | NULL | パスワードハッシュ（OAuth時はNULL） |
| display_name | TEXT | YES | NULL | 表示名 |
| avatar_url | TEXT | YES | NULL | アバター画像URL |
| bio | TEXT | YES | NULL | 自己紹介 |
| role | TEXT | NO | 'reader' | 権限（reader/writer/admin） |
| locale | TEXT | NO | 'ja' | 言語設定 |
| email_verified | INTEGER | NO | 0 | メール認証済みフラグ |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TEXT | NO | CURRENT_TIMESTAMP | 更新日時 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_role ON users(role);
```

---

### oauth_accounts（OAuth連携）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| user_id | TEXT | NO | - | ユーザーID（FK） |
| provider | TEXT | NO | - | プロバイダー（google/twitter/discord） |
| provider_user_id | TEXT | NO | - | プロバイダー側のユーザーID |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_oauth_provider_user ON oauth_accounts(provider, provider_user_id);
CREATE INDEX idx_oauth_user_id ON oauth_accounts(user_id);
```

---

### sessions（セッション）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| user_id | TEXT | NO | - | ユーザーID（FK） |
| token | TEXT | NO | - | リフレッシュトークン |
| expires_at | TEXT | NO | - | 有効期限 |
| user_agent | TEXT | YES | NULL | ユーザーエージェント |
| ip_address | TEXT | YES | NULL | IPアドレス |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |

**インデックス:**
```sql
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_token ON sessions(token);
CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);
```

---

### novels（小説）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー（nvl_xxx） |
| author_id | TEXT | NO | - | 作者ID（FK） |
| title | TEXT | NO | - | タイトル |
| synopsis | TEXT | YES | NULL | あらすじ |
| cover_url | TEXT | YES | NULL | カバー画像URL |
| genre | TEXT | NO | - | ジャンル |
| status | TEXT | NO | 'draft' | 状態（draft/ongoing/hiatus/completed） |
| is_r18 | INTEGER | NO | 0 | R18フラグ |
| original_language | TEXT | NO | 'ja' | 原作言語 |
| auto_translate | INTEGER | NO | 1 | 自動翻訳フラグ |
| views_count | INTEGER | NO | 0 | 総閲覧数 |
| bookmarks_count | INTEGER | NO | 0 | ブックマーク数 |
| rating_sum | INTEGER | NO | 0 | 評価合計 |
| ratings_count | INTEGER | NO | 0 | 評価数 |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TEXT | NO | CURRENT_TIMESTAMP | 更新日時 |

**インデックス:**
```sql
CREATE INDEX idx_novels_author_id ON novels(author_id);
CREATE INDEX idx_novels_genre ON novels(genre);
CREATE INDEX idx_novels_status ON novels(status);
CREATE INDEX idx_novels_created_at ON novels(created_at);
CREATE INDEX idx_novels_updated_at ON novels(updated_at);
CREATE INDEX idx_novels_views_count ON novels(views_count);
CREATE INDEX idx_novels_bookmarks_count ON novels(bookmarks_count);
```

---

### tags（タグ）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| name | TEXT | NO | - | タグ名 |
| slug | TEXT | NO | - | スラッグ（URL用） |
| novels_count | INTEGER | NO | 0 | 使用作品数 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_tags_slug ON tags(slug);
CREATE INDEX idx_tags_novels_count ON tags(novels_count);
```

---

### novel_tags（小説タグ関連）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| novel_id | TEXT | NO | - | 小説ID（FK） |
| tag_id | TEXT | NO | - | タグID（FK） |

**インデックス:**
```sql
CREATE PRIMARY KEY (novel_id, tag_id);
CREATE INDEX idx_novel_tags_tag_id ON novel_tags(tag_id);
```

---

### chapters（章）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー（chp_xxx） |
| novel_id | TEXT | NO | - | 小説ID（FK） |
| number | INTEGER | NO | - | 章番号 |
| title | TEXT | NO | - | 章タイトル |
| content | TEXT | NO | - | 本文 |
| word_count | INTEGER | NO | 0 | 文字数 |
| status | TEXT | NO | 'draft' | 状態（draft/scheduled/published） |
| author_note | TEXT | YES | NULL | 作者コメント |
| scheduled_at | TEXT | YES | NULL | 予約公開日時 |
| published_at | TEXT | YES | NULL | 公開日時 |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TEXT | NO | CURRENT_TIMESTAMP | 更新日時 |

**インデックス:**
```sql
CREATE INDEX idx_chapters_novel_id ON chapters(novel_id);
CREATE UNIQUE INDEX idx_chapters_novel_number ON chapters(novel_id, number);
CREATE INDEX idx_chapters_status ON chapters(status);
CREATE INDEX idx_chapters_published_at ON chapters(published_at);
```

---

### translations（翻訳）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| chapter_id | TEXT | NO | - | 章ID（FK） |
| language | TEXT | NO | - | 言語コード（en/ko/zh-TW等） |
| title | TEXT | NO | - | 翻訳されたタイトル |
| content | TEXT | NO | - | 翻訳された本文 |
| author_note | TEXT | YES | NULL | 翻訳された作者コメント |
| status | TEXT | NO | 'pending' | 状態（pending/processing/completed/failed） |
| model | TEXT | YES | NULL | 使用したAIモデル |
| cost | REAL | YES | NULL | 翻訳コスト（USD） |
| translated_at | TEXT | YES | NULL | 翻訳完了日時 |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_translations_chapter_lang ON translations(chapter_id, language);
CREATE INDEX idx_translations_status ON translations(status);
CREATE INDEX idx_translations_language ON translations(language);
```

---

### glossaries（用語集）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| novel_id | TEXT | NO | - | 小説ID（FK） |
| term | TEXT | NO | - | 用語 |
| reading | TEXT | YES | NULL | 読み方 |
| description | TEXT | YES | NULL | 説明 |
| category | TEXT | NO | 'other' | カテゴリ（character/location/item/other） |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TEXT | NO | CURRENT_TIMESTAMP | 更新日時 |

**インデックス:**
```sql
CREATE INDEX idx_glossaries_novel_id ON glossaries(novel_id);
CREATE INDEX idx_glossaries_category ON glossaries(category);
```

---

### glossary_translations（用語翻訳）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| glossary_id | TEXT | NO | - | 用語ID（FK） |
| language | TEXT | NO | - | 言語コード |
| translation | TEXT | NO | - | 翻訳 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_glossary_trans_lang ON glossary_translations(glossary_id, language);
```

---

### bookmarks（ブックマーク）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| user_id | TEXT | NO | - | ユーザーID（FK） |
| novel_id | TEXT | NO | - | 小説ID（FK） |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_bookmarks_user_novel ON bookmarks(user_id, novel_id);
CREATE INDEX idx_bookmarks_novel_id ON bookmarks(novel_id);
```

---

### read_progress（読書進捗）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| user_id | TEXT | NO | - | ユーザーID（FK） |
| chapter_id | TEXT | NO | - | 章ID（FK） |
| progress | REAL | NO | 0 | 進捗率（0.0〜1.0） |
| position | TEXT | YES | NULL | 位置情報 |
| updated_at | TEXT | NO | CURRENT_TIMESTAMP | 更新日時 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_read_progress_user_chapter ON read_progress(user_id, chapter_id);
CREATE INDEX idx_read_progress_user_id ON read_progress(user_id);
```

---

### follows（フォロー）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| follower_id | TEXT | NO | - | フォロワーID（FK） |
| following_id | TEXT | NO | - | フォロー対象ID（FK） |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_follows_pair ON follows(follower_id, following_id);
CREATE INDEX idx_follows_following_id ON follows(following_id);
```

---

### comments（コメント）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| user_id | TEXT | NO | - | ユーザーID（FK） |
| novel_id | TEXT | NO | - | 小説ID（FK） |
| chapter_id | TEXT | YES | NULL | 章ID（FK、章コメント時） |
| parent_id | TEXT | YES | NULL | 親コメントID（返信時） |
| content | TEXT | NO | - | コメント内容 |
| likes_count | INTEGER | NO | 0 | いいね数 |
| is_deleted | INTEGER | NO | 0 | 削除フラグ |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TEXT | NO | CURRENT_TIMESTAMP | 更新日時 |

**インデックス:**
```sql
CREATE INDEX idx_comments_novel_id ON comments(novel_id);
CREATE INDEX idx_comments_chapter_id ON comments(chapter_id);
CREATE INDEX idx_comments_parent_id ON comments(parent_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_comments_created_at ON comments(created_at);
```

---

### comment_likes（コメントいいね）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| user_id | TEXT | NO | - | ユーザーID（FK） |
| comment_id | TEXT | NO | - | コメントID（FK） |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |

**インデックス:**
```sql
CREATE PRIMARY KEY (user_id, comment_id);
```

---

### ratings（評価）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| user_id | TEXT | NO | - | ユーザーID（FK） |
| novel_id | TEXT | NO | - | 小説ID（FK） |
| score | INTEGER | NO | - | 評価点（1〜5） |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |
| updated_at | TEXT | NO | CURRENT_TIMESTAMP | 更新日時 |

**インデックス:**
```sql
CREATE UNIQUE INDEX idx_ratings_user_novel ON ratings(user_id, novel_id);
CREATE INDEX idx_ratings_novel_id ON ratings(novel_id);
```

---

### notifications（通知）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー |
| user_id | TEXT | NO | - | 宛先ユーザーID（FK） |
| type | TEXT | NO | - | 通知タイプ |
| title | TEXT | NO | - | 通知タイトル |
| body | TEXT | YES | NULL | 通知本文 |
| data | TEXT | YES | NULL | 追加データ（JSON） |
| is_read | INTEGER | NO | 0 | 既読フラグ |
| created_at | TEXT | NO | CURRENT_TIMESTAMP | 作成日時 |

**通知タイプ:**
- `new_chapter`: 新章公開
- `new_follower`: 新規フォロワー
- `comment_reply`: コメント返信
- `translation_complete`: 翻訳完了

**インデックス:**
```sql
CREATE INDEX idx_notifications_user_id ON notifications(user_id);
CREATE INDEX idx_notifications_is_read ON notifications(user_id, is_read);
CREATE INDEX idx_notifications_created_at ON notifications(created_at);
```

---

### translation_queue（翻訳キュー）

| カラム | 型 | NULL | デフォルト | 説明 |
|--------|-----|------|-----------|------|
| id | TEXT | NO | - | プライマリキー（ジョブID） |
| chapter_id | TEXT | NO | - | 章ID（FK） |
| target_language | TEXT | NO | - | 翻訳先言語 |
| priority | TEXT | NO | 'normal' | 優先度（low/normal/high） |
| status | TEXT | NO | 'queued' | 状態（queued/processing/completed/failed） |
| attempts | INTEGER | NO | 0 | 試行回数 |
| error_message | TEXT | YES | NULL | エラーメッセージ |
| queued_at | TEXT | NO | CURRENT_TIMESTAMP | キュー追加日時 |
| started_at | TEXT | YES | NULL | 処理開始日時 |
| completed_at | TEXT | YES | NULL | 処理完了日時 |

**インデックス:**
```sql
CREATE INDEX idx_translation_queue_status ON translation_queue(status);
CREATE INDEX idx_translation_queue_priority ON translation_queue(priority, queued_at);
CREATE INDEX idx_translation_queue_chapter ON translation_queue(chapter_id, target_language);
```

---

## Cloudflare KV スキーマ

### セッションキャッシュ

**Key**: `session:{token}`
**TTL**: 15分
```json
{
  "userId": "usr_abc123",
  "role": "writer",
  "locale": "ja"
}
```

### レート制限

**Key**: `ratelimit:{userId}:{endpoint}:{window}`
**TTL**: 1時間
```json
{
  "count": 42,
  "resetAt": 1705312800
}
```

### ランキングキャッシュ

**Key**: `ranking:{type}:{genre}:{language}`
**TTL**: 10分
```json
{
  "data": [...],
  "generatedAt": "2025-01-15T10:00:00Z"
}
```

### 小説メタデータキャッシュ

**Key**: `novel:{novelId}:meta`
**TTL**: 5分
```json
{
  "id": "nvl_xyz789",
  "title": "...",
  "author": {...},
  "stats": {...}
}
```

---

## Cloudflare R2 構造

### バケット構成

```
star-novel-assets/
├── avatars/
│   └── {userId}.webp
├── covers/
│   └── {novelId}.webp
└── attachments/
    └── {attachmentId}.{ext}
```

### 命名規則

| 種類 | パス | 例 |
|------|-----|-----|
| アバター | avatars/{userId}.webp | avatars/usr_abc123.webp |
| カバー | covers/{novelId}.webp | covers/nvl_xyz789.webp |
| 添付 | attachments/{id}.{ext} | attachments/att_001.png |

### CDN URL

```
https://cdn.star-novel.com/{path}
```

---

## マイグレーション

### 命名規則

```
{timestamp}_{description}.sql
例: 20250115100000_create_users_table.sql
```

### サンプルマイグレーション

```sql
-- 20250115100000_create_users_table.sql

CREATE TABLE IF NOT EXISTS users (
  id TEXT PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  username TEXT NOT NULL UNIQUE,
  password_hash TEXT,
  display_name TEXT,
  avatar_url TEXT,
  bio TEXT,
  role TEXT NOT NULL DEFAULT 'reader',
  locale TEXT NOT NULL DEFAULT 'ja',
  email_verified INTEGER NOT NULL DEFAULT 0,
  created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE UNIQUE INDEX IF NOT EXISTS idx_users_email ON users(email);
CREATE UNIQUE INDEX IF NOT EXISTS idx_users_username ON users(username);
CREATE INDEX IF NOT EXISTS idx_users_role ON users(role);
```

---

## パフォーマンス考慮事項

### インデックス設計方針

1. **検索条件に使用されるカラム**にインデックスを作成
2. **外部キー**には必ずインデックスを作成
3. **ソート・フィルタ頻度の高いカラム**にインデックスを作成
4. **複合インデックス**は選択度の高いカラムを先頭に

### クエリ最適化

| パターン | 対策 |
|---------|------|
| 一覧取得 | LIMIT + OFFSET またはカーソルページネーション |
| カウント | 非正規化カウンターを使用（views_count等） |
| 集計 | KV キャッシュを活用 |
| 全文検索 | 将来的に検索エンジン（Algolia等）を検討 |

### キャッシュ戦略

| データ | キャッシュ先 | TTL |
|--------|------------|-----|
| セッション | KV | 15分 |
| ユーザー情報 | KV | 5分 |
| 小説メタ | KV | 5分 |
| ランキング | KV | 10分 |
| 章本文 | CDN | 1時間 |

---

## 関連ドキュメント

| ドキュメント | 内容 |
|-------------|------|
| [API設計書](/specs/api-design/) | RESTful API 仕様 |
| [アーキテクチャ](/specs/architecture/) | システム全体設計 |
| [用語集](/specs/glossary/) | プロダクト用語定義 |
